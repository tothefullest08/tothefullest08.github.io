---
layout: post
title: next Permutation
category: Algorithm
tags: [Algorithm]
comments: true
---



# 문제

- 1부터 N까지의 수로 이루어진 순열이 있다. 사전순으로 다음에 오는 순열을 구하는 프로그램을 작성하시오.
- 사전 순으로 가장 앞서는 순열은 오름차순으로 이루어진 순열이고, 가장 마지막에 오는 순열은 내림차순으로 이루어진 순열이다.
- N = 3인 경우에 사전순으로 순열을 나열하면 다음과 같다.
  - 1, 2, 3
  - 1, 3, 2
  - 2, 1, 3
  - 2, 3, 1
  - 3, 1, 2
  - 3, 2, 1
  

# 나의 코드

- 다음 순열의 핵심은 N개의 숫자 중 N-1과 N 을 비교하여 N-1 < N 인 위치를 저장하는 것이다.
- 이후, N-1을 candidate 1 으로 저장한 후, 뒤에서부터 N-1보다 큰 숫자를 찾아 candidate2로 저장.
- cand1 과 cand2의 자리를 바꾼 후, cand1 이후의 숫자를 뒤집어주면 된다. 
- N-1 < N 이 없는 경우, 즉 첫번째 숫자가 가장 큰 경우는 다음 순열이 없는 경우이다.


```python
Data = list(map(int, input().split()))
N = len(Data)

cand1 = 0
for i in range(N-1):
    if Data[i] < Data[i+1]:
        cand1 = i

cand2 = N-1
while True:
    cand2 -=1
    if Data[cand1] > Data[cand2]:
        break

Data[cand1], Data[cand2] = Data[cand2], Data[cand1]
Data[cand1+1:] = Data[:cand1:-1]


if cand1 == 0:
    print("다음 순열이 없습니다")
else:
    print(Data)
```
