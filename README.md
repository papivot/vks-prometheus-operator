# vks-prometheus-operator

The following diagram shows the three logical planes of the deployed stack: how traffic flows in from a browser, how the Operator manages the Prometheus instance, and how Prometheus reaches all its scrape targets.

```mermaid
flowchart TD
    Browser["Browser / Client"] -->|"HTTPS :443"| IGW["Istio IngressGateway\nLoadBalancer IP"]
    IGW --> Ingress["K8s Ingress\ningressClassName: istio\ncert-manager TLS"]
    Ingress --> NginxProxy["nginx-auth-proxy\nBasic Auth :4180"]
    NginxProxy --> PromSvc["prometheus-operated\nClusterIP :9090"]
    PromSvc --> PromPod["Prometheus Pod\nStatefulSet / 30 Gi PVC"]

    CertMgrAddon["cert-manager addon"] -->|"Issues TLS cert"| TLSSec["prometheus-tls Secret"]
    TLSSec --> Ingress

    PromOp["Prometheus Operator\ntanzu-system-monitoring"] -->|"Reconciles Prometheus CR"| PromPod
    PromOp -->|"Watches CRDs cluster-wide"| SMs["ServiceMonitors\nPodMonitors"]
    SMs -->|"Generates scrape config\nhot-reload via /-/reload"| PromPod

    PromPod -->|"Scrapes"| KSM["kube-state-metrics\ntanzu-system-monitoring"]
    PromPod -->|"Scrapes"| NodeExp["node-exporter\ntanzu-system-monitoring"]
    PromPod -->|"Scrapes"| APIServer["kube-apiserver\ndefault/kubernetes :443"]
    PromPod -->|"Scrapes"| Kubelet["kubelet + cAdvisor\nkube-system :10250"]
    PromPod -->|"Scrapes"| CoreDNS["CoreDNS\nkube-system/kube-dns :9153"]
    PromPod -->|"Scrapes"| Istio["istiod :15014\nigw :15090\nztunnel :15020"]
    PromPod -->|"Scrapes"| CertMgr["cert-manager\ncert-manager ns :9402"]
    PromPod -->|"Scrapes"| Antrea["antrea ctrl :10349\nantrea agent :10350"]
    PromPod -->|"Scrapes"| CSI["vsphere-csi\nvmware-system-csi :2112/:2113"]
```
