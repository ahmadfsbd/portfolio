# Technical Capabilities

<section class="skills-orbit-showcase">
  <div class="skills-orbit-copy">
    <p class="skills-kicker">Technical stack by operating layer</p>
    <h2>A practical map of the infrastructure tools I use in production.</h2>
    <p>
      A layer-by-layer view of the stack beneath the platform work: cloud substrate, orchestration, automation, delivery, security, observability, and performance primitives.
    </p>
    <div class="skills-hero-actions" aria-label="Skills page quick links">
      <a href="#stack-layers">Stack Layers</a>
      <a href="#operating-depth">Operating Depth</a>
    </div>
  </div>

  <div class="skills-orbit-stage" aria-label="Animated orbital infrastructure tool map">
    <div class="skills-orbit-grid"></div>

    <div class="skills-orbit-core" aria-label="Infrastructure platform core">
      <div class="skills-core-mesh" aria-hidden="true">
        <span class="skills-core-ring skills-core-ring--outer"></span>
        <span class="skills-core-ring skills-core-ring--inner"></span>
        <svg class="skills-core-datacenter" viewBox="0 0 72 72" role="img" focusable="false">
          <path class="skills-core-dc-cloud" d="M19 28h34l-17-9-17 9Z"></path>
          <path class="skills-core-dc-floor" d="M16 55h40"></path>
          <rect class="skills-core-dc-rack skills-core-dc-rack--side" x="17" y="31" width="12" height="22" rx="2"></rect>
          <rect class="skills-core-dc-rack skills-core-dc-rack--center" x="30" y="26" width="12" height="27" rx="2"></rect>
          <rect class="skills-core-dc-rack skills-core-dc-rack--side" x="43" y="31" width="12" height="22" rx="2"></rect>
          <path class="skills-core-dc-line" d="M21 37h4M21 43h4M34 33h4M34 39h4M34 45h4M47 37h4M47 43h4"></path>
          <circle class="skills-core-dc-light skills-core-dc-light--one" cx="25" cy="49" r="1.3"></circle>
          <circle class="skills-core-dc-light skills-core-dc-light--two" cx="38" cy="49" r="1.3"></circle>
          <circle class="skills-core-dc-light skills-core-dc-light--three" cx="51" cy="49" r="1.3"></circle>
        </svg>
      </div>
    </div>

    <div class="skills-orbit-track skills-orbit-track--outer">
      <span class="skills-orbit-item" style="--angle: 0deg;"><span><img src="/images/logos/openstack.svg" alt="OpenStack" /></span></span>
      <span class="skills-orbit-item" style="--angle: 60deg;"><span><img src="/images/logos/googlecloud.svg" alt="Google Cloud" /></span></span>
      <span class="skills-orbit-item" style="--angle: 120deg;"><span><img src="/images/logos/aws.svg" alt="AWS" /></span></span>
      <span class="skills-orbit-item" style="--angle: 180deg;"><span><img src="/images/logos/redhat.svg" alt="Red Hat" /></span></span>
      <span class="skills-orbit-item" style="--angle: 240deg;"><span><img src="/images/logos/ubuntu.svg" alt="Ubuntu" /></span></span>
      <span class="skills-orbit-item" style="--angle: 300deg;"><span><img src="/images/logos/lxd.svg" alt="LXD" /></span></span>
    </div>

    <div class="skills-orbit-track skills-orbit-track--middle">
      <span class="skills-orbit-item" style="--angle: 30deg;"><span><img src="/images/logos/kubernetes.svg" alt="Kubernetes" /></span></span>
      <span class="skills-orbit-item" style="--angle: 102deg;"><span><img src="/images/logos/openshift.svg" alt="Red Hat OpenShift" /></span></span>
      <span class="skills-orbit-item" style="--angle: 174deg;"><span><img src="/images/logos/rancher.svg" alt="Rancher" /></span></span>
      <span class="skills-orbit-item" style="--angle: 246deg;"><span><img src="/images/logos/docker.svg" alt="Docker" /></span></span>
      <span class="skills-orbit-item" style="--angle: 318deg;"><span><img src="/images/logos/helm.svg" alt="Helm" /></span></span>
    </div>

    <div class="skills-orbit-track skills-orbit-track--inner">
      <span class="skills-orbit-item" style="--angle: 45deg;"><span><img src="/images/logos/terraform.svg" alt="Terraform" /></span></span>
      <span class="skills-orbit-item" style="--angle: 135deg;"><span><img src="/images/logos/ansible.svg" alt="Ansible" /></span></span>
      <span class="skills-orbit-item" style="--angle: 225deg;"><span><img src="/images/logos/juju.svg" alt="Juju" /></span></span>
      <span class="skills-orbit-item" style="--angle: 315deg;"><span><img src="/images/logos/maas.svg" alt="MAAS" /></span></span>
    </div>

    <div class="skills-orbit-pulse skills-orbit-pulse--one"></div>
    <div class="skills-orbit-pulse skills-orbit-pulse--two"></div>
  </div>

  <div class="skills-orbit-legend" aria-label="Orbit layer legend">
    <div><span>Outer Orbit</span><strong>Cloud, operating systems, virtualization</strong></div>
    <div><span>Middle Orbit</span><strong>Containers, orchestration, application runtime</strong></div>
    <div><span>Inner Orbit</span><strong>Provisioning, configuration, deployment automation</strong></div>
  </div>
