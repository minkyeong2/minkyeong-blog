---
title: "알고리즘의 성능을 결정짓는 기준, 복잡도와 Big-O 표기법"
datePublished: Tue Aug 05 2025 08:55:23 GMT+0000 (Coordinated Universal Time)
cuid: cmdyb0daj000x02ji19mi6hzy
slug: big-o
tags: algorithms, big-o, timecomplexity, spacecomplexity

---

알고리즘의 성능을 나타내는 척도로는 일반적으로 \*\*복잡도(Complexity)\*\*를 사용합니다. 복잡도는 크게 **시간 복잡도**와 **공간 복잡도**로 나뉘며, 각각 알고리즘이 실행되는 데 걸리는 시간과 필요한 메모리 공간의 양을 의미합니다.

동일한 기능을 수행하는 알고리즘이라면, 복잡도가 낮은 쪽이 더 효율적인 것으로 간주됩니다.

다만 시간과 공간은 종종 트레이드오프 관계에 있기 때문에, 코딩 테스트와 같은 환경에서는 메모리를 좀 더 사용하는 대신 실행 시간을 줄이는 전략이 많이 사용되곤 합니다.

이 글에서는 알고리즘 성능을 판단하는 두 기준인 시간 복잡도와 공간 복잡도, 그리고 이들을 표현하는 Big-O 표기법에 대해 정리해보겠습니다.

---

### 시간 복잡도 (Time Complexity)

시간 복잡도는 알고리즘이 입력 크기(n)에 따라 **얼마나 많은 연산을 수행하는지**를 수치적으로 표현합니다.

즉, 실행 속도를 예측하고 알고리즘 간의 성능을 비교하는 데 사용됩니다.

보통 Big-O 표기법으로 표현합니다.

ex) `O(1)`, `O(n)`, `O(n²)`, `O(log n)` 등

```java
for (int i = 0; i < n; i++) {
    System.out.println(i);
} // 입력 n에 대해 n번 반복하므로 시간 복잡도는 O(n)
```

---

### 공간 복잡도 (Space Complexity)

공간 복잡도는 알고리즘이 문제를 해결하는 동안 **얼마나 많은 추가 메모리를 사용하는지**를 나타내며,

변수,배열,큐,스택 등의 자료 구조 사용이 이에 해당됩니다.

시간 복잡도와 마찬가지로 Big-O 표기법으로 표현합니다.

```java
int[] arr = new int[n]; // n개의 정수 배열을 사용 -> 공간 복잡도는 O(n)
```

---

### Big-O 표기법이란?

**Big-O 표기법**은 시간 또는 공간 복잡도를 수학적으로 표현하는 방식으로, 보통 알고리즘을 설명할 때 최악의 경우를 기준으로 사용합니다.

입력 크기가 커질수록 실행 시간이나 메모리 사용량이 어떻게 증가하는지를 나타내며, 상수, 계수, 낮은 차수는 생략하고 가장 영향력이 큰 항만을 남겨 간결하게 표현합니다.

예를 들어, `O(3n² + 5n + 10)`은 `O(n²)`로 표기됩니다.

**Big-O는 성능 최적화가 필요한 상황에서 알고리즘의 효율성을 판단하는 중요한 기준이 됩니다.**

아래는 입력 크기 증가에 따른 시간 복잡도의 상대적 차이를 시각화한 그래프입니다.

빨간색 영역일수록 성능이 떨어지며, 초록색 영역일수록 효율적인 알고리즘을 의미합니다.

![Understanding time complexity with Python examples | by Kelvin Salton do  Prado | TDS Archive | Medium](https://miro.medium.com/v2/resize:fit:1400/1*5ZLci3SuR0zM_QlZOADv8Q.jpeg align="left")

---

### 대표적인 Big-O 표기법 정리

| 표기법 | 의미 | 설명 |
| --- | --- | --- |
| `O(1)` | 상수 시간 | 입력 크기에 관계없이 항상 동일한 시간 |
| `O(log n)` | 로그 시간 | 이진 탐색 등, 입력이 두 배가 되어도 연산 횟수는 소폭 증가 |
| `O(n)` | 선형 시간 | 입력 크기에 비례하여 연산이 증가 |
| `O(n log n)` | 로그 선형 시간 | 퀵 정렬, 병합 정렬 등 대부분의 효율적인 정렬 알고리즘 |
| `O(n²)` | 이차 시간 | 중첩 반복문 구조, 버블 정렬 등 |
| `O(2ⁿ)` | 지수 시간 | 모든 조합 탐색 등, 입력이 조금만 커져도 급격히 느려짐 |
| `O(n!)` | 팩토리얼 시간 | 순열 생성 등 모든 경우의 수를 탐색하는 완전탐색 |

참고 자료 : [https://medium.com/data-science/understanding-time-complexity-with-python-examples-2bda6e8158a7](https://medium.com/data-science/understanding-time-complexity-with-python-examples-2bda6e8158a7)

[https://nohack.tistory.com/65](https://nohack.tistory.com/65)