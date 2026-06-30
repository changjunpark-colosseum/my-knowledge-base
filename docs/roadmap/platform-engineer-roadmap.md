# 플랫폼 엔지니어 로드맵

목표: AWS, Docker, Kubernetes, Argo CD, GitOps, CNCF 생태계를 다루는 플랫폼 엔지니어가 되고, CKA 자격증까지 취득한다.

이 로드맵은 아무것도 모르는 상태에서 시작하는 것을 기준으로 한다. 핵심은 한 번에 많은 기술을 외우는 것이 아니라, 작은 서비스를 직접 배포하고 운영하면서 기술을 연결하는 것이다.

## 최종 목표

6-9개월 뒤 아래를 할 수 있으면 좋은 출발점에 도달한 것이다.

- Linux 서버에 접속해 프로세스, 포트, 로그, 디스크 상태를 확인할 수 있다.
- AWS에서 VPC, EC2, S3, IAM, ALB, CloudWatch의 역할을 설명하고 간단히 구성할 수 있다.
- Dockerfile을 작성하고 이미지를 빌드, 실행, push할 수 있다.
- Kubernetes에서 Deployment, Service, Ingress, ConfigMap, Secret, PVC를 사용할 수 있다.
- 장애가 난 Pod를 `kubectl`로 진단하고 복구할 수 있다.
- Argo CD로 Git 저장소의 manifest를 Kubernetes 클러스터에 배포할 수 있다.
- CKA 시험 범위에 맞춰 클러스터 운영, 네트워킹, 스토리지, 트러블슈팅 문제를 풀 수 있다.

## 전체 학습 순서

1. 컴퓨터/운영체제/네트워크 기초
2. Git, GitHub, CLI 사용법
3. AWS 기초
4. Docker와 컨테이너
5. Kubernetes 기본과 운영
6. Helm, Kustomize, manifest 관리
7. Argo CD와 GitOps
8. 관측성, 보안, 비용, 장애 대응
9. CKA 집중 준비

Kubernetes를 바로 시작하지 말고, Linux, 네트워크, Docker를 먼저 익힌다. Kubernetes는 이 개념들이 합쳐진 운영 플랫폼이기 때문에 기초가 없으면 모든 에러가 낯설어진다.

## 1단계: 기초 체력 만들기

기간: 3-4주

### Linux

배울 것:

- 파일/디렉토리 명령어: `pwd`, `ls`, `cd`, `cp`, `mv`, `rm`, `mkdir`
- 파일 확인: `cat`, `less`, `head`, `tail`
- 검색: `grep`, `find`, `rg`
- 권한: `chmod`, `chown`
- 프로세스: `ps`, `top`, `kill`
- 서비스/로그: `systemctl`, `journalctl`
- 디스크/메모리: `df`, `du`, `free`

실습:

- Ubuntu VM 또는 EC2 한 대를 만들고 SSH 접속한다.
- Nginx를 설치하고 브라우저에서 접속한다.
- Nginx 로그를 확인한다.
- 포트가 열려 있는지 확인한다.

### 네트워크

배울 것:

- IP, subnet, gateway
- DNS
- HTTP/HTTPS
- TLS 인증서
- port
- load balancer
- NAT
- firewall/security group

실습:

- `curl`로 HTTP 응답 확인하기
- `dig` 또는 `nslookup`으로 DNS 확인하기
- `nc` 또는 `telnet`으로 포트 확인하기
- 브라우저 요청이 서버까지 가는 흐름 그림으로 그려보기

### Git

배울 것:

- `git clone`
- `git status`
- `git add`
- `git commit`
- `git branch`
- `git checkout`
- `git pull`
- `git push`
- Pull Request 흐름

실습:

- GitHub에 `platform-engineering-lab` 저장소를 만든다.
- 실습 명령어와 에러 해결 과정을 Markdown으로 정리한다.

## 2단계: AWS 기초

기간: 4주

공식 학습 자료:

