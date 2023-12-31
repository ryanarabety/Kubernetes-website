---
title: 쿠버네티스 오브젝트 관리
content_type: concept
weight: 20
---

<!-- overview -->
`kubectl` 커맨드라인 툴은 쿠버네티스 오브젝트를 생성하고 관리하기 위한
몇 가지 상이한 방법을 지원한다. 이 문서는 여러가지 접근법에 대한 개요를
제공한다. Kubectl로 오브젝트 관리하기에 대한 자세한 설명은
[Kubectl 서적](https://kubectl.docs.kubernetes.io)에서 확인한다.

<!-- body -->

## 관리 기법

{{< warning >}}
쿠버네티스 오브젝트는 하나의 기법만 사용하여 관리해야 한다. 동일한 오브젝트에
대해 여러 기법을 혼용하는 것은 예상치 못한 동작을 초래하게 된다.
{{< /warning >}}

| 관리기법             | 대상             | 권장 환경    | 지원하는 작업자 수 | 학습 난이도 |
|--------------------|-----------------|------------|-----------------|----------|
| 명령형 커맨드         | 활성 오브젝트      | 개발 환경    | 1+             | 낮음       |
| 명령형 오브젝트 구성   | 개별 파일         | 프로덕션 환경 | 1               | 보통      |
| 선언형 오브젝트 구성   | 파일이 있는 디렉터리 | 프로덕션 환경 | 1+             | 높음      |

## 명령형 커맨드

명령형 커맨드를 사용할 경우, 사용자는 클러스터 내 활성 오브젝트를 대상으로
직접 동작시킨다. 사용자는 실행할 작업을 인수 또는 플래그로 `kubectl` 커맨드에
지정한다.

이것은 클러스터에서 일회성 작업을 개시시키거나 동작시키기 위한
추천 방법이다. 이 기법은 활성 오브젝트를 대상으로 직접적인
영향을 미치기 때문에, 이전 구성에 대한 이력을 제공해 주지 않는다.

### 예시

디플로이먼트 오브젝트를 생성하여 nginx 컨테이너의 인스턴스를 구동시킨다.

```sh
kubectl create deployment nginx --image nginx
```

### 트레이드 오프

오브젝트 구성에 비해 장점은 다음과 같다.

- 커맨드는 하나의 동작을 나타내는 단어로 표현된다.
- 커맨드는 클러스터를 수정하기 위해 단 하나의 단계만을 필요로 한다.

오브젝트 구성에 비해 단점은 다음과 같다.

- 커맨드는 변경 검토 프로세스와 통합되지 않는다.
- 커맨드는 변경에 관한 감사 추적(audit trail)을 제공하지 않는다.
- 커맨드는 활성 동작 중인 경우를 제외하고는 레코드의 소스를 제공하지 않는다.
- 커맨드는 새로운 오브젝트 생성을 위한 템플릿을 제공하지 않는다.

## 명령형 오브젝트 구성

명령형 오브젝트 구성에서는 kubectl 커맨드로 작업(생성, 교체 등),
선택적 플래그, 그리고 최소 하나의 파일 이름을 지정한다.
그 파일은 YAML 또는 JSON 형식으로 오브젝트의 완전한 정의를
포함해야만 한다.

오브젝트 정의에 대한 더 자세한 내용은 [API 참조](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)를
참고한다.

{{< warning >}}
명령형 `replace` 커맨드는 기존 spec을 새로 제공된 spec으로 바꾸고
구성 파일에서 누락된 오브젝트의 모든 변경 사항을 삭제한다.
이 방법은 spec이 구성 파일과는 별개로 업데이트되는 리소스 유형에는
사용하지 말아야한다.
예를 들어 `LoadBalancer` 유형의 서비스는 클러스터의 구성과 별도로
`externalIPs` 필드가 업데이트된다.
{{< /warning >}}

### 예시

구성 파일에 정의된 오브젝트를 생성한다.

```sh
kubectl create -f nginx.yaml
```

두 개의 구성 파일에 정의된 오브젝트를 삭제한다.

```sh
kubectl delete -f nginx.yaml -f redis.yaml
```

활성 동작하는 구성을 덮어씀으로써 구성 파일에 정의된 오브젝트를
업데이트한다.

```sh
kubectl replace -f nginx.yaml
```

### 트레이드 오프

명령형 커맨드에 비해 장점은 다음과 같다.

- 오브젝트 구성은 Git과 같은 소스 컨트롤 시스템에 보관할 수 있다.
- 오브젝트 구성은 푸시와 감사 추적 전에 변경사항을 검토하는 것과 같은 프로세스들과 통합할 수 있다.
- 오브젝트 구성은 새로운 오브젝트 생성을 위한 템플릿을 제공한다.

명령형 커맨드에 비해 단점은 다음과 같다.

- 오브젝트 구성은 오브젝트 스키마에 대한 기본적인 이해를 필요로 한다.
- 오브젝트 구성은 YAML 파일을 기록하는 추가적인 과정을 필요로 한다.

선언형 오브젝트 구성에 비해 장점은 다음과 같다.

- 명령형 오브젝트 구성의 동작은 보다 간결하고 이해하기 쉽다.
- 쿠버네티스 버전 1.5 부터는 더 성숙한 명령형 오브젝트 구성을 제공한다.

선언형 오브젝트 구성에 비해 단점은 다음과 같다.

- 명령형 오브젝트 구성은 디렉터리가 아닌, 파일에 가장 적합하다.
- 활성 오브젝트에 대한 업데이트는 구성 파일에 반영되어야 한다. 그렇지 않으면 다음 교체 중에 손실된다.

## 선언형 오브젝트 구성

선언형 오브젝트 구성을 사용할 경우, 사용자는 로컬에 보관된 오브젝트
구성 파일을 대상으로 작동시키지만, 사용자는 파일에서 수행 할
작업을 정의하지 않는다. 생성, 업데이트, 그리고 삭제 작업은
`kubectl`에 의해 오브젝트마다 자동으로 감지된다. 이를 통해 다른 오브젝트에 대해
다른 조작이 필요할 수 있는 디렉터리에서 작업할 수 있다.

{{< note >}}
선언형 오브젝트 구성은 변경 사항이 오브젝트 구성 파일에
다시 병합되지 않더라도 다른 작성자가 작성한 변경 사항을 유지한다.
이것은 전체 오브젝트 구성 변경을 위한 `replace` API를
사용하는 대신, `patch` API를 사용하여 인지되는 차이만
작성하기 때문에 가능하다.
{{< /note >}}

### 예시

`configs` 디렉터리 내 모든 오브젝트 구성 파일을 처리하고 활성 오브젝트를
생성 또는 패치한다. 먼저 어떠한 변경이 이루어지게 될지 알아보기 위해 `diff`
하고 나서 적용할 수 있다.

```sh
kubectl diff -f configs/
kubectl apply -f configs/
```

재귀적으로 디렉터리를 처리한다.

```sh
kubectl diff -R -f configs/
kubectl apply -R -f configs/
```

### 트레이드 오프

명령형 오브젝트 구성에 비해 장점은 다음과 같다.

- 활성 오브젝트에 직접 작성된 변경 사항은 구성 파일로 다시 병합되지 않더라도 유지된다.
- 선언형 오브젝트 구성은 디렉터리에서의 작업 및 오브젝트 별 작업 유형(생성, 패치, 삭제)의 자동 감지에 더 나은 지원을 제공한다.

명령형 오브젝트 구성에 비해 단점은 다음과 같다.

- 선언형 오브젝트 구성은 예상치 못한 결과를 디버깅하고 이해하기가 더 어렵다.
- diff를 사용한 부분 업데이트는 복잡한 병합 및 패치 작업을 일으킨다.

## {{% heading "whatsnext" %}}

- [명령형 커맨드를 이용한 쿠버네티스 오브젝트 관리하기](/ko/docs/tasks/manage-kubernetes-objects/imperative-command/)
- [구성파일을 이용한 명령형 쿠버네티스 오브젝트 관리](/ko/docs/tasks/manage-kubernetes-objects/imperative-config/)
- [구성 파일을 이용한 쿠버네티스 오브젝트의 선언형 관리](/ko/docs/tasks/manage-kubernetes-objects/declarative-config/)
- [Kustomize를 이용한 쿠버네티스 오브젝트의 선언형 관리](/ko/docs/tasks/manage-kubernetes-objects/kustomization/)
- [Kubectl 커맨드 참조](/docs/reference/generated/kubectl/kubectl-commands/)
- [Kubectl 서적](https://kubectl.docs.kubernetes.io)
- [쿠버네티스 API 참조](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)
