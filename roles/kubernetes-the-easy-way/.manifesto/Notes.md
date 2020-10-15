sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt install -y kubelet=1.16.10-00 kubeadm=1.16.10-00
sudo apt-mark hold kubelet kubeadm kubectl


# ------------------------------
# Pseudocode
# ------------------------------

# installation
1. Kubernetes binaries
2. Container binaries, i would choose cri-o/crictl
3. CNI

# Step
==> bootstrap == true
1. Provisioning 1 master
2. Apply CNI manifest
3. Apply health-check manifest (Kubernetes Deployment nginx:alpine, Kubernetes Service NodePort (30000) for port 80)
    kubectl create deploy healthz --image nginx:alpine
    kubectl expose deploy healthz --target 80 --target-port 80 --type NodePort
    kubectl patch svc healthz  -p '{"spec": {"ports": [{"nodePort": 30000, "port": 80}]}}'

==> bootstrap == false (default) AND control-plane == true
1. Join the other 2 master node into provisioned master in previous step
2. Smoke test each master node
3. Failure node, listed and applied in the next chance

==> bootstrap == false (default) AND control-plane == false (default)
4. Add all worker nodes into control-plane
5. Smoke test each worker node
6. Failure node, listed and applied in the next chance

SMOKE TEST consist of:
1. Test health-check connectivity with node port via curl <node_ip_addr>:30000/
2. Test kubelet connectivity via nc -zv <node_ip_addr> 10250