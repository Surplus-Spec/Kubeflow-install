# Kubeflow-install

> [Kubernetes: 1.9.0](https://kubernetes.io/blog/2017/12/kubernetes-19-workloads-expanded-ecosystem/) <br/>[Docker: 18.06.2](https://docs.docker.com/engine/release-notes/18.06/)<br/>[Kubeflow: 1.4](https://github.com/kubeflow/manifests) <br/>[Knative: 0.22.1](https://github.com/knative/serving/releases?page=2) <br/> [Kustomize: 3.2.0](https://github.com/kubernetes-sigs/kustomize/releases/tag/v3.2.0) <br/>

### 설치 순서
#### 1. dynamic volume provisioner setup <br/> 2. Kustomize download<br/>3. Kubeflow install<br/>4. print Dashboard

------------
#### 1. dynamic volume provisioner setup
```
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
https://www.kangwoo.kr/2020/02/18/pc%EC%97%90-kubeflow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-3%EB%B6%80-kubeflow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/

#### 2. Kustomize download
[kustomize_3.2.0_linux_amd64](https://github.com/kubernetes-sigs/kustomize/releases/download/v3.2.0/kustomize_3.2.0_linux_amd64)



#### 3. Kubeflow install
```ubuntu
git clone https://github.com/kubeflow/manifests.git
mv kustomize_3.2.0_linux_amd64 manifasts
while ! ./kustomize build example | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```

#### 4. print Dashboard
```
kubectl port-forward svc/istio-ingressgateway -n istio-system 8888:80
firefox
- localhost.8888/ 
```

## etc
##### istio error
```
vi /etc/kubernetes/manifests/kube-apiserver.yaml

spec:
  containers:
  - command:
    ...
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key # insert
    - --service-account-issuer=kubernetes.default.svc # insert
    ...
    
```

##### Knative-serving error
```
kubectl delete ValidatingWebhookConfiguration  -l serving.knative.dev/release=v0.22.1
kubectl delete MutatingWebhookConfiguration  -l serving.knative.dev/release=v0.22.1
```
##### Kubernetes pods uninstall(namespace)
```
kubectl delete deployments,jobs,services,pods --all -n <namespace>
```
##### Kubernetes pod reinstall
```
kubectl get pod <pod_name> -n <namespace> -o yaml | kubectl replace --force -f - # one

kubectl delete pod -n <namespace> --all            # namespace
kubectl get pods -A | awk 'NR>1{print $1" "$2}' \
    | while read ns pod; do \
        kubectl delete -n $ns pod $pod; done
```

------------
## Ref
> https://blog.kubeflow.org/kubeflow-1.4-release/ Kubeflow 1.4 설명<br/>
> https://github.com/kubeflow/manifests/tree/master Kubeflow 1.4<br/>
> https://suwani.tistory.com/m/18 Kubeflow 1.4 설치<br/>
> https://1week.tistory.com/113 Kubeflow 1.4 설치<br/>
> https://mkbahk.medium.com/ubuntu-18-04%EC%83%81%EC%97%90-kubeflow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-496a36c906f9 Kubeflow 설치<br/>
> https://cloudarchitecture.tistory.com/43 istio-system error 해결<br/>
> https://yjwang.tistory.com/68 Kubeflow install 에러 해결<br/>
> https://stackoverflow.com/questions/40686151/kubernetes-pod-gets-recreated-when-deleted Kubernetes pods uninstall<br/>
