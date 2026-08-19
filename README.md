# Kubernetes Practice

한국폴리텍대학 **Kubernetes 실습을 위한 예제 저장소**입니다.

VirtualBox 등의 가상머신 환경에 Kubernetes 클러스터를 구성한 후,
Pod부터 Deployment, Service, LoadBalancer, Ingress까지 Kubernetes의 핵심 기능을 단계적으로 실습합니다.

---

## 🎯 학습 목표

본 실습을 통해 다음 내용을 학습합니다.

* Kubernetes 기본 구조 이해
* Pod 생성 및 관리
* Multi-Container Pod 구성
* ReplicaSet을 이용한 Pod 복제
* Deployment를 이용한 애플리케이션 배포
* Service를 이용한 Pod 네트워크 연결
* ClusterIP / NodePort / LoadBalancer 비교
* MetalLB를 이용한 Bare-metal LoadBalancer 구성
* Ingress Controller를 이용한 HTTP 서비스 노출

---

## 📁 Repository 구조

```text
kopoK8sPractice
│
├── Practice
│   ├── ex01
│   │   ├── first-deploy.yml
│   │   ├── first-deploy-hc.yml
│   │   └── first-deploy-lp.yml
│   │
│   ├── ex02
│   │   └── multi-container.yml
│   │
│   ├── ex03-replica
│   │   └── repltest.yml
│   │
│   ├── ex04-deployment
│   │   ├── deploytest.yml
│   │   └── deploytest2.yml
│   │
│   ├── ex05-clusterIp
│   │   └── clusterip.yml
│   │
│   ├── ex06-nodeport
│   │   └── nodeport.yml
│   │
│   ├── ex07-loadbalancer
│   │   ├── loadbalancer.yml
│   │   ├── metallb-native.yaml
│   │   ├── metallb-config.yaml
│   │   ├── _2_IPAddrPool.yaml
│   │   └── _3_2Advertise.yaml
│   │
│   └── ex08-ingress
│       ├── basic.yml
│       ├── deploy.yaml
│       ├── ingress.yml
│       └── old_deploy.yaml
│
├── k8s practice 2026.pdf
├── LICENSE
└── README.md
```

---

# 🧪 실습 순서

## 1. Pod 기본 실습

```text
Practice/ex01
```

Kubernetes의 가장 기본적인 실행 단위인 **Pod**를 생성하고 관리합니다.

주요 학습 내용

* Pod 생성
* YAML Manifest 구조
* Container 실행
* Liveness Probe
* Readiness Probe
* Pod 상태 확인

예제 실행:

```bash
cd Practice/ex01

kubectl apply -f first-deploy.yml
```

Pod 확인:

```bash
kubectl get pods
```

상세 정보 확인:

```bash
kubectl describe pod <POD_NAME>
```

삭제:

```bash
kubectl delete -f first-deploy.yml
```

---

## 2. Multi-Container Pod

```text
Practice/ex02
```

하나의 Pod 안에서 **둘 이상의 Container를 실행하는 방법**을 실습합니다.

```bash
cd Practice/ex02

kubectl apply -f multi-container.yml
```

확인:

```bash
kubectl get pods
```

Pod 내부 Container 확인:

```bash
kubectl describe pod <POD_NAME>
```

특정 Container 로그 확인:

```bash
kubectl logs <POD_NAME> -c <CONTAINER_NAME>
```

---

## 3. ReplicaSet

```text
Practice/ex03-replica
```

ReplicaSet을 사용하여 동일한 Pod를 여러 개 실행하고 원하는 개수를 유지하는 방법을 학습합니다.

```bash
cd Practice/ex03-replica

kubectl apply -f repltest.yml
```

확인:

```bash
kubectl get pods
kubectl get replicasets
```

또는 축약형으로:

```bash
kubectl get rs
```

ReplicaSet은 지정된 수의 Pod가 항상 실행되도록 관리합니다.

```text
ReplicaSet
   │
   ├── Pod
   ├── Pod
   └── Pod
```

---

## 4. Deployment

```text
Practice/ex04-deployment
```

Deployment를 이용하여 ReplicaSet과 Pod를 관리하는 방법을 실습합니다.

