---
title: "순열과 조합"
date: 2021-07-15T15:30:59+09:00
draft: False
tags: ["이산수학","Python"]
categories: ["이산수학","Python"]
---

3월부터 공부한 이산수학 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 조합과 순열의 특징

순열과 조합은 모두 숫자를 나열하는 방법의 수를 나타냅니다. 두 가지의 차이점은 순열은 순서가 중요하다는 것입니다. 예를 들면 조합에서는 1,2,3 과 2,1,3, 3,2,1 등과 같은 수열을 같은 것으로 봅니다. 순서가 없기 때문입니다. 반면 순열에서는 1,2,3 과 2,1,3은 다른 수열로 봅니다. 순서가 다르기에 수열도 다르다고 보는 것입니다. 따라서 일반적으로 순열의 개수가 조합의 개수보다 많습니다.

일반적인 순열과 조합에서는 중복을 허용하지 않습니다. 예를 들면 1,1,1 같은 수열은 수를 나열하는 방법으로 올 수 없다는 말입니다. 그러나 중복을 허용하는 중복 순열과 중복 조합이 있습니다. 중복 순열과 중복 조합은 중복을 허용하기에 1,1,1과 같은 수열이 수를 나열하는 방법으로 올 수 있습니다. 중복 순열과 중복 조합의 차이도 마찬가지로 순서입니다. 예를 들면 1,1,2 와 1,2,1을 같은 수열로 보지 않는 경우가 중복 순열이며 같은 수열로 보는 경우가 중복 조합에 해당됩니다.

## 코드 실행 결과

```bash
# 순열
len(list(permutations(tmp,3)))
60

# 조합
len(list(combinations(tmp,3)))
10

# 중복 순열
len(list(product(tmp,repeat=3)))
125

# 중복 조합
len(list(combinations_with_replacement(tmp,3)))
35

(2, 3, 5)
(2, 3, 5)
(2, 3, 5)
(2, 3, 5)
```

순열의 개수가 60개 조합의 개수가 10개로 더 많은 것을 확인할 수 있습니다. 중복 허용한 결과에서도 마찬가지로 중복 순열의 개수가 125개 중복 조합의 개수가 35개로 더 많은 것을 확인할 수 있습니다. 정리하면 가장 많은 나열 방법은 125가지의 나열 방법의 수를 가지는 중복을 허용하고 순서를 중요시하는 중복 순열 입니다. 가장 적은 나열 방법은 10가지 나열 방법의 수를 가지는 중복을 허용하지 않고 순서를 무시하는 방법인 조합 입니다.

각 조합과 순열에서 다음 수열의 값은 [2,3,5] 로 모두 같은 수열이 나오게 됩니다.

## 문제 풀이

1,2,3,4,5 중 3개를 뽑는 방법의 수 - 순열, 조합, 중복 여부에 따라 어떻게 변하는지 모두 확인하시오.
또한, 각 순열, 조합의 경우에서 2,3,4 다음에 오는 수열을 구하시오.

```python
from itertools import permutations
from itertools import combinations
from itertools import product
from itertools import combinations_with_replacement

tmp = [1,2,3,4,5]
len(list(permutations(tmp,3)))
len(list(combinations(tmp,3)))
len(list(product(tmp,repeat=3)))
len(list(combinations_with_replacement(tmp,3)))

for i in range(0,len(list(permutations([1,2,3,4,5],3)))):
    if list(permutations([1,2,3,4,5],3))[i] == (2,3,4):
        print(list(permutations([1,2,3,4,5],3))[i+1])

for i in range(0,len(list(combinations([1,2,3,4,5],3)))):
    if list(combinations([1,2,3,4,5],3))[i] == (2,3,4):
        print(list(combinations([1,2,3,4,5],3))[i+1])

for i in range(0,len(list(product([1,2,3,4,5],repeat=3)))):
    if list(product([1,2,3,4,5],repeat=3))[i] == (2,3,4):
        print(list(product([1,2,3,4,5],repeat=3))[i+1])
    
for i in range(0,len(list(combinations_with_replacement([1,2,3,4,5],3)))):
    if list(combinations_with_replacement([1,2,3,4,5],3))[i] == (2,3,4):
        print(list(combinations_with_replacement([1,2,3,4,5],3))[i+1])
```