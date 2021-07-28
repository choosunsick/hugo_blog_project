---
title: "몬테카를로 시뮬레이션 - 시뮬레이션 회수에 따른 실험"
date: 2021-07-28T21:16:26+09:00
draft: False
tags: ["이산수학","Python"]
categories: ["이산수학","Python"]
---

[이전 글](https://choosunsick.github.io/post/montecalo_pyvsrs/) 에서 시뮬레이션 회수에 대비해 실제 파이 값과 어느 정도로 차이가 나는지에 대한 실험을 해보겠습니다.

이번에는 시뮬레이션 회수를 변경해 가면서 실험합니다. 점 개수를 10000 개로 고정하고 시뮬레이션 회수를 100에서 10000까지 100씩 증가시키면서 파이 값을 추정 합니다. 이후 추정된 값과 실제 파이 값과 차이의 절대값을 구하고 그래프를 그려 방향성을 확인합니다.

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

# 점 개수는 10000으로 고정하면서 시뮬레이션 회수를 100~10000까지 실행 
pi_set2 = [simulation(10000,i) for i in range(100,10100,100)]
diff2 = [abs(3.14159265359 -i) for i in pi_set2]

plt.plot(range(100,10100,100),diff2)
plt.ylabel('diff')
plt.xlabel('number of simulation')
plt.show()
```

![시뮬레이션 회수 변화에 따른 파이 값과 차이](https://user-images.githubusercontent.com/19144813/127321267-05998d63-b8c4-4604-817c-194d94732e62.png)

## 실험 결과

이제 두 번째 시뮬레이션 회수 변화에 따른 그래프인 두 번째 그래프를 살펴봅시다. 그래프의 방향은 우하향을 이루고 있습니다. 그리고 변동폭 역시 시뮬레이션 회수가 늘어날 수록 점차 줄어 드는 양상을 보입니다. 실제 파이 값과의 차이 역시 기존에 0.002 이상 차이나는 것에서 회수가 증가 함에 따라 0.0005와 0 사이의 값으로 줄어들어감을 확인할 수 있습니다.

시뮬레이션 회수가 증가하면 실제 값과의 차이가 점차 줄어드는 양상 또한 확인할 수 있었습니다. 이는 두 번째 그래프의 방향성이 우하향을 그리고 있음을 통해 알 수 있습니다. 즉 단순히 파이 값을 추정하는데서 그치는 것이 아닌 시뮬레이션의 회수를 늘려도 추정된 값과 실제 파이 값과의 차이를 줄일 수 있습니다. 따라서 시뮬레이션 회수는 실제 파이 값 추정할 때 값의 정확성에 영향을 미친다는 점을 알 수 있었습니다.