---
layout: post
title: Securing Akka cluster communication in Kubernetes
author: James Roper
short: Securing Akka cluster communication in Kubernetes with mTLS certificates that are frequently rotated
category: article
tags: [kubernetes,cluster,security]
---

A feature of Akka that we've been using in production for some time now but haven't made a big deal about is Akka remoting's support for using mTLS certificates that are frequently rotated. This support is designed to work with cert-manager and other Kubernetes based secret providers with an absolute minimum of configuration. All you need is two lines of configuration in Akka's configuration file, and you're ready to go on the Akka side.

This feature is important for secure Akka deployments. Akka clusters use a proprietary protocol to communicate with each other. This protocol by default contains no authentication or encryption, and so to prevent malicious hosts from joining your Akka clusters or eavesdropping on your Akka cluster communication, you need to use mTLS to secure the communication.

In a Kubernetes environment, many people turn to a service mesh such as Istio, Linkerd or Consul to authenticate and encrypt their network communications, however unfortunately this is not an option for Akka cluster communication. The goal of a service mesh is to ensure that services do not need to be aware of where and how the services they talk to are deployed, and so service meshes hide this, so the service thinks its only talking to one logical service, while the service mesh handles concerns such as load balancing, encryption, authentication and authorization, canary and A/B deploys, and so on. Akka clusters however need to understand how and where they are deployed, in order to implement their stateful features such as sharding, replication and P2P messaging. So, when deploying a service that uses Akka clustering to a service mesh, the Akka cluster communication must bypass the service mesh.

## Prerequesites