```bash
cd Practice/ex04-deployment

kubectl apply -f deploytest.yml
```

확인:

```bash
kubectl get deployments
kubectl get replicasets
kubectl get pods
```

또는:

```bash
kubectl get all
```

Kubernetes의 일반적인 애플리케이션 관리 구조는 다음과 같습니다.

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
   Pods
```

Deployment를 이용하면 다음 기능을 사용할 수 있습니다.

* Pod 복제
* Scale Out / Scale In
* Rolling Update
* Rollback
* 애플리케이션 버전 관리

Scale 변경 예:

```bash
kubectl scale deployment <DEPLOYMENT_NAME> --replicas=3
```

---

# 🌐 Kubernetes Service

Pod의 IP 주소는 Pod가 다시 생성될 때 변경될 수 있습니다.

따라서 Kubernetes에서는 **Service**를 이용하여 여러 Pod에 대해 일정한 접근 경로를 제공합니다.

```text
Client
  │
  ▼
Service
  │
  ├── Pod
  ├── Pod
  └── Pod
```

---

## 5. ClusterIP

```text
Practice/ex05-clusterIp
```

ClusterIP는 Kubernetes Service의 기본 타입입니다.

```bash
cd Practice/ex05-clusterIp

kubectl apply -f clusterip.yml
```

확인:

```bash
kubectl get services
```

또는:

```bash
kubectl get svc
```

구조:

```text
Kubernetes Cluster

Client Pod
    │
    ▼
ClusterIP Service
    │
    ├── Pod
    ├── Pod
    └── Pod
```

ClusterIP는 기본적으로 **Kubernetes 클러스터 내부에서 접근하기 위한 Service**입니다.

---

## 6. NodePort

```text
Practice/ex06-nodeport
```

NodePort는 Kubernetes Node의 특정 포트를 외부에 공개합니다.

```bash
cd Practice/ex06-nodeport

kubectl apply -f nodeport.yml
```

확인:

```bash
kubectl get svc
```

구조:

```text
Client
   │
   ▼
NodeIP:NodePort
   │
   ▼
Service
   │
   ▼
Pods
```

접속 형태:

```text
http://<NODE_IP>:<NODE_PORT>
```

NodePort는 기본적으로 다음 범위의 포트를 사용합니다.

```text
30000 ~ 32767
```

---

## 7. LoadBalancer

```text
Practice/ex07-loadbalancer
```

일반적인 Cloud 환경에서는 AWS, Azure, GCP 등의 Load Balancer와 Kubernetes Service를 연동할 수 있습니다.

하지만 VirtualBox와 같은 **Bare-metal 환경에는 Cloud Load Balancer가 존재하지 않기 때문에 MetalLB를 이용하여 LoadBalancer 기능을 구현**할 수 있습니다.

구조:

```text
Client
   │
   ▼
External IP
   │
   ▼
LoadBalancer Service
   │
   ▼
Pods
```

### MetalLB

실습 디렉터리에는 MetalLB 구성을 위한 YAML 파일이 포함되어 있습니다.

```text
metallb-native.yaml
_2_IPAddrPool.yaml
_3_2Advertise.yaml
loadbalancer.yml
```

MetalLB 설치 후 IP Pool과 Advertisement를 설정합니다.

예:

```bash
kubectl apply -f metallb-native.yaml
```

MetalLB Pod 확인:

```bash
kubectl get pods -n metallb-system
```

IP Address Pool 설정:

```bash
kubectl apply -f _2_IPAddrPool.yaml
```

L2 Advertisement 설정:

```bash
kubectl apply -f _3_2Advertise.yaml
```

LoadBalancer Service 생성:

```bash
kubectl apply -f loadbalancer.yml
```

External IP 확인:

```bash
kubectl get svc
```

예:

```text
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)
my-nginx   LoadBalancer   10.100.10.20    192.168.56.200    80:xxxxx/TCP
```

웹 브라우저에서 다음과 같이 접근할 수 있습니다.

```text
http://192.168.56.200
```

> IP 주소는 각 실습 환경의 MetalLB IP Pool 설정에 따라 달라질 수 있습니다.

---

# 🚪 8. Ingress

```text
Practice/ex08-ingress
```

Ingress는 여러 HTTP/HTTPS 서비스를 하나의 진입점으로 관리하기 위해 사용합니다.

예를 들어:

```text
                         ┌── Service A ── Pods