</section>

## Stack Layers

<div class="skills-visual-grid">
  <article class="skills-visual-card skills-visual-card--platform">
    <div class="skills-card-body">
      <span class="skills-card-index">01</span>
      <h3>Cloud substrate</h3>
      <p>Compute, storage, network, tenancy, and cloud service foundations.</p>
      <div class="skills-tool-list"><span>OpenStack</span><span>GCP</span><span>AWS</span><span>Azure</span><span>Ceph</span><span>KVM/LXD</span><span>VMware</span></div>
    </div>
  </article>

  <article class="skills-visual-card skills-visual-card--runtime">
    <div class="skills-card-body">
      <span class="skills-card-index">02</span>
      <h3>Orchestration and runtime</h3>
      <p>Cluster lifecycle, application packaging, ingress patterns, and workload placement.</p>
      <div class="skills-tool-list"><span>Kubernetes</span><span>Rancher</span><span>OpenShift</span><span>Docker</span><span>Helm</span><span>MicroK8s</span></div>
    </div>
  </article>

  <article class="skills-visual-card skills-visual-card--automation">
    <div class="skills-card-body">
      <span class="skills-card-index">03</span>
      <h3>Provisioning and configuration</h3>
      <p>Declarative infrastructure, bare-metal provisioning, configuration, and repeatable builds.</p>
      <div class="skills-tool-list"><span>Terraform</span><span>Ansible</span><span>Juju</span><span>MAAS</span><span>CloudFormation</span><span>Kolla Ansible</span></div>
    </div>
  </article>

  <article class="skills-visual-card skills-visual-card--delivery">
    <div class="skills-card-body">
      <span class="skills-card-index">04</span>
      <h3>Delivery and engineering tooling</h3>
      <p>Source control, CI/CD, image builds, scripting, and operational utilities.</p>
      <div class="skills-tool-list"><span>GitLab CI</span><span>GitHub Actions</span><span>Jenkins</span><span>Gerrit</span><span>GitOps</span><span>Python</span><span>Bash</span></div>
    </div>
  </article>

  <article class="skills-visual-card skills-visual-card--security">
    <div class="skills-card-body">
      <span class="skills-card-index">05</span>
      <h3>Security and identity controls</h3>
      <p>Hardening, vulnerability scanning, identity integration, and secure service boundaries.</p>
      <div class="skills-tool-list"><span>CIS</span><span>Trivy</span><span>Snyk</span><span>Vault</span><span>TLS</span><span>OAuth</span><span>AD</span><span>Entra ID</span></div>
    </div>
  </article>

  <article class="skills-visual-card skills-visual-card--telco">
    <div class="skills-card-body">
      <span class="skills-card-index">06</span>
      <h3>Performance and network primitives</h3>
      <p>Hardware-aware infrastructure for packet-heavy and latency-sensitive workloads.</p>
      <div class="skills-tool-list"><span>SR-IOV</span><span>DPDK</span><span>OVS-DPDK</span><span>VPP</span><span>NUMA</span><span>HugePages</span><span>Smart NICs</span></div>
    </div>
  </article>
</div>

## Operating Depth

<div class="skills-decision-flow">
  <div><span>1</span><strong>Design</strong><p>Topology, tenancy, storage, network paths, identity boundaries, and lifecycle choices.</p></div>
  <div><span>2</span><strong>Build</strong><p>Infrastructure code, configuration, pipelines, image flows, and reusable platform modules.</p></div>
  <div><span>3</span><strong>Run</strong><p>Upgrades, observability, backup paths, incident support, capacity, and cost visibility.</p></div>
</div>
