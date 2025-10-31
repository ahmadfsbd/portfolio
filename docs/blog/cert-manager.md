# Automating TLS Certificate Management with Cert-Manager, ACME, and Infoblox

<img src="/images/blog/cert-manager.png" alt="CertManager ACME Infoblox" height="40" />

---

# Introduction

Managing TLS certificates manually is time-consuming and error-prone. Keeping track of expiry dates, generating CSRs, and updating Kubernetes secrets manually can easily lead to outages. In modern **cloud-native** environments, where services are dynamic and ephemeral, automation is essential — not optional.

This is where **cert-manager** comes in.

Cert-manager is a Kubernetes controller that automates the **issuance, renewal, and management of TLS certificates** from multiple sources such as Let's Encrypt, Venafi, Vault, or enterprise ACME servers like **EJBCA**.  

When integrated with enterprise DNS systems like **Infoblox**, it enables fully automated, secure, and policy-compliant certificate management — ideal for internal and external workloads.

---

## What Is Cert-Manager?

**Cert-manager** extends Kubernetes by introducing custom resources to manage certificates and issuers.  
It continuously monitors certificate expiration, automatically requests renewals, and updates secrets used by Ingress or workloads.

**Key resources introduced by cert-manager:**

- **Issuer / ClusterIssuer** – Defines how certificates are obtained (e.g., ACME, Vault, CA).
- **Certificate** – Describes what you need (DNS names, secret storage).
- **CertificateRequest** – Internal resource for tracking issuance requests.
- **Order** – Tracks the lifecycle of ACME requests and challenges.

---

## Understanding ACME and DNS-01 Challenges

**ACME (Automatic Certificate Management Environment)** is the protocol behind Let’s Encrypt and many enterprise CAs.  
It automates certificate issuance through domain ownership validation, using challenges like **DNS-01** or **HTTP-01**.

Here’s the simplified ACME flow:

1. Cert-manager requests a certificate for `example.org`.
2. The ACME server issues a **challenge**.
3. Cert-manager proves ownership (e.g., via a DNS TXT record).
4. Once validated, the CA issues a signed certificate.
5. Cert-manager stores it as a **Kubernetes Secret** and renews it automatically.

**DNS-01 challenges** are particularly useful in internal environments where HTTP validation isn’t possible. They work by adding a `_acme-challenge.example.org` TXT record in DNS — typically automated via API access to your DNS provider.

---

## Integrating Infoblox DNS for Automation

In enterprise networks, **Infoblox** often manages internal DNS zones.  
To perform DNS-01 challenges automatically, cert-manager uses a **webhook solver** — a lightweight service that integrates with Infoblox’s **WAPI (Web API)**.

The Infoblox webhook:

- Receives DNS-01 validation requests.
- Connects to the Infoblox Grid via WAPI.
- Creates and deletes TXT records for domain validation.
- Reports status back to cert-manager.

This enables **fully automated validation** without manual DNS changes.

---

## Architecture Overview

<img src="/images/blog/cert-manager-02.png" alt="cert-manager-02.png" height="40" />

---

## Step-by-Step Tutorial

### 1. Prerequisites

- A working Kubernetes cluster.
- An ACME-compatible CA (e.g., Let’s Encrypt or internal EJBCA).
- Infoblox WAPI access (host, username, password).
- Helm and kubectl installed locally.

---

### 2. Install Cert-Manager

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

kubectl create namespace cert-manager

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set installCRDs=true
```

---

### 3. Deploy Infoblox Webhook

```bash
helm repo add infobloxopen https://infobloxopen.github.io/cert-manager-webhook-infoblox/
helm repo update

helm install cert-manager-webhook-infoblox \
  infobloxopen/cert-manager-webhook-infoblox \
  -n cert-manager
```

---

### 4. Create Infoblox Credentials Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: infoblox-credentials
  namespace: cert-manager
type: Opaque
stringData:
  password: "<INFOBLOX_PASSWORD>"
```

