---
layout: post
title: 버블 정렬, 카운팅 정렬
category: Algorithm
tags: [Algorithm, 정렬]
comments: true
---





# 버블 정렬 (Bubble Sort)

> - 인접한 두개의 원소를 비교하여 자리를 계속 교환하는 방식
> - 수열을 정렬하는 알고리즘 중 하나
> - 시간 복잡도: O(n^2)



### 정렬 과정

- 저울이 있다고 가정, 배열의 길이는 N
- 저울의 왼쪽/오른쪽에 배열의 첫번째 숫자와 인접한 숫자(두번째 숫자)를 올려 크기를 비교함
  - 두번째 숫자가 작을 경우, 서로 위치를 바꿈
- 저울의 왼쪽/오른쪽에 배열의 두번째 숫자와 세번째 숫자를 올려 크기를 비교함
- 동일한 방법으로 배열의 끝(N)까지 숫자를 비교할 경우, 배열의 처음에는 가장 작은 숫자가 위치하게 됨.
- 2번째 Pass에는 배열의 두번째 숫자부터 다시 비교
- 정렬이 완료 될 때 까지 위의 과정을 반복함

  



### 나의 코드

- 주석처리한 `print` 함수를 이용하면 정렬되는 과정을 알 수 있음.

```python
Data = [55, 7, 78, 12, 42]
N = len(Data)

#앞에서 부터 정렬
for i in range(N-1):
    for j in range(i, N-1):
        #print(Data)
        if Data[j] > Data[j+1]:
            Data[j], Data[j+1] = Data[j+1], Data[j]
        #print(Data)
    #print(Data)

    
#뒤에서 부터 정렬
for i in range(N-1, 0, -1):
    for j in range(0, i):
        #print(Data)
        if Data[j] > Data[j+1]:
            Data[j], Data[j+1] = Data[j+1], Data[j]
        #print(Data)
    #print(Data)
```



<br>

# 카운팅 정렬(counting Sort)

> - 항목들의 순서를 결정하기 위해 집합에 각 항목이 몇개씩 있는지 세는 작업을 하여, 선형시간에 정렬하는 효율적인 알고리즘으로, 속도가 빠르며 안정적임.
> - 제한 사항
>   - 정수나 정수로 표현 할 수 있는 자료에만 적용 가능 (각 항목의 발생 횟수를 기록하기 위해 정수 항목으로 인덱스 되는 카운트들의 배열을 사용하기 때문)
>   - 카운트들을 위한 충분한 공간을 할당하려면 집합 내 가장 큰 정수를 알아야함
> - 시간 복잡도: O(n+k)  - n은 리스트의 길이, k는 정수의 최대 값



- 배열내 숫자를 카운팅 배열 내 인덱스로 접근하여 발생 횟수를 증가시키는 개념이 사실 직관적으로 이해하는데 많은 시간이 걸렸다.
- YABOONG님의 블로그에 친절한 설명이 적힌 슬라이드를 발견하였다. 참조해보자! [동작방식 링크](https://www.slideshare.net/devDaniel/countingsort-91287575) 



### 나의 코드

```python
Data = [1,0,3,1,3,1]
N = len(Data)

#인덱스 값 + 발생횟수 저장할 리스트 생성.
#인덱스는 0부터 시작하므로 정수의 최대값 +1 값 적용
counting_lst = [0] * (max(Data)+1)
#최종 정렬 리스트
sorted_lst = [0] * N

#각 숫자의 갯수를 계산하여, count 리스트의 인덱스에 발생횟수를 증가시킨다.
for i in range(N):
    counting_lst[Data[i]] += 1

#누적합 계산
for i in range(len(counting_lst)-1):
    counting_lst[i+1] += counting_lst[i]


#기본 리스트의 끝에서부터 역으로 계산하기 위해 range(N-1, -1, -1) 적용
for i in range(N-1, -1, -1):
    #누적합을 -1 해주면, 그 값이 바로 최종 리스트의 위치가 됨.
    counting_lst[Data[i]] -= 1
    sorted_lst[counting_lst[Data[i]]] = Data[i]
    print(sorted_lst)
```

