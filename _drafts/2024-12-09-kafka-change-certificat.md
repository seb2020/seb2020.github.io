---
layout: post
title: How to change Kafka SSL certificat in OpenShift (AMQ Streams)
date: 2024-12-01 15:09:00
description: How to change Kafka SSL certificat in OpenShift (AMQ Streams) based on Strimzi
tags: kafka
categories: kafka
featured: true
toc:
  sidebar: left
---

## Generate the new certificat

Create the `wildcard_subdomain_<kafka-cluster>_<year>.conf`:

```
[ req ]
default_bits = 2048
default_md = sha256
prompt = no
encrypt_key = no
distinguished_name = dn
req_extensions = req_ext
[ dn ]
CN = *.<kafka-cluster>.xxxx.ch
emailAddress = xxx@domain.ch
O = My Company
OU = IT
L = Rechy
ST = Valais
C = CH
[ req_ext ]
subjectAltName = @alt_names
[ alt_names ]
DNS.1 = *.<kafka-cluster>.xxxx.ch
```

Generate the CSR:

```bash
openssl req -new -out wildcard_subdomain_<kafka-cluster>.xxxx.ch.csr -keyout wildcard_subdomain_<kafka-cluster>.xxxx.ch.key -config wildcard_subdomain_<kafka-cluster>.xxxx.ch_<year>.conf
```	

Submit the request to your CA.

## Add the new certificat

Disable reconciliation of Kafka cluster: 

```bash
oc annotate Kafka <kafka-cluster> strimzi.io/pause-reconciliation="true"
```

Update the Kafka CRD to disable certificate management:

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
# ...
spec:
# ...
  clusterCa:
    generateCertificateAuthority: false
```


Update the certificat secret. The certificat must be the full chain. The old certificat must be renamed with the date and the p12 must be regenerated with the following command:

```bash
openssl pcs12 -export -in wildcard_subdomain_<kafka-cluster>.xxxx.ch_<year>_full.crt -nokeys -out ca.p12 -password pass:<PASSWORD> -caname ca.crt
```

Mettre dans le secret les nouvelles valeurs et augmenter de 1 l'annotation "strimzi.io/ca-cert-generation"`

Update the secret `<kafka-cluster>-cluster-ca-cert` and increase the annotation `strimzi.io/ca-cert-generation`.

```yaml
kind: Secret
apiVersion: v1
metadata:
  annotations:
    strimzi.io/ca-cert-generation: '2'
  name: <kafka-cluster>
  namespace: amq-streams-kafka
  labels:
    app.kubernetes.io/instance: <kafka-cluster>
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: certificate-authority
    app.kubernetes.io/part-of: strimzi-<kafka-cluster>
    strimzi.io/cluster: <kafka-cluster>
    strimzi.io/component-type: certificate-authority
    strimzi.io/kind: Kafka
    strimzi.io/name: strimzi
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FU...
  ca-2024-09-05T08-15-07Z.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0...
  ca.p12: MIIRJwIBAzCCEN0GCSqGSIb3DQEHAaCCEM4EghDKMI...
  ca.password: WVRLMFoyVDY2Mzc0
type: Opaque
```

Update the secret `<kafka-cluster>-cluster-ca` and increase the annotation `strimzi.io/ca-key-generation`:

```yaml
kind: Secret
apiVersion: v1
metadata:
  annotations:
    strimzi.io/ca-key-generation: '2'
  name: <kafka-cluster>-cluster-ca
  namespace: amq-streams-kafka
  labels:
    app.kubernetes.io/instance: <kafka-cluster>
    app.kubernetes.io/managed-by: strimzi-cluster-operator
    app.kubernetes.io/name: certificate-authority
    app.kubernetes.io/part-of: strimzi-<kafka-cluster>
    strimzi.io/cluster: <kafka-cluster>
    strimzi.io/component-type: certificate-authority
    strimzi.io/kind: Kafka
    strimzi.io/name: strimzi
data:
  ca.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV...
type: Opaque
```

Enable the reconciliation of Kafka cluster:

```bash
oc annotate Kafka <kafka-cluster> strimzi.io/pause-reconciliation-
```