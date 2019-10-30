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
* Modify the key `cluster_name` in `manifests/telegraf-ds/10-telegraf-configuration.yaml` 
* Modify the key `cluster_name` in `manifests/telegraf-as/10-telegraf-configuration.yaml` 

4. Create a secret containing InfluxDB login details running `kubectl create secret -n=monitoring generic telegraf-secrets --from-literal=db=your-db --from-literal=pw=your-pw --from-literal=url=https://your-influx-server.com:8086 --from-literal=user=your-user`

4. Deploy Telegraf daemonSet with `kubectl apply -f manifests/telegraf-ds`

5. Deploy Telegraf deployment with `kubectl apply -f manifests/telegraf-sa`


Note about having 2 different Telegraf replicaSets: We use Telegraf deployed as a DaemonSet (i.e: running in each node) alongside with a Deployment of Telegraf with 1 single replica. They do different things, the former gets data from the node, like system metrics or Docker container metrics, the later queries APIs with aggregated data. We wouldn't want to be queriying kube-state-metrics (cluser level object metrics aggregator) from every node, a thus duplicating metrics on Influx.
