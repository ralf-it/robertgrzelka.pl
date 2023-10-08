# Azure AKS use Cert Manager to generate TLS Cert with Azure DNS and Letsencrypt


# Kubernetes Cert Manager with Azure DNS and LetsEncrypt

Following my last piece on [Azure DNS as external resource for SSL ACME Challenge](/blog/azure/2023-10-03-azure-dns-for-letsencrypt-acme-challenge/), today, we are diving deep into securing Kubernetes clusters using the Cert-Manager to generate trusted TLS certificates, particularly for Nginx Ingress, with Azure DNS and Let’s Encrypt.

## Introduction

The quest for enhanced security in Kubernetes often leads to the integration of TLS certificates. Leveraging Cert Manager with Azure DNS and Let’s Encrypt simplifies this process, ensuring not only the automation of certificates issuance but also their renewal, enhancing the cluster’s security.

## ACME Challenges

Let’s Encrypt employs the ACME protocol to validate domain ownership before issuing certificates. Among the various challenge types, HTTP01 and DNS01 are prominent.

### HTTP01

This challenge requires the applicant to prove the ownership of the domain by providing a special file on an HTTP server at a predefined URL.

### DNS01

In the DNS01 challenge, the domain owner proves their ownership by creating a specific DNS TXT record. This approach is often favored for its ability to bypass the HTTP ingress firewalls.

### Azure DNS Zone for DNS01 Challenge

Azure DNS facilitates the hosting of your DNS domain and the management of your domain records. Integrating this with Cert Manager enables automated DNS01 challenges completion.

## Code Blueprints

### Cert Manager via Helm Chart

The Helm charts below define the deployment of Cert Manager in the Kubernetes cluster, ensuring the CRDs installation and setting the DNS policy for the pods.

#### Helm cert-manager/Chart.yaml
```yaml
apiVersion: v2
name: cert-manager
version: 1.11.0
appVersion: 1.11.0
dependencies:
  - name: cert-manager
    version: 1.11.0
    repository: https://charts.jetstack.io
```

#### Helm cert-manager/values.yaml
```yaml
cert-manager:
  installCRDs: true
  podDnsPolicy: "None"
  podDnsConfig:
    nameservers:
      - "8.8.8.8"
```

### Cert Manager Certs via Helm Chart

The configuration below defines a Kubernetes secret to store Azure DNS credentials and a ClusterIssuer for Let's Encrypt ACME DNS01 challenge using Azure DNS.

#### Helm cert-manager-certs/template/cluster-issuer.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cert-manager-azure-dns
type: Opaque
stringData:
  SP_CLIENT_SECRET: {{ .Values.azureDNS.SP_CLIENT_SECRET }}

---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: azuredns-letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: {{ .Values.email }}
    privateKeySecretRef:
      name: azuredns-letsencrypt
    solvers:
    - dns01:
        cnameStrategy: Follow
        azureDNS:
          environment: AzurePublicCloud
          hostedZoneName: {{ .Values.azureDNS.AZURE_ZONE_NAME }}
          subscriptionID: {{ .Values.azureDNS.AZURE_SUBSCRIPTION_ID }}
          resourceGroupName: {{ .Values.azureDNS.AZURE_RESOURCE_GROUP }}
          tenantID: {{ .Values.azureDNS.SP_TENANT_ID }}
          clientID: {{ .Values.azureDNS.SP_CLIENT_ID }}
          clientSecretSecretRef:
            name: cert-manager-azure-dns
            key: SP_CLIENT_SECRET
```

### Nginx Ingress via Helm Chart

This section illustrates how to use the ClusterIssuer to generate a TLS certificate through annotations in the values for the nginx ingress of a service.

#### Helm <my-exposed-service>/values.yaml
```yaml
ingress:
    enabled: true
    pathType: ImplementationSpecific
    ingressClassName: "nginx-ingress"
    hosts:
    - name: example.com
      path: "/"
    hostname: example.com
    tls:
      - secretName: example.com-tls
        hosts:
          - example.com

    annotations:
      kubernetes.io/tls-acme: "true"
      ingress.kubernetes.io/ssl-redirect: "true"
      cert-manager.io/cluster-issuer: "azuredns-letsencrypt"
```

## Bringing It All Together

With the integration of Azure DNS, Cert Manager, and Let’s Encrypt, Kubernetes clusters can now seamlessly generate and manage TLS certificates. This not only bolsters the security posture but also automates the renewal process, ensuring uninterrupted secure services.

### Debugging and Validation

Ensure to monitor the certificate issuance process by checking the `Certificate`, `CertificateRequest`, and `Order` objects. Validate that the certificates are correctly configured and downloaded to your cluster.

### Final Thoughts

The amalgamation of Azure DNS with Kubernetes Cert Manager and Let’s Encrypt offers an efficient, automated, and secure method to manage TLS certificates. Always ensure to redirect all traffic to your AKS cluster post the successful setup.

Stay tuned for more insights, and feel free to reach out for any queries or discussions!

*Stay Secure, Stay Automated!*

---

> Author: [Robert Grzelka](https://robertgrzelka.pl)
> URL: https://robertgrzelka.pl/blog/kubernetes/azure-aks-cert-manager-azure-dns-letsencrypt-copy/

