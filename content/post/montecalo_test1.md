---
title: "몬테카를로 시뮬레이션 점 개수에 대한 실험"
date: 2021-07-27T19:23:09+09:00
draft: False
tags: ["이산수학","Python"]
categories: ["이산수학","Python"]
---

이번에는 점 개수가 늘어날 때 실제 파이 값과 실제로 가까워 지는지 확인 해보겠습니다. 실험의 계획은 다음과 같습니다. 먼저 점 개수를 1000~100000개 까지 1000개씩 늘려가며 시뮬레이션 회수는 1000회로 고정하고 파이 값을 추정합니다. 그리고 추정된 100개의 파이 값과 실제 파이 값과 차이의 절대값을 구합니다. 이후 차이 값들의 그래프를 통해 방향성을 확인합니다.

```python
from random import uniform
import math
import matplotlib.pyplot as plt

def pi(n):
    cnt = 0
    for _ in range(n):
        x = uniform(-1,1)
        y = uniform(-1,1)
        if math.sqrt(x**2+y**2) < 1:
            cnt += 1
    return (4 * cnt)/ n

def simulation(n,simul_num):
    temp = [n]*simul_num
    pi_vals = [pi(i) for i in temp]
    return np.mean(pi_vals)

# 점 개수의 변화를 주면서 1000회 시뮬
pi_set = [simulation(i,1000) for i in range(1000,101000,1000)]

diff = [abs(3.14159265359 -i) for i in pi_set]

plt.plot(range(1000,101000,1000),diff)
plt.ylabel('diff')
plt.xlabel('point_num')
plt.show()
```
[점 개수 변화에 따른 파이 값과 차이](https://user-images.githubusercontent.com/19144813/127138086-5b9f291e-70e4-4ca1-8447-98ae007d8361.png)


## 실험 결과

먼저 점 개수 변화에 따른 그래프인 첫 번째 그래프를 살펴보면 변동 폭이 좀 있긴 하지만 곡선이 우하향하고 있다는 것을 확인할 수 있습니다. 100에 가까워 질 수록 차이가 주는 것도 확인할 수 있습니다. 처음의 차이 0.0014에서 0으로 떨어져 가는 것을 확인할 수 있습니다. 점 개수가 증가할 수록 실제 값에 가까워 진다는 것을 확인할 수 있었습니다. 그러나 의외로 점 개수가 일정하게 증가한다해서 추정 값과 실제 값 간의 차이도 일정하지는 않다는 것입니다. 이는 첫 번째 그래프에서 변동폭이 크게 나타나는 것을 통해 알 수 있습니다.
