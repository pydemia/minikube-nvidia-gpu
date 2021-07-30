# Minikube

## none, nvidia-docker

* https://minikube.sigs.k8s.io/docs/tutorials/nvidia_gpu/#using-the-none-driver
* https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#setting-up-nvidia-container-toolkit

### Setting up NVIDIA Container Toolkit

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
sudo apt-get install -y nvidia-docker2
```

Change default runtime to `nvidia`:
`/etc/docker/daemon.json`
```diff
{
++  "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

```bash
sudo systemctl restart docker
```

Test:

```bash
sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

## Run minikube

```bash
$ minikube start --kubernetes-version=v1.18.17 --driver=none --apiserver-ips 127.0.0.1 --apiserver-name localhost

😄  minikube v1.22.0 on Ubuntu 20.04
❗  Kubernetes 1.18.17 has a known performance issue on cluster startup. It might take 2 to 3 minutes for a cluster to start.
❗  For more information, see: https://github.com/kubernetes/kubeadm/issues/2395
✨  Using the none driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🤹  Running on localhost (CPUs=20, Memory=64324MB, Disk=937366MB) ...
ℹ️  OS release is Ubuntu 20.04.2 LTS
🐳  Preparing Kubernetes v1.18.17 on Docker 20.10.2 ...
    ▪ kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
    > kubectl.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubeadm.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubelet.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubeadm: 37.93 MiB / 37.93 MiB [---------------] 100.00% 1.84 MiB p/s 21s
    > kubectl: 41.95 MiB / 41.95 MiB [---------------] 100.00% 1.96 MiB p/s 22s
    > kubelet: 108.04 MiB / 108.04 MiB [------------] 100.00% 1.57 MiB p/s 1m9s
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🤹  Configuring local host environment ...

❗  The 'none' driver is designed for experts who need to integrate with an existing VM
💡  Most users should use the newer 'docker' driver instead, which does not require root!
📘  For more information, see: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

❗  kubectl and minikube configuration will be stored in /home/airuntime
❗  To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:

    ▪ sudo mv /home/airuntime/.kube /home/airuntime/.minikube $HOME
    ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

💡  This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner

❗  /usr/bin/kubectl is version 1.21.3, which may have incompatibilites with Kubernetes 1.18.17.
    ▪ Want kubectl v1.18.17? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

```bash
$ kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/master/nvidia-device-plugin.yml
daemonset.apps/nvidia-device-plugin-daemonset created
```

```bash
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
```