- AWS Cloud Practitioner Essentials: https://aws.amazon.com/training/digital/aws-cloud-practitioner-essentials/
- AWS Skill Builder: https://aws.amazon.com/training/digital/

배울 것:

- IAM: user, role, policy
- VPC: subnet, route table, internet gateway, NAT gateway
- EC2: instance, AMI, security group, key pair
- S3: bucket, object, policy
- RDS: managed database 개념
- ALB: HTTP load balancing
- CloudWatch: metric, log, alarm
- 비용 관리: Budget, Cost Explorer

실습:

1. AWS Budget을 먼저 설정한다.
2. EC2 한 대를 만든다.
3. Security Group으로 22, 80 포트를 제어한다.
4. EC2에 Nginx를 설치한다.
5. ALB를 붙여 HTTP로 접속한다.
6. CloudWatch에서 로그와 메트릭을 확인한다.

주의:

- 처음부터 EKS를 만들지 않는다. 비용과 설정 복잡도가 높다.
- NAT Gateway는 비용이 빠르게 발생할 수 있으므로 실습 후 삭제한다.
- 계정 루트 사용자는 사용하지 않고 IAM 사용자를 만든다.

## 3단계: Docker와 컨테이너

기간: 3-4주

배울 것:

- image와 container 차이
- Dockerfile
- build context
- layer cache
- volume
- network
- environment variable
- Docker Compose
- image registry

실습 프로젝트:

간단한 웹 애플리케이션을 만든다.

예시 구성:

- frontend 또는 backend 앱 하나
- PostgreSQL 또는 MySQL
- Dockerfile
- `docker-compose.yml`

해야 할 일:

1. 로컬에서 앱을 실행한다.
2. Dockerfile을 작성한다.
3. 이미지를 빌드한다.
4. 컨테이너로 실행한다.
5. Compose로 앱과 DB를 같이 실행한다.
6. 이미지를 Docker Hub 또는 AWS ECR에 push한다.

완료 기준:

- `docker build`로 이미지가 만들어진다.
- `docker run`으로 앱이 뜬다.
- Compose로 앱과 DB가 같이 뜬다.
- README에 실행 방법이 정리되어 있다.

## 4단계: Kubernetes 기본

기간: 6-8주

공식 학습 자료:

- Kubernetes Basics: https://kubernetes.io/docs/tutorials/kubernetes-basics/
- Kubernetes Documentation: https://kubernetes.io/docs/

처음에는 `kind` 또는 `minikube`로 로컬 클러스터를 사용한다.

배울 것:

- Cluster, Node, Control Plane
- Pod
- ReplicaSet
- Deployment
- Service
- Ingress
- Namespace
- ConfigMap
- Secret
- Job, CronJob
- Volume, PersistentVolume, PersistentVolumeClaim
- ServiceAccount
- RBAC
- Resource request/limit
- Liveness probe, readiness probe
- Taint, toleration
- Affinity

주요 명령어:

```bash
kubectl get pods
kubectl get deploy
kubectl get svc
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- sh
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>
```

실습:

1. Docker 단계에서 만든 앱을 Kubernetes에 배포한다.
2. Deployment를 만든다.
3. Service로 노출한다.
4. Ingress를 붙인다.
5. ConfigMap과 Secret으로 설정을 분리한다.
6. readiness/liveness probe를 추가한다.
7. resource request/limit을 설정한다.
8. 일부러 에러를 만들고 `kubectl describe`, `kubectl logs`로 해결한다.

완료 기준:

- YAML을 보고 어떤 리소스인지 설명할 수 있다.
- Pod가 Pending, CrashLoopBackOff, ImagePullBackOff 상태일 때 원인을 찾을 수 있다.
- Service와 Ingress의 차이를 설명할 수 있다.

## 5단계: Helm, Kustomize, Manifest 관리

기간: 2-3주

배울 것:

- Kubernetes YAML 구조
- Helm chart
- `values.yaml`
- Kustomize base/overlay
- dev/stage/prod 환경 분리

실습:

