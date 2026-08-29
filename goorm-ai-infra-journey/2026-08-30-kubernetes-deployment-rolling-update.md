# ☸️ Kubernetes Deployment / Rolling Update / Rollback 실습

## 📌 학습 목표

이번 실습에서는 Kubernetes에서 Pod를 직접 생성하는 방식에서 한 단계 더 나아가 **Deployment를 이용해 애플리케이션을 관리하는 방법**을 학습했다.

주요 학습 내용은 다음과 같다.

- Deployment / ReplicaSet / Pod의 관계
- replicas를 이용한 Pod 개수 관리
- Desired State와 Self-Healing
- Rolling Update를 이용한 버전 교체
- Readiness Probe와 배포의 관계
- 잘못된 이미지 배포 시 `ImagePullBackOff` 장애 확인
- `rollout status`, `rollout history`, `rollout undo`를 이용한 배포 관리
- Rollback 이후 YAML 파일 관리 시 주의점

---

## 1. Deployment / ReplicaSet / Pod 구조

Kubernetes에서 Deployment를 생성하면 내부적으로 ReplicaSet이 생성되고, ReplicaSet이 실제 Pod들을 관리한다.

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
```

각 리소스의 역할은 다음과 같다.

| 리소스 | 역할 |
|---|---|
| Deployment | 애플리케이션의 배포 상태와 버전을 관리 |
| ReplicaSet | 지정된 개수의 Pod를 유지 |
| Pod | 컨테이너가 실행되는 Kubernetes의 기본 실행 단위 |
| Container | 실제 애플리케이션 프로세스 실행 |

---

## 2. Deployment 생성

Nginx를 실행하는 Deployment를 작성했다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: dev
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 2
            periodSeconds: 5
```

Deployment 적용:

```bash
kubectl apply -f 11-web-deploy.yaml
```

상태 확인:

```bash
kubectl get deploy,rs,pods -n dev
```

Deployment 아래에 ReplicaSet이 생성되고 ReplicaSet이 지정된 수의 Pod를 생성하는 것을 확인할 수 있었다.

---

## 3. Desired State와 Self-Healing

Deployment에 다음과 같이 설정했다.

```yaml
replicas: 5
```

이는 단순히 처음에 Pod 5개를 생성하라는 의미를 넘어 **Pod가 5개인 상태를 유지하도록 선언한 것**이다.

실제로 Deployment가 관리하는 Pod 하나를 직접 삭제했다.

```bash
kubectl delete pod <pod-name> -n dev
```

Pod가 삭제되자 새로운 Pod가 자동으로 생성되면서 다시 5개가 유지되었다.

```text
Desired State : Pod 5개
Actual State  : Pod 4개
        ↓
ReplicaSet이 새로운 Pod 생성
        ↓
Actual State  : Pod 5개
```

이를 통해 Kubernetes가 현재 상태를 사용자가 선언한 **Desired State(원하는 상태)**에 지속적으로 맞추는 동작을 확인했다.

---

## 4. Rolling Update 실습

기존 Nginx 이미지 버전을 변경해 Rolling Update를 진행했다.

```text
nginx:1.27
    ↓
nginx:1.25
```

Pod의 변화를 실시간으로 확인했다.

```bash
kubectl get pods -n dev -w
```

다른 터미널에서 변경된 Deployment를 적용했다.

```bash
kubectl apply -f 11-web-deploy.yaml
```

실제 Pod의 변화를 확인해보니 새로운 ReplicaSet이 만들어지고 새로운 Pod가 생성되면서 기존 Pod가 순차적으로 종료되었다.

```text
기존 ReplicaSet
      ↓
기존 Pod 실행 중
      ↓
새 ReplicaSet 생성
      ↓
새로운 Pod 생성
      ↓
새 Pod가 Ready 상태가 됨
      ↓
기존 Pod 일부 종료
      ↓
위 과정 반복
      ↓
새로운 버전의 Pod로 교체 완료
```

Pod 이름의 중간 hash 값이 변경되는 것도 확인했다.

예를 들어:

```text
web-646959b5dd-xxxxx   # 기존 ReplicaSet의 Pod
web-5f6874665d-xxxxx   # 새로운 ReplicaSet의 Pod
```

Rolling Update가 완료된 이후에는 새로운 ReplicaSet이 Pod를 관리하고 기존 ReplicaSet은 replica가 `0`인 상태로 남아 있었다.

```text
NAME             DESIRED   CURRENT   READY
web-5f6874665d   5         5         5
web-646959b5dd   0         0         0
```

이전 ReplicaSet을 바로 삭제하지 않기 때문에 이후 Rollback에 활용할 수 있다.

---

## 5. Readiness Probe와 Rolling Update

