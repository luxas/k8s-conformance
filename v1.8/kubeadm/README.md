### kubeadm

Official documentation:
 - https://kubernetes.io/docs/setup/independent/install-kubeadm/
 - https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

#### How to reproduce:

Operating System used for these sample reproduction steps: Ubuntu 17.10
These commands are run as the `root` user.

```bash
# Install kubeadm
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubelet kubeadm kubectl docker.io

# Set up an one-master cluster
kubeadm init
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://git.io/weave-kube-1.6
kubectl taint nodes --all node-role.kubernetes.io/master-
curl -L https://raw.githubusercontent.com/cncf/k8s-conformance/master/sonobuoy-conformance.yaml | kubectl apply -f -

# Wait for `kubectl -n sonobuoy logs sonobuoy` to display "no-exit was specified, sonobuoy is now blocking"
while [[ $(kubectl -n sonobuoy logs sonobuoy | grep "sonobuoy is now blocking") == "" ]]; do sleep 1; done
# Copy out the results tarball from the container
kubectl -n sonobuoy cp sonobuoy:$(kubectl -n sonobuoy logs sonobuoy | grep "Results available" | rev | awk '{print $1}' | rev | sed 's/"//g') sonobuoy.tar.gz
tar -xzf sonobuoy.tar.gz
# Sonobuoy results available!
cp plugins/e2e/results/{e2e.log,junit_01.xml} .
kubectl version > version.txt
# Then check in e2e.log, junit_01.xml and version.txt into source control
```