- 같은 앱을 `dev`, `stage`, `prod` 디렉토리로 나눈다.
- image tag, replica 수, ingress host를 환경별로 다르게 설정한다.
- Helm 또는 Kustomize 중 하나를 먼저 깊게 익힌다.

추천 순서:

1. 순수 YAML
2. Kustomize
3. Helm

## 6단계: Argo CD와 GitOps

기간: 3-4주

공식 학습 자료:

- Argo CD Getting Started: https://argo-cd.readthedocs.io/en/stable/getting_started/

GitOps 핵심 개념:

- Git이 배포 상태의 source of truth다.
- 사람이 클러스터에 직접 수정하지 않고 Git 변경으로 배포한다.
- Argo CD가 Git과 클러스터 상태 차이를 감지하고 동기화한다.

배울 것:

- Argo CD Application
- Sync
- Auto Sync
- Self Heal
- Prune
- App of Apps
- Helm/Kustomize 연동
- Rollback

실습:

1. 로컬 Kubernetes 클러스터에 Argo CD를 설치한다.
2. GitHub repo에 Kubernetes manifest를 올린다.
3. Argo CD Application을 만든다.
4. Git 변경이 클러스터에 반영되는지 확인한다.
5. 클러스터에서 직접 리소스를 바꿔보고 Argo CD가 되돌리는지 확인한다.
6. dev/stage/prod 환경을 분리한다.

완료 기준:

- Git commit으로 배포가 바뀐다.
- Argo CD UI에서 sync 상태를 확인할 수 있다.
- OutOfSync, Healthy, Degraded 상태 의미를 설명할 수 있다.

## 7단계: 플랫폼 엔지니어링 운영 영역

기간: 계속 반복

### 관측성

배울 것:

- Prometheus
- Grafana
- Alertmanager
- Loki 또는 다른 로그 시스템
- metric, log, trace 차이

실습:

- Prometheus/Grafana를 설치한다.
- Pod CPU/Memory 사용량을 대시보드로 본다.
- 간단한 알람을 만든다.

### 보안

배울 것:

- Kubernetes RBAC
- NetworkPolicy
- Secret 관리
- image scan
- least privilege
- admission controller 개념

실습:

- namespace별 권한을 분리한다.
- 특정 Pod 간 통신만 허용하는 NetworkPolicy를 만든다.
- Secret을 plain YAML로 관리할 때의 위험을 정리한다.

### IaC

배울 것:

- Terraform 기본
- provider
- resource
- state
- module
- remote backend

실습:

- Terraform으로 VPC와 EC2를 만든다.
- 나중에 EKS 구성을 Terraform으로 확장한다.

## 8단계: CKA 준비

기간: Kubernetes 기본을 익힌 뒤 6-8주

공식 정보:

- CKA: https://www.cncf.io/training/certification/cka/

CKA는 객관식 시험이 아니라 온라인 실기 시험이다. 제한 시간 안에 터미널에서 Kubernetes 문제를 직접 해결해야 한다.

시험 범위 비중:

- Cluster Architecture, Installation & Configuration: 25%
- Workloads & Scheduling: 15%
- Services & Networking: 20%
- Storage: 10%
- Troubleshooting: 30%

공부 순서:

1. `kubectl` 명령어를 빠르게 입력하는 연습
2. Pod, Deployment, Service, Ingress 문제 풀이
3. ConfigMap, Secret, Volume 문제 풀이
4. RBAC 문제 풀이
5. NetworkPolicy 문제 풀이
6. kubeadm 기반 클러스터 설치/업그레이드
7. etcd backup/restore
8. node 장애, DNS 장애, kubelet 장애 트러블슈팅
9. mock exam 반복

중요한 습관:

- YAML을 처음부터 다 쓰지 말고 `kubectl create ... --dry-run=client -o yaml`로 생성한다.
- 공식 문서를 빠르게 검색하는 연습을 한다.
- alias를 준비한다.
- 문제를 읽고 리소스, namespace, context를 먼저 확인한다.