이번 Deployment에는 다음과 같은 Readiness Probe가 설정되어 있었다.

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 2
  periodSeconds: 5
```

컨테이너가 실행되었다고 해서 바로 요청을 받을 준비가 끝났다고 볼 수는 없다.

실습에서도 새로운 Pod가 다음과 같이 변하는 모습을 확인했다.

```text
0/1 Pending
    ↓
0/1 ContainerCreating
    ↓
0/1 Running
    ↓
1/1 Running
```

`STATUS`가 Running이어도 `READY`가 `0/1`일 수 있다.

Readiness Probe를 통과하여 `1/1` Ready 상태가 되면 해당 Pod가 요청을 처리할 준비가 되었다고 판단할 수 있다.

따라서 Rolling Update에서는 **새로운 Pod의 준비 상태를 확인하면서 기존 Pod를 순차적으로 교체하는 것**이 서비스 안정성에 중요하다.

---

## 6. 의도적으로 잘못된 이미지 배포

배포 실패 상황을 확인하기 위해 존재하지 않는 이미지 태그를 설정했다.

```yaml
image: nginx:no-such-tag
```

Deployment를 다시 적용했다.

```bash
kubectl apply -f 11-web-deploy.yaml
```

Pod 상태를 실시간으로 확인했다.

```bash
kubectl get pods -n dev -w
```

새로운 Pod가 다음과 같은 상태로 변했다.

```text
Pending
    ↓
ContainerCreating
    ↓
ErrImagePull
    ↓
ImagePullBackOff
```

`nginx:no-such-tag` 이미지를 가져올 수 없기 때문에 컨테이너가 정상적으로 생성되지 못한 것이다.

---

## 7. 배포 실패 시 기존 Pod 확인

잘못된 이미지로 새로운 Pod가 Ready 상태가 되지 못하면서 Rolling Update가 완료되지 않았다.

배포 상태를 확인했다.

```bash
kubectl rollout status deploy/web -n dev
```

결과:

```text
Waiting for deployment "web" rollout to finish:
3 out of 5 new replicas have been updated...
```

새로운 버전이 정상적으로 준비되지 않아 rollout이 계속 대기했다.

ReplicaSet 상태도 확인했다.

```bash
kubectl get rs -n dev
```

실습 당시 상태:

```text
NAME             DESIRED   CURRENT   READY
web-5f6874665d   4         4         4
web-646959b5dd   0         0         0
web-65d5d89588   3         3         0
```

여기서 중요한 점은 새로운 ReplicaSet의 Pod가 실패했다고 해서 기존 정상 ReplicaSet의 Pod를 모두 제거하지 않았다는 것이다.

```text
기존 정상 ReplicaSet
web-5f6874665d
        ↓
정상 Pod 일부 유지

새 ReplicaSet
web-65d5d89588
        ↓
ImagePullBackOff
```

즉 새로운 배포에 문제가 발생하면서 Rolling Update가 중간에 멈췄지만, 기존 정상 Pod들이 남아 있는 것을 확인할 수 있었다.

---

## 8. Rollout History 확인

Deployment의 배포 이력은 다음 명령어로 확인했다.

```bash
kubectl rollout history deploy/web -n dev
```

실습 결과:

```text
REVISION  CHANGE-CAUSE
3         <none>
4         <none>
```

Deployment는 변경된 Pod Template을 기준으로 새로운 ReplicaSet을 생성하고 배포 이력을 관리한다.

ReplicaSet 이름의 hash와 Deployment의 Revision 번호는 같은 값은 아니며 서로 다른 목적으로 사용된다.

---

## 9. Rollback으로 이전 버전 복구

잘못된 이미지 배포를 확인한 뒤 이전 정상 버전으로 Rollback했다.

```bash
kubectl rollout undo deploy/web -n dev
```

결과:

```text
deployment.apps/web rolled back
```

다시 상태를 확인했다.

```bash
kubectl get deploy,rs,pods -n dev
```

결과:

```text
deployment.apps/web   5/5   5   5

NAME             DESIRED   CURRENT   READY
web-5f6874665d   5         5         5
web-646959b5dd   0         0         0
web-65d5d89588   0         0         0
```

실패했던 ReplicaSet은 `0`으로 줄어들고 이전 정상 ReplicaSet이 다시 Pod 5개를 유지하면서 Deployment가 정상 상태로 복구되었다.

```text
정상 버전
    ↓
새 버전 배포
    ↓
ImagePullBackOff 발생
    ↓
Rollout 실패
    ↓
kubectl rollout undo
    ↓
이전 ReplicaSet 활성화
    ↓
