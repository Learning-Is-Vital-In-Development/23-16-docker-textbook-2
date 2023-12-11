
## Chapter 13 - 도커 스웜 스택으로 분산 애플리케이션 배포하기

- config 를 통해 적절하게 설정값들을 관리하는 것이 중요
- 비밀값들은 secret 으로 관리
- 설정값들을 업데이트하고 싶다면 서비스를 업데이트하면 됨
- node 에 label 을 부여하여 애플리케이션 실행 노드를 강제할 수 있음. 특정 노드에 저장되어 있는 볼륨 데이터를 유지하는데에 유용
- 네트워크는 애플리케이션과 별도로 관리
- 서비스는 스택이 배포될 때 생성되거나 제거됨
- 서비스의 의존성을 정의하기보다는 모든 서비스를 가능한 빨리 실행시켜서 최종적으로 의존관계를 완성하는 것에 주안점을 둠

### 도커 컴포즈를 사용한 운영 환경

스웜 모드에서는 애플리케이션을 배포할 때 스택을 만든다.

> [!NOTE] 스택이란?
> 서비스, 네트워크, 볼륨 등 여러 개의 도커 리소스를 묶어 만든 리소스

도커 컨테이너는 상한치를 지정하지 않는 한 호스트 컴퓨터의 CPU 와 메모리를 무제한으로 사용할 수 있으므로, 운영 환경에서는 제약을 두어 버그나 악의적인 사용자로부터 보호할 필요가 있다.

### 컨피그 객체를 이용한 설정값 관리

애플리케이션 설정은 모든 컨테이너 오케스트레이션 도구가 전용 1등급 리소스를 따로 마련해 둘만큼 중요한 요소다.

컨피그 객체는 민감한 데이터를 보관하기 위한 수단이 아니다. 컨테이너 간 같은 설정을 공유하게 함으로써 관리를 용이하게 할 수 있다.

### 비밀값을 이용한 대외비 설정 정보 관리하기

비밀값은 컨피그 객체와 비슷한 점이 많지만, 컨피그 객체와 가장 크게 다른 점은 비밀값을 사용하는 워크플로 중 **비밀값이 컨테이너에 전달된 상태에서만 복호화된 비밀값을 볼 수 있다**는 것이다.

비밀값과 컨피그 객체는 매니저 노드에 위치한 분산형 데이터베이스에 저장돼 필요로 하는 어느 노드라도 이를 사용할 수 있다.

컨피그 객체와 비밀값은 수정이 불가하다. 내용을 변경할 필요가 생긴다면 새로운 컨피그 객체나 비밀값을 만들어야 한다. 결국 설정값을 수정하려면 서비스를 업데이트해야 한다. 쿠버네티스에서는 클러스터에 저장된 기존 컨피그 객체나 비밀값을 수정할 수 있다.

### 스웜에서 볼륨 사용하기

컨테이너에서 볼륨의 개념과 오케스트레이션 플랫폼에서의 개념은 같지만, 데이터가 저장되는 방식에 큰 차이가 있다.

클러스터에서는 레플리카가 어떤 노드에서 실행될지 알 수 없다. 따라서 기존과 다른 노드에서 레플리카가 실행되면 데이터가 유실된 것처럼 보일 수 있다. 간단한 해결책은 `label` 을 통해 항상 같은 노드에서 레플리카가 실행될 수 있도록 하는 것이다. 이렇게 배포한 애플리케이션은 스택을 제거하더라도 레이블이 붙은 노드 자체가 남아 있기만 하면 데이터를 유실하지 않는다.

### 클러스터는 스택을 어떻게 관리하는가

- 스웜도 볼륨을 생성하고 삭제할 수 있다
- 비밀값과 컨피그 객체는 설정값이 든 파일을 클러스터에 업로드하는 방법으로 생성한다
- 네트워크는 애플리케이션과 별도로 관리된다
- 서비스는 스택이 배포될 때 생성되거나 제거된다

---

# Chapter 14

## 업그레이드와 롤백을 이용한 업데이트 자동화

빌드에 대한 신뢰감을 쌓는 것이 중요하다. 핵심은 애플리케이션 헬스 체크.

실습은 아래 사이트를 활용하면 편리하다.

https://labs.play-with-docker.com

### 도커를 사용한 애플리케이션 업그레이드 프로세스

정해진 주기 없이 수시로 애플리케이션을 업데이트하는 데 익숙해져야 한다. 이 과정을 자동화하여 빌드에 신뢰감이 생길 수 있도록 하고 바로바로 배포에 포함시킬 수 있도록 해야 한다.

이 과정의 핵심은 애플리케이션 헬스 체크다. 헬스 체크는 애플리케이션이 자기 수복성을 가질 수 있게 하고 안전한 업데이트와 롤백도 가능하게 한다.

이후는 실습.

### 운영 환경을 위한 롤링 업데이트 설정하기

롤링 업데이트 과정을 세세하게 조율하여 업데이트 과정에 신뢰감을 더할 수 있다. 완전히 레플리카가 정상 동작한 이후 다음 레플리카를 교체하는 방식 등이다.

### 서비스 롤백 설정하기

대개 롤백은 자동으로 진행된다. 롤링 업데이트 설정을 잘해두었다면, '업데이트가 끝났는데 신규 기능이 나타나지 않네' 하고 조금 이상한 것을 깨닫고 말 뿐, 더 이상의 일은 일어나지 않는다.

배포 실패가 좋은 일은 아니지만, 자동으로 롤백된다면 애플리케이션이 장애를 일으키지는 않을 것이다. 또한 롤백은 가능한 한 신속하게 이전 상태로 돌아가는 것이 낫다. 서비스 처리 용량이 저하된 상태를 길게 지속하지 않기 위해서이다.

### 클러스터의 중단 시간

오케스트레이션 도구는 여러 대의 컴퓨터를 묶어 하나의 강력한 클러스터로 만든다. 결국 실제로 컨테이너를 실행하는 것은 각각의 컴퓨터이므로 디스크, 네트워크, 전원 등의 이유로 중단 시간이 발생할 수 있다. 대부분의 클러스터는 애플리케이션을 그대로 실행할 수 있지만, 적극적인 조치가 필요한 경우도 있다.

- drain 은 노드를 유지보수 모드로 전환한다.

drain 모드가 되면 해당 노드에서는 레플리카가 종료되고 새로운 레플리카를 실행하지도 않는다. 유지보수 작업이 완료된 이후 다시 클러스터에 합류 시켜서 동작하게 할 수 있다. ([[Kubernetes]] 에도 같은 기능이 있음)

### 스웜 클러스터의 고가용성

지역을 아우르는 거대한 규모의 장애에도 애플리케이션이 계속 동작해야 할 필요가 있다면, 이를 실현할 수 있는 방법은 클러스터를 여러 개 구성하는 것뿐이다. 네트워크 지연으로 정상 노드가 계속 재배치되며 문제를 악화시키는 최악의 시나리오보다는 통제하기 쉽다.
