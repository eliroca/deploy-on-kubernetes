### Deploy [Bitpoll](https://github.com/fsinfuhh/Bitpoll) to kubernetes

[More info here](https://www.linode.com/docs/kubernetes/deploy-container-image-to-kubernetes)

#### Create a namespace
  - `kubectl create -f namespace.yaml`
  - `kubectl get namespaces`

#### Create a service
  - `kubectl create -f service.yaml`
  - `kubectl get service -n bitpoll-ns`

#### Optional: Create a secret with docker credentials to pull a private image
  - [More info here](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry)
  - `kubectl create secret generic regcred --from-file=.dockerconfigjson=/home/eroca/.docker/config.json --type=kubernetes.io/dockerconfigjson -n bitpoll-ns`

#### Hacky solution: PersistentVolume for database storage on the node itself

##### Persistent Volume for the database
- `kubectl apply -f persistent-volume.yaml`
- `kubectl describe pv bitpoll-pv -n bitpoll-ns`
- `kubectl get pv bitpoll-pv -n bitpoll-ns`

##### Persistent Volume Claim for the database
- `kubectl apply -f persistent-volume-claim.yaml`
- `kubectl describe pvc bitpoll-pv-claim -n bitpoll-ns`
- `kubectl get pv bitpoll-pv -n bitpoll-ns`

##### Create a privileged pod with root access and create the directory structure
  - `kubectl create -f privileged-pod.yaml`
  - `kubectl exec -ti privileged-pod sh`
  - `mkdir -p /host/mnt/bitpoll/database` as configured in the PersistentVolume

#### Create ConfigMaps
  - `kubectl create -f bitpoll-settings-local-configmap.yaml -n bitpoll-ns`

#### Create a deployment
  - `kubectl create -f deployment.yaml`
  - `kubectl get deployment -n bitpoll-ns`

#### Find out on which port the app is running
  - `kubectl get services -n bitpoll-ns`

#### Open a browser
  - `firefox http://10.86.3.237:31466`
  - 10.86.3.237 is my worker node's IP

#### Troubleshooting crashloopbackoff
  - [More info here](https://managedkube.com/kubernetes/pod/failure/crashloopbackoff/k8sbot/troubleshooting/2019/02/12/pod-failure-crashloopbackoff.html)
  - Find out pod name:
    `kubectl get pods -n bitpoll-ns`
  - Look through logs of the pod:
    `kubectl logs -f bitpoll-deployment-54fb99ff79-l46m4 -n bitpoll-ns`
