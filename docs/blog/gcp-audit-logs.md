---
title: "Building Audit-Based Alerting for Sensitive Cloud Storage Buckets"
description: "How I designed a low-noise Google Cloud alerting pattern for detecting suspicious reads and deletes against sensitive storage buckets in a Trusted Research Environment."
tags:
  - Google Cloud
  - Terraform
  - Cloud Logging
  - Cloud Monitoring
  - Security Engineering
  - Infrastructure as Code
---

# Building Audit-Based Alerting for Sensitive Cloud Storage Buckets

<img src="/images/blog/gcp-audit-alerting-architecture.jpg" alt="Architecture diagram for sensitive Cloud Storage bucket audit alerting in Google Cloud" />

Sensitive research environments usually have a simple rule hiding a difficult engineering problem: data should be usable inside the environment, but it should not quietly leave it.

I worked on a Google Cloud alerting design for a Trusted Research Environment where users can legitimately read data from controlled buckets inside approved virtual machines. The risk we wanted to detect was different: a user account or workload identity successfully reading sensitive Cloud Storage objects from outside the approved network path. We also included object deletes as a high-signal destructive event.

This post describes the architecture I designed and implemented with Terraform.

## The Problem

The environment had multiple project types:

- Sandbox projects, where researchers work in isolated environments.
- A library project, where shared sensitive datasets can live.
- A master project, where central platform services are managed.

Some buckets are intentionally downloadable, so alerting on every object read would create noise. Other buckets contain sensitive data and should not be accessed from outside approved TRE network ranges.

The control needed to answer one question:

> Did a successful Cloud Storage object read or delete happen against a sensitive bucket from outside an approved TRE source?

We were not trying to alert on failed attempts. Failed reads are useful for investigation, but successful movement or deletion of data is the higher-priority signal.

## Design Goals

The design had a few constraints:

- Use Google Cloud native services where possible.
- Avoid running custom detectors or scheduled scanners unless necessary.
- Keep alert noise low.
- Preserve enough logs centrally for investigation.
- Avoid putting object paths into metric labels or broad notifications.
- Manage the whole pattern with Terraform.
- Make rollout project-by-project, using feature flags.

The final design uses Cloud Audit Logs, Cloud Logging sinks, a central Cloud Logging log bucket, a bucket-scoped log-based metric, and a Cloud Monitoring alert policy.

## Architecture

At a high level, each enabled source project keeps its normal local audit logs, but also routes a filtered copy of relevant Cloud Storage access events into a central Cloud Logging log bucket in the master project.

```text
Source projects
  Cloud Storage buckets
    -> Cloud Audit Logs
    -> Source project Cloud Logging
    -> Project log sink
    -> Master project Cloud Logging log bucket
    -> Bucket-scoped log-based metric
    -> Cloud Monitoring alert
    -> Email / Monitoring notification channels
```

The central bucket is a Cloud Logging log bucket, not a Cloud Storage bucket. That distinction matters because the system is routing logs through Cloud Logging, not copying audit data into object storage.

Bucket-scoped log-based metrics fit this pattern well because they evaluate entries routed into a specific log bucket, including logs that originated in other projects.

## What Gets Routed

The source project sink routes successful Cloud Storage object access events for scoped sensitive buckets:

```text
log_id("cloudaudit.googleapis.com/data_access")
resource.type="gcs_bucket"
protoPayload.serviceName="storage.googleapis.com"
(
  protoPayload.methodName="storage.objects.get"
  OR protoPayload.methodName="storage.objects.delete"
)
NOT protoPayload.status.code:*
```

The real sink filter then adds the bucket scoping rules: include the sensitive buckets and exclude buckets that are intentionally public, downloadable, or otherwise out of scope.

For reads, Cloud Storage Data Access audit logging needs `DATA_READ`. If delete detection is enabled, `DATA_WRITE` is also required because Cloud Storage object deletes are write-classified data access events.

The important design decision is that the sink does not apply the outside-network check. It routes all successful reads and deletes for scoped sensitive buckets. The network condition is applied later by the metric and alert policy.

That gives two benefits:

1. The central log bucket remains useful for investigation, even when an event does not alert.
2. The alerting logic can be adjusted without changing every source project sink.

## What Triggers an Alert

The master project owns a bucket-scoped log-based metric over the central log bucket. The metric counts events where the caller IP is not inside an approved TRE CIDR:

```text
NOT (
  ip_in_net(protoPayload.requestMetadata.callerIp, "<APPROVED_CIDR_1>")
  OR ip_in_net(protoPayload.requestMetadata.callerIp, "<APPROVED_CIDR_2>")
)
```

