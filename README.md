kubernetes-to-influx

This repository contains a set of Kubernetes manifests for enabling the collection of Kubernetes cluster level and container level metrics, and for sending them to InfluxDB.

There are 3 components needed:
1. [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics): Talks to the Kubernetes API for getting the status of the objects in the cluster and exposes this information in Prometheus format.
2. [metrics-server](https://github.com/kubernetes-incubator/metrics-server): Aggregates cluster-wide resource usage data coming from each Kubelet, and exposes it in Prometheus format
3. [Telegraf](https://github.com/influxdata/telegraf): Fetches the data exposed by `kube-state-metrics` and `metrics-server`, and it sends it to an InfluxDB server.

## Deploying
1. Deploy `kube-state-metrics` running `kubectl apply -f manifests/kube-state-metrics`

2. Deploy `metrics-server` running `kubectl apply -f manifests/metrics-server`

3. Before deploying Telegraf you have to customize its configuration file to add a cluster name, and create a secret with the right InfluxDB URL, user and password.
* Modify the key `cluster_name` in `manifests/telegraf-ds/20-telegraf-configuration.yaml` 
* Create the secret following the template in `manifests/telegraf-ds/30-secret.yaml`
* Deploy Telegraf running `kubectl apply -f manifests/telegraf-ds`