Client ── Ingress ───────┤
                         └── Service B ── Pods
```

또는 Host를 이용하여 다음과 같이 서비스를 구분할 수 있습니다.

```text
app1.example.com ──┐
                   ├── Ingress Controller
app2.example.com ──┘
```

실습 파일:

```text
basic.yml
deploy.yaml
ingress.yml
```

애플리케이션 배포:

```bash
kubectl apply -f deploy.yaml
```

Ingress 생성:

```bash
kubectl apply -f ingress.yml
```

확인:

```bash
kubectl get ingress
```

Ingress Controller 확인:

```bash
kubectl get pods -n ingress-nginx
```

Service 확인:

```bash
kubectl get svc -n ingress-nginx
```

---

# 🏗️ 전체 실습 흐름

이 저장소의 실습은 Kubernetes 객체의 관계를 단계적으로 이해하도록 구성되어 있습니다.

```text
Container
    │
    ▼
   Pod
    │
    ▼
ReplicaSet
    │
    ▼
Deployment
    │
    ▼
 Service
    │
    ├── ClusterIP
    │
    ├── NodePort
    │
    └── LoadBalancer
            │
            ▼
         Ingress
```

실제 애플리케이션에서는 일반적으로 다음과 같은 구조로 사용됩니다.

```text
                       Internet
                          │
                          ▼
                       Ingress
                          │
                          ▼
                        Service
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
            Pod          Pod          Pod
```

---

# 🔧 자주 사용하는 kubectl 명령어

### 전체 Resource 확인

```bash
kubectl get all
```

### Node 확인

```bash
kubectl get nodes
```

자세히 확인:

```bash
kubectl get nodes -o wide
```

### Pod 확인

```bash
kubectl get pods
```

```bash
kubectl get pods -o wide
```

### Deployment 확인

```bash
kubectl get deployments
```

### ReplicaSet 확인

```bash
kubectl get replicasets
```

또는:

```bash
kubectl get rs
```

### Service 확인

```bash
kubectl get services
```

또는:

```bash
kubectl get svc
```

### Ingress 확인

```bash
kubectl get ingress
```

### Resource 상세 정보

```bash
kubectl describe pod <POD_NAME>
```

### Pod 로그

```bash
kubectl logs <POD_NAME>
```

### YAML 적용

```bash
kubectl apply -f <FILE_NAME>.yml
```

### YAML Resource 삭제

```bash
kubectl delete -f <FILE_NAME>.yml
```

---

# 💡 kubectl Alias

반복적으로 `kubectl`을 입력하는 것이 번거로운 경우 alias를 사용할 수 있습니다.

```bash
alias k=kubectl
```

이후 다음 명령:

```bash
kubectl get pods
```

을 다음과 같이 사용할 수 있습니다.

```bash
k get pods
```

예:

```bash
k get all
k get nodes
k get pods
k get svc
k get ingress
```

---

# 📚 실습 자료

본 저장소의 Kubernetes 실습 교안은 다음 파일을 참고합니다.

```text
k8s practice 2026.pdf
```

GitHub 저장소에서 PDF를 직접 열거나 다운로드하여 실습 자료와 함께 사용할 수 있습니다.

---

# ⚠️ 참고 사항

본 Repository는 **Kubernetes 학습 및 교육 실습용**으로 작성되었습니다.

사용하는 Kubernetes 버전, Container Runtime, CNI Plugin, MetalLB 및 Ingress Controller 버전에 따라 일부 명령어나 YAML 설정이 달라질 수 있습니다.

특히 LoadBalancer 및 Ingress 실습은 클러스터의 네트워크 환경에 따라 IP 주소 및 접속 방법을 적절하게 변경해야 합니다.

---

# 📜 License

This project is licensed under the **Apache License 2.0**.

자세한 내용은 Repository의 `LICENSE` 파일을 참고하시기 바랍니다.
