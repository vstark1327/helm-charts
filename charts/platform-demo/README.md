# Platform Demo helm chart

This helm chart showcases the capabilities of SOGNO project via a demo. It currently implements Dpsim, Pyvolt and Pintura from the SOGNO project using Kafka as the interface.

## Preliminaries

Follow the instructions here to get started:
https://sogno-platform.github.io/docs/getting-started/

## Prerequisites

- k3s or Kubernetes cluster running
- Helm 3
- kubectl configured

### Helm Repos

Ensure that the following Helm Chart Repos are set up or add them locally:

```bash
helm repo add sogno https://sogno-platform.github.io/helm-charts

helm repo add strimzi https://strimzi.io/charts/
helm repo add influxdata https://influxdata.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### HugePages

The current setup requires HugePages support for the real-time simulator. This can be checked and activated (temporarily) as follows:

```bash
# Verify HugePages
cat /proc/meminfo | grep Huge

AnonHugePages:    104448 kB
ShmemHugePages:        0 kB
FileHugePages:         0 kB
HugePages_Total:       0		<-- we require a minimum of 1024
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB

# Increase No of HPgs
echo 1024 | sudo tee /proc/sys/vm/nr_hugepages

# Check it worked
cat /proc/meminfo | grep Huge

AnonHugePages:    104448 kB
ShmemHugePages:        0 kB
FileHugePages:         0 kB
HugePages_Total:    1024
HugePages_Free:     1024
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:         2097152 kB
```
If you don't see 1024 next to HugePages_Total, you may need to restart
your system and try again with a fresh boot.

```
# Restart k3s service to apply changes
sudo systemctl restart k3s

# Ensure the KUBECONFIG env is still set correctly
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

If this still does not work add
```
vm.nr_hugepages = 1024
```
at the top of `/etc/sysctl.conf`


### Building the `dpsim-kafka-service` image for dpsim

The `dpsim-kafka-service` is an image that wraps DPsim with a Kafka nodetype in VILLASNode. Build and push it to k3s before running the demo

### Building the `pyvolt-kafka-service` image for pyvolt

The `pyvolt-kafka-service` is an image that wraps PyVolt with a Kafka interface. Build and push it to k3s before running the demo

### Installing Strimzi Operator

```bash
helm install strimzi-operator strimzi/strimzi-kafka-operator \
 --namespace default \
 --wait
```

## Running the Demo

```bash
cd platform-demo/
helm install demo .
```

## Seeing the Demo
`grafana` is available at port 31230: http://localhost:31230

`pintura` is available at port 31234 (set in cim-editor/pintura-values.yaml or pyvolt-dpsim-demo/values.yaml depending on the demo you are running ): http://localhost:31234

## Cleanup 

### for the Demo

```bash
helm uninstall demo
```

### Optional: for Strimzi Operator
If you are done using the demo then you can uninstall the strimzi-operator and its related resources. 
```bash
# uninstall the operator
helm uninstall strimzi-operator

# delete the crd's created by the operator
kubectl delete crd \
$(kubectl get crd | grep strimzi | awk '{print $1}')
```

