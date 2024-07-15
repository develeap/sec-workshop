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