In this blog post, I'll explain how to provision the certificates needed by Akka using [cert-manager](https://cert-manager.io/). I'll assume you have a Kubernetes cluster with a [standard cert-manager installation](https://cert-manager.io/docs/installation/).

## Understanding the certificates

First off, let me explain a few concepts. cert-manager has a concept of `Certificates` and `Issuers`. A `Certificate` is a CRD that you deploy that cert-manager will reconcile into a Kubernetes `Secret` containing a TLS certificate. The `Certificate` references a `Issuer`, and the `Issuer` describes how `Certificates` that reference it should be issued.

In order to support frequently rotated certificates, Akka can't just use a self signed certificate, since self signed certificates need to be the same at both ends to authenticate each other properly, and during the time when the certificate is being rotated, two different Akka nodes may have different certificates. Instead, Akka needs certificates issued by a certificate authority (CA). The CA verifies whether a certificate should be trusted or not, so during rotation, both the old and the new certificate can work together, because both are signed by the same CA. So, when we issue our certificates, we'll use cert-managers CA `Issuer` type.

The CA `Issuer` itself needs a certificate to do its signing, and this certificate we'll also provision using `cert-manager`. That certificate we're not going to rotate - its private key never gets shared with anything outside of `cert-manager`, and so rotating it is not as necessary. Because of this, it will use a self signed certificate, and provisioning that certificate can be done by using a cert-manager self signed `Issuer` type.

So, in total, we're going to have two issuers, a self signed issuer that issues certificates for the CA issuer, and then that CA issuer will issue certificates that are frequently rotated for our Akka service to use. The self signed issuer, certificate, and CA issuer can be reused across different Akka deployments - more on that later.

## Kubernetes resources

First we deploy the self signed issuer:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: self-signed-issuer
spec:
  selfSigned: {}
```

We're creating this for the whole cluster, self signed issuers don't have any state or configuration, there's no reason to have more than one for your entire cluster.

Next we create a self signed certificate for our CA issuer to use that references this issuer:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: akka-tls-ca-certificate
  namespace: default
spec:
  issuerRef:
    name: self-signed-issuer
    kind: ClusterIssuer
  secretName: akka-tls-ca-certificate
  commonName: default.akka.cluster.local
  # 100 years
  duration: 876000h
  # 99 years
  renewBefore: 867240h
  isCA: true
  privateKey:
    rotationPolicy: Always
```

We've created this in the `default` namespace, which will be the same namespace that our Akka service is deployed to. If you're using a different namespace, you'll need to update accordingly.

The `commonName` isn't very important, it's not actually used anywhere, though may be useful for debugging purposes if you're ever looking into why a particular certificate isn't trusted by a service. We use a naming convention for common names and DNS names that follows the the pattern `<service-name>.<namespace>.akka.cluster.local`. The CA uses the same convention without the service name. This convention doesn't need to be followed, but it makes it easy to reason about the purpose of any given certificate.

Now we create the CA issuer:

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: akka-tls-ca-issuer
  namespace: default
spec:
  ca:
    secretName: akka-tls-ca-certificate
```

This uses the secret that we configured to be provisioned in the certificate above. Finally, we provision the certificate that our Akka service is going to use - we're assuming that the name of the service in this case is `my-service`:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-service-akka-tls-certificate
  namespace: default
spec:
  issuerRef:
    name: akka-tls-ca-issuer
  secretName: my-service-akka-tls-certificate
  dnsNames:
  - my-service.default.akka.cluster.local
  duration: 24h
  renewBefore: 16h
  privateKey:
    rotationPolicy: Always
```

The actual `dnsName` configured isn't important, since Akka cluster does not actually use these names for looking up the service, as long as it's unique to the service within the issuer. Akka's mTLS support will verify that the DNS name supplied by an incoming connection matches the DNS name supplied in its own secret, and reject it otherwise. Again, we're using the naming convention for the `dnsName` mentioned above.

This certificate is configured to last for 24 hours, and rotate every 16 hours.

## Configuring Akka

To configure your Akka application, you need to have artery based remoting enabled, which will be the case if you've followed the Akka guide for [configuring cluster bootstrap in Kubernetes](https://doc.akka.io/docs/akka-management/current/kubernetes-deployment/forming-a-cluster.html), with the following additional configuration:

```
akka.remote.artery {
  transport = tls-tcp
  ssl.ssl-engine-provider = "akka.remote.artery.tcp.ssl.RotatingKeysSSLEngineProvider"
}
```

This instructs Akka to use TLS, with the `RotatingKeysSSLEngineProvider`, an SSL engine provider that is designed to pick up Kubernetes TLS secrets, and poll the file system for when they get rotated. It also applies authorization by matching the incoming DNS name with the DNS name of its own certificate.

## Configuring the Akka deployment

Having configured Akka and built a new Docker image, you can now configure your Akka deployment. To do this, you need to mount the certificate at the path `/var/run/secrets/akka-tls/rotating-keys-engine`. This is the default path that the `RotatingKeysSSLEngineProvider` uses to pick up its certificates. So, add the following volume to your pod:

```yaml
      volumes:
      - name: akka-tls
        secret:
          secretName: my-service-akka-tls-certificate
```

And then you can mount that in your container:

```yaml
        volumeMounts:
        - name: akka-tls
          mountPath: /var/run/secrets/akka-tls/rotating-keys-engine
```

Your complete deployment YAML, configured as described in our [Kubernetes deployment guide](https://doc.akka.io/docs/akka-management/current/kubernetes-deployment/preparing-for-production.html#deployment-spec), might look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
  namespace: default
  labels:
    app: my-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-service
  template:
    metadata:
      labels:
        app: my-service
    spec:
      containers:
      - name: my-service
        image: my-service:latest
        readinessProbe:
          httpGet:
            path: /ready
            port: management
        livenessProbe:
          httpGet:
            path: /alive
            port: management
        ports:
        - name: management
          containerPort: 8558
          protocol: TCP
        - name: http
          containerPort: 8080
          protocol: TCP
        resources:
          limits:
            memory: 1024Mi
          requests:
            cpu: 2
            memory: 1024Mi
        volumeMounts:
        - name: akka-tls
          mountPath: /var/run/secrets/akka-tls/rotating-keys-engine
      volumes:
      - name: akka-tls
        secret:
          secretName: my-service-akka-tls-certificate
```

And now you have secured your Akka cluster communication with mTLS using frequently rotated certificates. This will prevent both eavesdropping, as well as malicious services trying to join your Akka cluster.

Note that if you apply this to an existing running cluster deployment for the first time, you will need to do an Akka cluster restart. This is because the new nodes will be unable to connect to the old nodes because the new nodes will be attempting to speak TLS to the old nodes, while the old nodes will not be configured to speak TLS. The easiest way to restart the Akka cluster is to scale the deployment down to 0, and then back up to what it was before.

If you have more Akka services that you wish to deploy in the same namespace, you can reuse the same CA `Issuer`, you only need to deploy an additional `Certificate` for each service.
