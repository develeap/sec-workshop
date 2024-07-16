# Develeap Basic K8s Security

## Set Up Kind with Calico

Use the kind_config.yaml file to create a kind cluster with the default CNI disabled and seccomp profiles loaded.

```bash
cd 00_kind-with-calico && kind create cluster --config=kind_config.yaml
```

Install Calico:

```bash
chmod +x 00_kind-with-calico/calico_install.sh && 00_kind-with-calico/calico_install.sh
```

## Task 1: Security Context

Create the `secure` namespace:

```bash
kubectl apply -f 01_PodSecurityContext/namespace_open.yaml
```

Create the "bad" and "secure" pods:

```bash
kubectl apply -f 01_PodSecurityContext/pod_bad.yaml
kubectl apply -f 01_PodSecurityContext/pod_secure.yaml
```

Your task is to put a security standard in place that will disallow the `bad` pod from being created.
Note: The pod will not be destroyed if already created - destroy and recreate to see this take effect.

## Task 2: Network Policy

Create the `app-a`, `app-b`, and `app-c` pods:

```bash
kubectl apply -f 02_NetworkPolicy/apps.yaml
```

You task is to put policies in place so that traffic from `app-a` can reach `app-b`, but not `app-c`.

## Task 3: Seccomp Profiles

Create `default-pod`:

```bash
kubectl apply -f 03_seccomp-profiles/pod_default.yaml
```

Task: Experiment changing the seccomp profile from `RuntimeDefault` to `Localhost`.
Tip: Examine the `profiles` folder in the repo to see the seccomp profiles available.
Localhost Profiles will be located under `profiles/<profile name>`.

## Task 4 (Demo) Image Signing with `cosign`

Install `cosign`:

```bash
chmod +x 04_image-signing/cosign_install.sh && 04_image-signing/cosign_install.sh
```

Generate a key pair:

```bash
cosign generate-key-pair
```

Pull the latest Alpine image:

```bash
docker pull alpine:latest
```

Attempt to sign an image where we have no push permissions:

```bash
cosign sign --key cosign.key alpine:latest
```

Attempt to sign an image that's not been pushed:

```bash
docker tag alpine:latest fatliverfreddny/contenttrust:not_uploaded
cosign sign --key cosign.key fatliverfreddny/contenttrust:not_uploaded
```

Retag the Alpine image and push:

```bash
docker tag alpine:latest fatliverfreddny/contenttrust:signed
docker push fatliverfreddny/contenttrust:signed
```

Now this is done, proceed to sign the image:

```bash
cosign sign --key cosign.key fatliverfreddny/contenttrust:signed
```

Verify the signature:

```bash
cosign verify --key cosign.pub fatliverfreddny/contenttrust:signed | jq .
```

Now let's modify the image without signing it, and attempt to verify the signature:

```bash
docker pull alpine:3.19
docker tag alpine:3.19 fatliverfreddny/contenttrust:signed
docker push fatliverfreddny/contenttrust:signed
cosign verify --key cosign.pub fatliverfreddny/contenttrust:signed | jq .
```