예시 alias:

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
```

CKA 완료 기준:

- mock exam을 시간 안에 안정적으로 통과한다.
- 장애 상황에서 `describe`, `logs`, `events`, `exec`를 자연스럽게 사용한다.
- etcd backup/restore를 문서 없이 여러 번 성공한다.

## 포트폴리오 프로젝트

### 프로젝트 1: Dockerized Web App

구성:

- 간단한 웹 앱
- DB
- Dockerfile
- Docker Compose

보여줄 역량:

- 컨테이너화
- 환경 변수 관리
- 로컬 개발 환경 구성

### 프로젝트 2: Kubernetes Deployment

구성:

- Deployment
- Service
- Ingress
- ConfigMap
- Secret
- PVC
- resource limit
- health check

보여줄 역량:

- Kubernetes 기본 운영
- manifest 작성
- 장애 진단

### 프로젝트 3: GitOps Platform Lab

구성:

- GitHub repo
- Argo CD
- dev/stage/prod 환경
- Helm 또는 Kustomize
- Prometheus/Grafana

보여줄 역량:

- GitOps 배포
- 환경별 구성 관리
- 관측성
- 플랫폼 운영 흐름 이해

## 주간 학습 루틴

평일:

- 30분: 개념 학습
- 60분: 실습
- 15분: 오늘의 에러와 해결법 기록

주말:

- 2-4시간: 작은 프로젝트 완성
- README 정리
- 배운 내용을 그림으로 정리

기록 방식:

- 오늘 배운 개념
- 실행한 명령어
- 발생한 에러
- 원인
- 해결 방법
- 다음에 확인할 것

## 하지 말아야 할 것

- Kubernetes부터 바로 시작하지 않기
- CNCF Landscape의 모든 도구를 외우려 하지 않기
- 처음부터 EKS, Istio, service mesh를 건드리지 않기
- 강의만 보고 실습을 미루지 않기
- 에러를 지우고 넘어가지 않기
- 비용 알림 없이 AWS 실습하지 않기

## CNCF를 보는 법

CNCF Landscape는 모든 도구를 외우는 지도가 아니라, 분야별 대표 도구를 파악하는 참고 자료다.

공식 CNCF Landscape:

- https://landscapeapp.cncf.io/cncf/

초보자는 아래 정도만 먼저 알면 된다.

- Orchestration: Kubernetes
- Observability: Prometheus, Grafana
- Service Proxy / Ingress: Envoy, NGINX, Traefik
- GitOps / CD: Argo CD, Flux
- Container Runtime: containerd
- Security: Falco, OPA/Gatekeeper
- Networking: CNI, Cilium, Calico
- Storage: CSI, Rook, Longhorn

## 추천 학습 순서 요약

```text
Linux / Network / Git
  -> AWS basics
  -> Docker
  -> Kubernetes
  -> Helm / Kustomize
  -> Argo CD / GitOps
  -> Observability / Security / IaC
  -> CKA
```

## 첫 2주 체크리스트

- [ ] GitHub 저장소 `platform-engineering-lab` 만들기
- [ ] Ubuntu 환경 준비하기
- [ ] Linux 기본 명령어 연습하기
- [ ] SSH 접속 연습하기
- [ ] Nginx 설치하고 접속하기
- [ ] Nginx 로그 보기
- [ ] HTTP, DNS, port 개념 정리하기
- [ ] AWS 계정 생성하기
- [ ] AWS Budget 설정하기
- [ ] IAM 사용자 만들기
- [ ] EC2 한 대 생성하기
- [ ] EC2에 웹 서버 띄우기

## 마음가짐

플랫폼 엔지니어링은 많은 도구를 아는 일이 아니라, 개발자가 안정적으로 배포하고 운영할 수 있는 기반을 만드는 일이다. 처음에는 넓게 보되, 실습은 작게 해야 한다.

첫 번째 목표는 "Kubernetes 전문가"가 아니라 "작은 서비스를 컨테이너로 만들고, Kubernetes에 배포하고, GitOps로 운영해 본 사람"이 되는 것이다.