This is intentionally conservative. In Cloud Logging filters, `ip_in_net` returns false if the field is missing, defaulted, or not a legal IP address. With the `NOT (...)` wrapper, those cases become suspicious instead of being silently trusted.

The metric includes labels for investigation:

```text
source_project_id
bucket_name
principal_email
caller_ip
method_name
```

Object names are deliberately not labels. They can create high-cardinality time series and may leak sensitive metadata into alert notifications.

The Cloud Monitoring alert policy is simple:

```text
Severity: CRITICAL
Condition: sum over 10 minutes > 0
Grouped by: source project, bucket, principal, caller IP, and method
```

That grouping means repeated events from the same actor and source are collected into one incident, while distinct users or source IPs can still open separate incidents.

## Terraform Implementation

The implementation is controlled by feature flags.

At the global level:

```hcl
bucket_audit_alerting = {
  enabled                       = true
  library_project_enabled       = false
  include_deletes               = true
  log_bucket_id                 = "storage-access"
  log_bucket_location           = "europe-west2"
  retention_days                = 365
  additional_approved_cidrs     = []
  excluded_sandbox_bucket_names = []
  notification_channel_names    = []
}
```

At the sandbox level:

```hcl
storage_audit_alerting_enabled = true
```

The Terraform creates:

- Cloud Storage Data Access audit logging for enabled projects.
- Source project Cloud Logging sinks.
- A central Cloud Logging log bucket with 365-day retention.
- IAM for sink writer identities.
- A bucket-scoped log-based metric.
- A Cloud Monitoring alert policy.
- Email or existing Monitoring notification channels.

The central log bucket has `prevent_destroy = true` to reduce the chance of accidental deletion through Terraform.

## Validation and Edge Cases

One of the main lessons was that audit log filters need to be tested with real log entries, not guessed from memory. Cloud Logging fields for Cloud Storage access are precise, and small differences in query syntax matter.

For example, this pattern worked reliably for excluding approved internal source ranges:

```text
NOT (ip_in_net(protoPayload.requestMetadata.callerIp, "<APPROVED_CIDR>"))
```

That filter should be tested with:

- A successful object read from an approved TRE VM.
- A successful object read from outside the approved CIDRs.
- A successful object delete, if delete detection is enabled.
- A log entry with a missing, redacted, or non-IP caller field.
- A bucket that is intentionally excluded.

There are also Cloud Storage audit logging limitations to keep in mind. Cloud Audit Logs do not track access to public objects, and some authenticated browser download paths can redact `principalEmail` and `callerIp` when the download happens outside the Google Cloud console. Those cases need explicit validation before relying on the alert as a complete exfiltration control.

Finally, log-based metrics do not backfill old logs. After creating a metric, a fresh matching event is needed to validate the metric and alert path.

## Cost Considerations

The main cost driver is Cloud Logging volume and retention:

- Source projects keep their normal audit logs.
- The central log bucket stores a filtered copy of successful sensitive bucket reads and deletes.
- Retention beyond the default period adds storage cost.

The design controls cost by:

- Routing only `storage.objects.get` and, when enabled, `storage.objects.delete`.
- Excluding intentionally downloadable buckets.
- Avoiding high-cardinality metric labels such as object path.
- Rolling out per project with a feature flag.
- Reviewing log volume after the first full billing week.

For most environments, this is much cheaper and simpler than triggering a custom function for every audit event.

## Why This Pattern Worked Well

This design gives the platform team a focused detection path:

- Native audit source.
- Central retention.
- Minimal custom code.
- Clear Terraform ownership.
- Alert grouping to avoid notification spam.
- Enough context in each incident to investigate quickly.

It also keeps the architecture flexible. If the definition of an approved network changes, the metric filter can be updated. If more buckets become intentionally downloadable, they can be excluded from the sink. If Slack is approved later, it can be added through Cloud Monitoring notification channels without putting Slack tokens into application code.

## Future Improvements

The next useful improvements would be:

- Move from broad project-level sink writer permissions to log-bucket-level IAM where the provider and module structure support it cleanly.
- Add dashboards for access volume by project, bucket, and principal.
- Add a separate low-severity control for denied access attempts.
- Periodically review excluded buckets to make sure they are still intentionally public or downloadable.
- Add automated validation tests that generate a harmless object read and confirm the metric increments.

## Closing Thought

Good security alerting is not just about catching everything. It is about choosing the right signal, keeping the noise low, and making sure the person receiving the alert has enough context to act.

For this use case, Cloud Audit Logs plus Cloud Logging and Cloud Monitoring provided a strong native foundation. Terraform made the pattern repeatable across projects, and the central log bucket gave the team a reliable place to investigate suspicious access without building a custom detection pipeline from scratch.
