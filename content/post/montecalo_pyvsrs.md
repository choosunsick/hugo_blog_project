---
title: "몬테카를로 시뮬레이션을 통한 파이값 추정 작동시간 비교하기"
date: 2021-07-22T18:41:12+09:00
draft: False
tags: ["이산수학","Rust","Python"]
categories: ["이산수학","Rust","Python"]
---

파이값을 추정하는 몬테카를로 시뮬레이션을 파이썬과 Rust로 구현하고 작동 시간을 비교합니다. 몬테카를로 시뮬레이션에 대한 설명은 [링크](https://choosunsick.github.io/post/montecalo/)를 통해 확인하실 수 있습니다.

## 파이썬 구현

참고로 random 과 math 패키지는 기본 패키지이기 때문에 별도의 설치가 필요하지 않습니다. 단 numpy의 경우 `pip3 install numpy`로 패키지의 설치가 우선되어야 합니다. 시뮬레이션 횟수는 점의 수는 100000개를 찍으며 시뮬레이션 횟수는 100회로 가정합니다. 100회 시뮬레이션 이후 평균 값을 통해 파이값을 추정해볼 수 있습니다. 시간 측정에는 “pytest-benchmark” 패키지를 이용합니다. 패키지 설치 방법은 다음과 같습니다. `pip3 install pytest-benchmark` 패키지를 설치한 다음 아래 코드를 simulation.py 파일로 저장합니다. 이후 테스트 실행 방법은 `pytest --benchmark-only simulation.py`를 터미널에서 입력해주시면 됩니다.

아래 코드는 simulation.py 의 내용입니다.

```python
from random import uniform
import math
import numpy as np

def pi(n):
    cnt = 0
    for _ in range(n):
        x = uniform(-1,1)
        y = uniform(-1,1)
        if math.sqrt(x**2+y**2) < 1:
            cnt += 1
    return (4 * cnt)/ n

def simulation():
    temp = [100000]*100
    pi_vals = [pi(i) for i in temp]
    return np.mean(pi_vals)

def test(benchmark):
    benchmark(simulation)
```

추정된 파이값 확인하기

```python
from random import uniform
import math
import numpy as np

def pi(n):
    cnt = 0
    for _ in range(n):
        x = uniform(-1,1)
        y = uniform(-1,1)
        if math.sqrt(x**2+y**2) < 1:
            cnt += 1
    return (4 * cnt)/ n

def simulation(n):
    temp = [100000]*n
    pi_vals = [pi(i) for i in temp]
    return np.mean(pi_vals)

print(simulation(100))
print(simulation(1000))
print(simulation(10000))
```

## rust 구현

아래 코드에서 사용하는 rand 크레이트를 cargo.toml에 추가해주어야 코드가 작동합니다. 추가 방법은 [dependencies] 칸 아래 `rand = "0.8"`를 적어주는 것입니다. pi 함수의 `rng.gen_range(-1.0..1.0)` 부분은 두 수의 범위 내의 랜덤한 숫자를 생성하는 함수입니다. 이 함수는 rand crate에 속해 있는 함수 입니다. 이에 따라 rand 크레이트를 사용하기 위한 과정이 필요합니다. `powi()` 함수는 러스트의 기본 함수로 어떤 값을 정수 제곱 해주는 함수입니다. 러스트의 경우 정수 제곱 배가 아닌 경우를 위한 함수도 존재합니다. 그 함수는 `powf()`입니다. 참고로 `sqrt()` 함수는 루트를 의미하는 기본 함수입니다. 또한 pi 함수에서 점 개수가 10억개가 넘어갈 경우 오버플로우가 발생할 수 있기 때문에 cnt 값의 타입을 usize로 설정해 줍니다.

```rust
use rand::Rng;
use std::time::Instant;

fn main() {
    let hundred_thousand = 100000;
    let million = 1000000;
    let now = Instant::now();
    println!(
        "100회 시뮬레이션 추정값: {:?}",
        simulation(100, hundred_thousand)
    );
    let after = Instant::now();
    println!(
        "100회 시뮬레이션 시간: {:?}",
        after.checked_duration_since(now)
    );
    println!(
        "1000회 시뮬레이션 추정값: {:?}",
        simulation(1000, hundred_thousand)
    );
    println!(
        "10000회 시뮬레이션 추정값: {:?}",
        simulation(10000, hundred_thousand)
    );
    println!(
        "점 100만개 100회 시뮬레이션 추정값: {:?}",
        simulation(100, million)
    );
    println!(
        "점 100만개 1000회 시뮬레이션 추정값: {:?}",
        simulation(1000, million)
    );
    println!(
        "점 100만개 10000회 시뮬레이션 추정값: {:?}",
        simulation(10000, million)
    );
}

fn pi(n: i32) -> f32 {
    let mut cnt: usize = 0;
    let mut rng = rand::thread_rng();
    for _ in 0..n {
        let x = rng.gen_range(-1.0..1.0);
        let y = rng.gen_range(-1.0..1.0);
        let nums = f32::powi(x, 2) + f32::powi(y, 2);
        if nums.sqrt() < 1.0 {
            cnt += 1;
        }
    }
    return (4 * cnt) as f32 / n as f32;
}

fn mean(data: &Vec<f32>) -> Option<f32> {
    let sum = data.iter().sum::<f32>() as f32;
    let count = data.len();

    match count {
        positive if positive > 0 => Some(sum / count as f32),
        _ => None,
    }
}

fn simulation(n: i32, num: i32) -> Option<f32> {
    let mut vec = Vec::new();
    for _ in 0..n {
        vec.push(pi(num));
    }
    return mean(&vec);
}
```

## 작동 시간 비교

파이썬 파이값 추정과 벤치마크 결과 입니다.

```python
print(simulation(100))
print(simulation(1000))
print(simulation(10000))
3.1418292
3.14167696
3.141648096
```

```bash

------------------------------------------- benchmark: 1 tests ------------------------------------------
Name (time in s)        Min     Max    Mean  StdDev  Median     IQR  Outliers     OPS  Rounds  Iterations
---------------------------------------------------------------------------------------------------------
test                 9.0515  9.4819  9.3240  0.1632  9.3614  0.1651       1;0  0.1072       5           1
---------------------------------------------------------------------------------------------------------

```

rust 코드 실행 결과 입니다.

```bash
100회 시뮬레이션 추정값: 3.141172
100회 시뮬레이션 시간: Some(161.112731ms)
1000회 시뮬레이션 추정값: 3.1418517
10000회 시뮬레이션 추정값: 3.1416712
점 100만개 100회 시뮬레이션 추정값: 3.1415176
점 100만개 1000회 시뮬레이션 추정값: 3.1416123
점 100만개 10000회 시뮬레이션 추정값: 3.1415687
```

파이썬의 경우 평균 9.3초가 걸렸으며, 밀리초로 환산하면 9300밀리초가 됩니다. rust의 경우 약 161 밀리초가 걸렸습니다. 두 코드의 작동 시간 차이는 약 50배 이상으로 어마어마한 차이를 보입니다. 그러나 추정된 값의 차이는 큰 차이가 없는 것으로 보입니다. 두 코드의 파이 추정값은 3.141까지 맞춘것으로 동일하기 때문입니다. 참고로 찍는 점의 개수를 10만개 보다 더 많이 할 경우 파이값의 추정치가 더 정확해지게 됩니다.

그렇다면 시뮬레이션 회수를 늘리면 파이값 추정치는 어떻게 될까요? 러스트의 경우 1000번 시뮬레이션의 경우 3.1418의 값이 나왔습니다. 10000번 시뮬레이션 결과를 보면 파이 추정치 값 3.1416으로 실제 파이값의 3.1415에 근접한 것을 확인할 수 있습니다. 파이썬의 경우도 마찬가지로 1000번 시뮬레이션한 결과 3.1416의 값으로 3.1415 값에 더 가까워졌습니다. 참고로 파이썬의 경우 10000번 시뮬레이션 할 경우 시간이 상당히 오래 소요됩니다. 수행 결과 3.141648096로 추정값 자체는 3.1415로 점차 작아지는 모습을 확인할 수 있습니다.

그러나 한번 가지고는 시뮬레이션 회수의 영향을 알 수 없기 때문에 점 개수가 1000000 일 경우에도 시뮬레이션 회수에 따라 실제 파이값과 같아지는지 rust로 확인해보겠습니다. 파이썬의 경우 점 1000000개 부터 시간이 너무 오래 걸리기 때문에 rust로만 실험을 진행하겠습니다. 점 1000000개 일때 100회 시뮬레이션 결과는 3.1415176 로 3.1415까지 실제 값과 동일합니다. 그러나 1000회 시뮬레이션 결과에서는 3.1416123으로 오히려 실제 값과 멀어진 것을 확인할 수 있습니다. 10000회 결과 3.1415687가 나옵니다. 그러나 이 실험 결과만으로는 시뮬레이션 회수가 늘어나는 것과 파이 추정 값이 실제에 더 가까워지는지에 대한 상관성을 도출하기에는 부족해 보입니다. 결론을 내리자면 점의 개수는 커질 수록 파이 추정치가 파이의 실제 값과 유사해지는 것을 확인할 수 있습니다. 다음 글에서 시뮬레이션 회수와 파이 값 추정의 정확도에 대한 자세한 실험을 통해 알아보도록 하겠습니다.