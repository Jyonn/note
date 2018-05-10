# 在istio上运行Hello World样例
### minikube v1.9.4(server) + kubectl v1.9.6(client) on macOS

## 安装istio
```
# install kubectl
brew install kubectl
# install minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.25.2/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

# check minikube
minikube start
kubectl version

# downlaod istio
curl -L https://git.io/getLatestIstio | sh -

# get into env
cd istio-0.6
export PATH=$PWD/bin:$PATH

# install istio
kubectl apply -f install/kubernetes/istio.yaml

# check istio
kubectl get svc -n istio-system

# install istio sidecar
./install/kubernetes/webhook-create-signed-cert.sh \
    --service istio-sidecar-injector \
    --namespace istio-system \
    --secret sidecar-injector-certs
kubectl apply -f install/kubernetes/istio-sidecar-injector-configmap-release.yaml
cat install/kubernetes/istio-sidecar-injector.yaml | \
     ./install/kubernetes/webhook-patch-ca-bundle.sh > \
     install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml
kubectl apply -f install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml

# check istio sidecar
kubectl get pods -n istio-system
```

## 运行helloworld
### create helloworld namespace
```
cat hw-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helloworld
  labels:
    name: helloworld
```
```
kubectl create -f hw-namespace.yaml 
```

### manual injection
```
cd ./sample/helloworld
istioctl kube-inject -f helloworld.yaml -o helloworld-istio.yaml
kubectl create -f helloworld-istio.yaml -n helloworld
```

```
istioctl kube-inject -f disk.yaml --includeIPRanges=10.0.0.1/24 -o disk-istio.yaml
```

### auto injection
```
kubectl label namespace helloworld istio-injection=enabled

# check injection
kubectl get namespace -L istio-injection
kubectl create -f helloworld.yaml -n helloworld
```

### run helloworld
```
# check pods
kubectl get pods -n helloworld

# get helloworld url
export HELLOWORLD_URL=$(kubectl -n istio-system get po -l istio=ingress -o 'jsonpath={.items[0].status.hostIP}'):$(kubectl -n istio-system get svc istio-ingress -o 'jsonpath={.spec.ports[0].nodePort}')

# check result
curl http://$HELLOWORLD_URL/hello
```
