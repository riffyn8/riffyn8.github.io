---
layout: post
title: 가비지 컬렉션(Garbage Collection)
date: 2026-03-12
categories: [Java]
math: true
---

## 가비지 컬렉션(GC)의 필요성
Stack 영역에 저장되는 메서드 정보와 지역 변수들은 생명주기가 명확하다. 
메서드가 호출될 때 생성되고, 종료되면 마지막에 실행된 메서드부터 차례대로 메모리에서 소멸한다. 
반면 Heap 영역에 생성된 객체들은 여러 메서드나 클래스에서 참조될 수 있어 언제까지 유지되어야 하는지 예측하기 어렵다.
사용이 끝난 객체들을 그대로 방치하면 메모리 누수(Memory Leak)가 발생하므로, 이를 해결하기 위해 가비지 컬렉션(GC)이 필요하다. <br/> 

GC는 객체의 유효성을 판단하기 위해 루트 객체(Root Set)에서 시작하여 참조 관계를 추적한다.
- **루트 객체**: 현재 실행 중인 스레드 스택의 지역 변수, 메서드 영역의 static 변수, 네이티브 메서드 스택에서 참조 중인 객체 등이 포함된다.
- **Garbage 판정**: 루트 객체로부터 참조 사슬이 연결된 객체는 '도달 가능한 객체(Reachable)'로 유지하고, 어떤 경로로도 도달 할 수 없는 객체는 '가비지(Unreachable)'로 간주하여 메모리에서 제거한다.

정리하면, 실행 흐름에 필요한 참조값은 Stack에, 실제 데이터는 Heap에 저장된다. 지역 변수는 메서드 종료와 함께 즉시 소멸하지만, 인스턴스 변수는 더 이상 참조되는 곳이 없어질 때 GC가 감지하여 메모리를 정리한다.

> 가비지 컬렉터(Garbage Collector)는 JVM 실행 엔진의 구성 요소로, Heap 메모리에서 더 이상 참조되지 않는 객체를 찾아 삭제하는 실질적인 추제이다.
> 가비지 컬렉션(GC)은 이 컬렉터가 불필요한 메모리를 자동으로 해제하는 일련의 과정 또는 메커니즘을 의미한다.

---

## 힙(Heap) 메모리 구조

![heap 메모리 구조](https://lh3.googleusercontent.com/vTAw_POS001w3iGsxIbq74nMLDKxh1_7Ng0_C6YjkjgNMmD_3KaKwj2hkKFGuE4ieqD1tPQwNVjhmPR1e-WJkY7t0C035VbLWX2FEHcg4RlM0MQ0xEkuITG3CZVYNs8bYF0qhm3v){: .normal }

힙은 **Young Generation**과 **Old Generation**으로 구분된다. <br/>

### Young Generation
Eden과 Survivor 0/1 영역으로 구성된다.
- **Eden**: `new` 키워드로 객체를 생성할 때 최초로 할당되는 공간이다. 성경의 에덴동산에서 유래한 이름으로, 최초의 인류인 아담과 이브가 탄생한 것처럼 객체가 처음 생성되는 곳임을 비유한 것이다.
- **Survivor 0/1**: Eden에서 GC 이후 살아남은 객체들이 머무는 공간이다. 항상 둘 중 하나는 반드시 비어 있어야 한다. GC가 발생할 때마다 살아남은 객체는 비어있는 쪽 Survivor로 이동하며, 이 과정을 반복하면서 age가 증가한다.

### Old Generation
Young Generation에서 오랫동안 살아남아 age 임계치를 초과한 객체들이 이동해 오는 공간이다. 일반적으로 Young Generation 보다 크게 할당된다.

---

## GC의 동작
### 동작 구분
- **Stop-The-World(STW)**: GC가 실행될 때 JVM이 애플리케이션 실행을 멈추는 현상이다. 모든 스레드가 중단되며 GC가 완료될 때까지 재개되지 않는다.
- **Mark and Sweep**: 살아있는 객체를 식별(Mark)하고 불필요한 객체를 제거(Sweep)하는 GC의 기본 동작 방식이다.

### 동작 과정
1. **객체 생성**:`new` 키워드로 객체를 생성하면 Eden 영역에 위치한다.
2. **Minor GC 발생**: Eden이 가득차면 Minor GC가 발생한다.
3. **Mark**: 살아있는 객체를 식별한다.
4. **Sweep**: 죽은 객체(Unreachable)를 메모리에서 제거한다.
5. **Compaction**: 남은 객체를 압축하여 메모리 단편화를 방지한다. 
6. **Promotion**: 살아남은 객체는 Survivor를 오가며 age가 증가하고, 임계치를 초과하면 Old Generation으로 이동한다.
7. **Major GC 발생**: Old Generation이 가득차면 Major GC가 발생한다. Minor GC보다 훨씬 느리며 STW 시간도 길다.

> Minor GC와 Major GC가 발생할 때마다 Stop-The-World가 함께 발생하며, 이를 최소화하기 위해 다양한 GC 알고리즘이 사용된다.

---

## GC 알고리즘
- **Serial GC**: 단일 스레드로 GC 수행. 소규모 애플리케이션에 적합
- **Parallel GC**: 멀티 스레드로 GC 수행. 처리량(Throughput) 위주 시스템에 적합 (Java 8 기본값)
- **CMS GC**: STW 최소화. 그러나 Compaction를 수행하지 않으므로 메모리 단편화가 발생할 수 있다. (Java 14에서 제거되어 더 이상 사용하지 않는다)
- **G1 GC**: 힙을 Region으로 나눠 관리. 예측 가능한 짧은 STW 시간을 제공한다. (Java 9+ 기본값)
- **ZGC**: 초저지연 GC. 대용량 메모리에서도 수 밀리초 단위의 STW를 제공한다. (Java 15+에서 정식 도입)
- **Shenandoah**: ZGC와 유사한 초저지연 GC. (Java 12+ (OpenJDK))

---

참고하기 좋은 자료: [Java Garbage Collection](https://d2.naver.com/helloworld/1329)
