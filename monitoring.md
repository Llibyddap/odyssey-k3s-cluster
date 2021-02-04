ssh into k3s_master01

apt-get update && apt-get install -y build-essential golang

git clone https://github.com/carlosedp/cluster-monitoring.git
see (https://github.com/carlosedp/cluster-monitoring)

cd cluster-monitoring

nano vars.jssonnet

To deploy the monitoring stack on your K3s cluster, there are four parameters that need to be configured in the vars.jsonnet file:

Set k3s.enabled to true.
Change your K3s master node IP(your VM or host IP) on k3s.master_ip parameter.
Edit suffixDomain to have your node IP with the .nip.io suffix or your cluster URL. This will be your ingress URL suffix.
Set traefikExporter enabled parameter to true to collect Traefik metrics and deploy dashboard.

make vendor
make
kubectl apply -f manifests/setup/
kubectl apply -f manifests/
