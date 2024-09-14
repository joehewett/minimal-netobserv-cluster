# Minimal Kubernetes + Netobserv + Loki + Grafana Setup 

This is a minimal setup for Netobserv, Loki and Grafana on Kubernetes. It is intended to be used for testing and development purposes.

## Prerequisites
- A Kubernetes cluster, run `./k3s.sh` to bring up a local cluster.
- Read https://github.com/netobserv/netobserv-ebpf-agent/tree/main/deployments 

## Setup

Apply the permissions:

```bash
kubectl apply -f ./perms.yml
```

Then, create deploy loki:
```bash
curl -S -L https://raw.githubusercontent.com/netobserv/documents/main/examples/zero-click-loki/1-storage.yaml | kubectl create -n netobserv -f - 
curl -S -L https://raw.githubusercontent.com/netobserv/documents/main/examples/zero-click-loki/2-loki.yaml | kubectl create -n netobserv -f -
```

Finally bring up eBPF and FLP:
```bash
kubectl apply -f ./flp-service.yml
```

## Accessing Grafana

To access Grafana, apply the config in `grafana.yml`:
```bash
kubectl apply -f ./grafana.yml
```

Then, port forward the Grafana service:

```bash
kubectl port-forward -n netobserv service/grafana 3000:3000
```

Then, open your browser and go to `http://localhost:3000`. You should be able to add the Loki datasource with `http://loki:3100` as the URL.

## Generating Traffic

To generate traffic, you can use the `busybox` image:

You can create a deployment of the nginx image:
```bash
kubectl apply -f ./nginx-test.yml
```

Then, port forward the service:
```bash
kubectl port-forward service/nginx 8080:80
```

Then, you can use `curl` to generate traffic:
```bash
curl http://localhost:8080
```

Alternatively you can exec into the pod and run `curl`:
```bash
kubectl exec -it <pod-name> -- /bin/sh
curl 1.1.1.1
```

These connection logs should show up in Grafana.
