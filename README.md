<p align="center">
  <img src="https://raw.githubusercontent.com/cert-manager/cert-manager/d53c0b9270f8cd90d908460d69502694e1838f5f/logo/logo-small.png" height="256" width="256" alt="cert-manager project logo" />
</p>

# ACME Wedos webhook

This solver can be used when you want to use cert-manager with Wedos WAPI API

## Installation

### cert-manager

Follow the [instructions](https://cert-manager.io/docs/installation/) using the cert-manager documentation to install it
within your cluster.

### Webhook

#### Using public helm chart

```bash
helm repo add cert-manager-webhook-wedos https://code-growers.github.io/cert-manager-webhook-wedos/
helm install --namespace cert-manager cert-manager-webhook-wedos cert-manager-webhook-wedos/cert-manager-webhook-wedos
```

## Issuer

Create a `ClusterIssuer` or `Issuer` resource as following:
(Keep in Mind that the Example uses the Staging URL from Let's Encrypt. Look
at [Getting Start](https://letsencrypt.org/getting-started/) for using the normal Let's Encrypt URL.)

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: wedos-issuer
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory

    # Email address used for ACME registration
    email: mail@example.com # REPLACE THIS WITH YOUR EMAIL!!!

    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging

    solvers:
      - dns01:
          webhook:
            groupName: hetzner.cert-mananger-webhook.noshoes.xyz
            solverName: wedos
            config:
                apiUsername: services@mccidentity.com
                apiKeySecretRef:
                  name: webhook-wedos-secret
                  key: password
```

### Credentials

In order to access the WAPI API, the webhook needs **WAPI** password. See https://kb.wedos.global/wapi-manual/#integrate

If you choose another name for the secret than `webhook-wedos-secret`, you must install the chart with a modified `secretName`
value. Policies ensure that no other secrets can be read by the webhook. Also modify the value of `secretName` in the
`[Cluster]Issuer`.

The secret for the example above will look like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: webhook-wedos-secret
  namespace: cert-manager
type: Opaque
data:
  password: your-key-base64-encoded
```

### Create a certificate

Finally you can create certificates, for example:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-cert
  namespace: cert-manager
spec:
  dnsNames:
    - "*.example.com"
  issuerRef:
    name: wedos-issuer
    kind: ClusterIssuer
  secretName: example-cert
```
