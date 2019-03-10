# Kubernetes_Deploy_Projects

Docker Image로 만들어진 Deep Learning 분석 서비스를 CI/CD를 고려하여 딥러닝 모델 배포후 관리를 하기 위한 연습 프로젝트입니다. Dockerhub에서 Deep Learning 분석 서비스와 관련된 docker image들을 관리하고, Kubernetes를 이용하여 Deep Learning 분석 서비스를 자동으로 배포하고 그에 필수자원인 GPU 및 CPU, RAM 등을 실시간으로 monitoring 합니다. 또한 지속적으로 쌓인 데이터를 Deep Learning 학습 모델이 주기적으로 재학습하고, 재학습된 모델들을 이전 학습 모델들과 Ranking하여, 분석에 있어 최적의 모델이 언제나 Runtime에 올라가 있도록 합니다.

* Project List
  * Project 1 : [Human Pose MET Score Estimation](https://github.com/Hahnnz/Human_Pose_MET_Score_Estimation) Deployment
  * Project 2 : [FAST NLP Analysis](https://github.com/cocoanlab/fast_nlp) Deployment

> 해당 Project는 향후 영어로 번역될 예정입니다.

## Basic Setting & Installation

### Docker and Nvidia-docker Installation & Setting

### Kubernetes Installation & Setting

* Master node 
  - eta
* Worker node 
  - delta, chi

[Creating a single master cluster with kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

#### Maser node setting 

```
$ sudo kubeadm --kubernetes-version=1.13.2 init
```

init을 실행하면 worker node 에서 master node로 접속할 수 있는 token hash 값을 내준다. 이것을 이용해 master node에 접속한다.

`ex) kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`

만약 다음과 같이 swap 에러가 발생한다면,
```
[ERROR Swap]: running with swap on is not supported. Please disable swap
```
```
$ sudo swapoff -a
```
명령으로 swap을 꺼주면 된다.

To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Installing a pod network add-on

문서에 보면 여러가지 network를 설정할 수 있는 것들이 존재하는데 우리는 [Canal](https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/flannel)을 사용하였다.
 
```
$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```

잘 작동하고 있는지 pod을 확인하자.

```
$ kubectl get pods -n kube-system

NAME                          READY   STATUS              RESTARTS   AGE
canal-vb5fw                   0/3     ContainerCreating   0          27s
coredns-86c58d9df4-mf68s      0/1     Pending             0          2s
coredns-86c58d9df4-zbxnp      0/1     Pending             0          2s
etcd-eta                      1/1     Running             0          26s
kube-apiserver-eta            1/1     Running             0          45s
kube-controller-manager-eta   1/1     Running             0          33s
kube-proxy-tww4r              1/1     Running             0          82s
kube-scheduler-eta            1/1     Running             0          24s
```

만약 coredns가 fail이 나거나 error가 발생하면 다음 설정으로 해결할 수 있다.
```
$ kubectl -n kube-system edit configmap coredns
loop 삭제
$ kubectl -n kube-system delete pod -l k8s-app=kube-dns
```

#### Joining your nodes

아까 master node에서 init할 때 나온 token hash값을 이용해서 node들을 join 시켜보자.

worker node에서는 init 할 필요없이 바로 join 명령어만 실행 하면된다.

```
$ kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

master에서 확인해보면,
```
$ kubectl get pods -n kube-system

NAME                          READY   STATUS    RESTARTS   AGE
canal-6gg89                   3/3     Running   0          24m
canal-7dhgw                   3/3     Running   0          24m
canal-vb5fw                   3/3     Running   0          24m
coredns-86c58d9df4-mf68s      1/1     Running   0          24m
coredns-86c58d9df4-zbxnp      1/1     Running   0          24m
etcd-eta                      1/1     Running   0          24m
kube-apiserver-eta            1/1     Running   0          25m
kube-controller-manager-eta   1/1     Running   0          24m
kube-proxy-57qh5              1/1     Running   0          24m
kube-proxy-tww4r              1/1     Running   0          25m
kube-proxy-x6rng              1/1     Running   0          24m
kube-scheduler-eta            1/1     Running   0          24m
```

node 개수 만큼 canal, proxy등이 추가 된것을 확인 할 수 있다.

```
$ kubectl get nodes

NAME    STATUS   ROLES    AGE   VERSION
chi     Ready    <none>   14m   v1.13.2
delta   Ready    <none>   14m   v1.13.2
eta     Ready    master   16m   v1.13.2
```

### Our CI 

## Work-Flow