Apply the secret:

```bash
kubectl apply -f infoblox-secret.yaml
```

---

### 5. Define a ClusterIssuer for ACME + Infoblox

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: internal-acme
spec:
  acme:
    email: devops@example.org
    server: https://certauthority.internal.example.org/ejbca/acme/Corporate_CA/directory
    privateKeySecretRef:
      name: internal-acme-account-key
    solvers:
    - dns01:
        webhook:
          groupName: acme.mycompany.com
          solverName: infoblox-wapi
          config:
            host: infoblox-gm.internal.example.org
            view: internal
            port: 443
            sslVerify: false
            wapiVersion: "2.11.2"
            username: acme-api-user
            secretRef:
              name: infoblox-credentials
              key: password
```

Apply it:

```bash
kubectl apply -f clusterissuer.yaml
```

---

### 6. Request a Certificate

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: test-apps-example-org-tls
  namespace: apps
spec:
  secretName: test-apps-example-org-tls
  dnsNames:
  - test-apps.internal.example.org
  issuerRef:
    kind: ClusterIssuer
    name: internal-acme
```

```bash
kubectl apply -f certificate.yaml
```

---

### 7. Verify the Certificate Request

```bash
kubectl get certificaterequests -n apps
kubectl describe certificaterequest test-apps-example-org-tls-1 -n apps
```

Expected status:

```
Message: Certificate request has been approved by cert-manager.io
Reason: Pending
```

Once the Infoblox DNS validation succeeds, the certificate will be issued and stored as a Kubernetes Secret.

---

### 8. Check the TLS Secret

```bash
kubectl get secret test-apps-example-org-tls -n apps -o yaml
```

You'll see base64-encoded values for:

- `tls.crt` (the certificate)
- `tls.key` (the private key)

---

## Using Ingress Annotations for Auto-Cert Generation

Cert-manager can automatically generate and renew TLS certificates for Ingress resources using simple annotations.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-app-ingress
  namespace: apps
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "internal-acme"
spec:
  tls:
  - hosts:
    - test-apps.internal.example.org
    secretName: test-apps-example-org-tls
  rules:
  - host: test-apps.internal.example.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: test-app-service
            port:
              number: 80
```

When you apply this Ingress, cert-manager:

1. Detects the annotation.
2. Automatically creates a Certificate resource.
3. Handles validation via Infoblox.
4. Stores and renews the TLS secret automatically.

**No manual Certificate or Secret creation needed.**

---

## Automatic Renewal

Cert-manager continuously monitors certificate expiration and renews them well before expiry (by default, at 2/3 of the certificate lifetime).  
The entire ACME → DNS validation → issuance → Secret update happens automatically.

---

## Troubleshooting Tips

| Issue | Likely Cause | Fix |
|-------|--------------|-----|
| Certificate stuck in "Pending" | DNS TXT record not created | Check Infoblox WAPI access or webhook logs |
| `forbidden: subjectaccessreviews.authorization.k8s.io` | Missing RBAC roles | Ensure webhook ClusterRoleBindings exist |
| `APIService acme.mycompany.com already exists` | Leftover webhook CRDs | Delete with `kubectl delete apiservice v1alpha1.acme.mycompany.com` |
| DNS port unreachable | Firewall or network policy | Test Infoblox host connectivity via `nc -vz <host> 443` |

---

## Conclusion

By integrating cert-manager, ACME, and Infoblox, organizations can automate their entire certificate lifecycle — from issuance to renewal — securely and at scale.  
This eliminates manual effort, reduces human error, and ensures applications remain compliant and available.

**In short:**  
✅ Set it once.  
✅ Let cert-manager handle the rest.

---

**Tags:** #CertManager #Kubernetes #DevOps #CloudSecurity #ACME #Infoblox #EJBCA #TLS #Automation #K8sCertificates