Deployment 5/5 정상 복구
```

---

## 10. Rollback 시 주의할 점

실습 중 다음과 같은 Warning도 확인했다.

```text
Rolling back will not update the
kubectl.kubernetes.io/last-applied-configuration annotation
```

`kubectl rollout undo`는 Kubernetes 클러스터의 Deployment 상태를 이전 버전으로 되돌리는 것이지, 로컬에 작성해둔 YAML 파일까지 자동으로 수정해주는 것은 아니다.

예를 들어 YAML에 여전히 다음 내용이 남아 있다면:

```yaml
image: nginx:no-such-tag
```

나중에 다시 `kubectl apply`를 실행했을 때 잘못된 설정이 다시 적용될 수 있다.

따라서 장애 발생 시에는 다음과 같은 흐름으로 대응할 수 있다.

```text
배포 장애 발생
    ↓
상태 확인
    ↓
서비스 영향이 크면 우선 Rollback
    ↓
정상 서비스 복구
    ↓
장애 원인 분석
    ↓
YAML 수정
    ↓
정상 버전으로 다시 배포
```

---

## 🛠️ 주요 명령어 정리

| 명령어 | 용도 |
|---|---|
| `kubectl get deploy,rs,pods -n dev` | Deployment / ReplicaSet / Pod 상태 확인 |
| `kubectl get pods -n dev -w` | Pod 상태 실시간 확인 |
| `kubectl apply -f 11-web-deploy.yaml` | Deployment YAML 적용 |
| `kubectl get rs -n dev` | ReplicaSet 확인 |
| `kubectl rollout status deploy/web -n dev` | 배포 진행 상태 확인 |
| `kubectl rollout history deploy/web -n dev` | Deployment 배포 이력 확인 |
| `kubectl rollout undo deploy/web -n dev` | 이전 버전으로 Rollback |
| `kubectl describe pod <pod-name> -n dev` | Pod 장애 원인 상세 확인 |

---

## 🚨 장애 기록

### 상황 / 증상

Deployment 이미지 태그를 존재하지 않는 `nginx:no-such-tag`로 변경했다.

새로운 Pod가 정상적으로 실행되지 않고 다음 상태가 발생했다.

```text
ErrImagePull
ImagePullBackOff
```

### 확인

```bash
kubectl get pods -n dev
kubectl get rs -n dev
kubectl rollout status deploy/web -n dev
```

새로운 ReplicaSet의 Pod가 `READY 0` 상태였고 Rolling Update가 완료되지 않는 것을 확인했다.

### 원인

존재하지 않는 이미지 태그를 사용하여 Kubernetes가 컨테이너 이미지를 Pull할 수 없었다.

### 해결

```bash
kubectl rollout undo deploy/web -n dev
```

이전 정상 Deployment 버전으로 Rollback했다.

### 해결 확인

```bash
kubectl get deploy,rs,pods -n dev
```

Deployment가 다시 `5/5` Ready 상태가 되고 이전 정상 ReplicaSet의 Pod 5개가 실행되는 것을 확인했다.

---

## 💡 오늘 배운 점

이번 실습을 통해 Deployment는 단순히 여러 개의 Pod를 생성하는 리소스가 아니라는 것을 이해했다.

```text
Desired State 선언
        ↓
ReplicaSet이 Pod 개수 유지
        ↓
Pod 장애 시 자동 복구
        ↓
Rolling Update로 새로운 버전 배포
        ↓
새로운 Pod의 Ready 상태 확인
        ↓
기존 Pod 순차 교체
        ↓
배포 실패 시 기존 정상 Pod 유지
        ↓
필요하면 Rollback
```

특히 정상적인 배포만 진행한 것이 아니라 **일부러 잘못된 이미지 태그를 적용해 `ImagePullBackOff` 장애를 발생시키고, 상태 확인 → ReplicaSet 확인 → Rollback → 정상화까지 직접 진행했다.**

Docker Compose에서 컨테이너를 직접 내렸다가 다시 올리는 방식과 비교하면서 Kubernetes가 왜 운영 환경에서 Deployment와 같은 Controller를 이용해 애플리케이션의 상태를 관리하는지도 조금 더 이해할 수 있었다.

---

## 📌 핵심 정리

> **Deployment는 원하는 Pod 개수를 지속적으로 유지하고, Rolling Update를 통해 새로운 버전을 순차적으로 배포하며, 배포에 문제가 발생하면 이전 정상 버전으로 Rollback할 수 있도록 애플리케이션의 상태와 배포를 관리한다.**

다음 학습에서는 Deployment로 관리되는 여러 Pod 앞에 **Service**를 연결하여, Pod가 교체되고 IP가 변경되어도 하나의 안정적인 접근 지점을 통해 애플리케이션에 접근하는 구조를 학습할 예정이다.