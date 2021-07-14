---
title: "피보나치 수열"
date: 2021-07-13T19:41:57+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 피보나치 수열 풀이 방식 비교

피보나치 수열을 계산하는 알고리즘에는 3 가지가 있습니다. 일반적인 재귀 방식 풀이, 반복문을 이용한 풀이, 동적 계획법(bottom-up 방식)을 이용한 풀이가 있습니다. 이 3 가지 방식을 이용하여 피보나치 수열을 구할 때 어느 방식의 풀이가 가장 시간이 적게 걸리는지 비교해 보도록 하겠습니다.

동적계획법이란 문제를 하위 문제로 나누고 이후 하위 문제들을 결합하여 최종 문제를 푸는 방법입니다. 동적계획법의 장점은 하위 문제들의 풀이를 저장에 있습니다. 문제를 푸는 과정에서 진행했던 연산이 다시 한번 필요한 경우 새로 계산하지 않고 저장된 값을 사용하여 중복 계산을 하지 않아 연산 시간을 줄일 수 있습니다.

## 시간비교

문제 풀이 코드를 작동한 결과 저는 아래와 같은 결과가 나왔습니다.

```bash
6765
피보나치 재귀함수의 걸린 시간: Some(62.309µs)
6765
반복문 피보나치 함수의 걸린 시간: Some(10.013µs)
6765
동적계획법 피보나치 함수의 걸린 시간: Some(7.197µs)
```

3가지 방식 중 재귀를 사용한 방식의 경우가 가장 시간이 오래 걸렸습니다. 약 62 마이크로 초가 걸렸습니다. 일반적인 반복문을 이용한 경우 10 마이크로 초가 걸렸습니다. 재귀를 이용한 경우보다 약 6배 가량 더 빠르게 작동합니다. 가장 빠르게 작동한 경우는 동적계획법을 이용한 경우는 7 마이크로초가 걸려서 가장 적은 시간이 걸립니다.
반복문을 이용한 방식과 약 3 마이크로초 차이로 큰 차이는 안나지만 좀 더 빠르게 작동한다는 점은 분명합니다. 이러한 차이가 발생하는 이유는 미리 만들어진 것과 만들어나가는 방식 간 차이라 생각됩니다. 동적계획법에 따른 피보나치 수열 계산 방법이 가장 빠른 풀이 방식이 되고 재귀 알고리즘을 이용한 풀이 방식이 가장 느린 풀이 방식이 됩니다.

## 문제풀이

피보나치 수열에서 20번째 수를 구하라

```Rust
use std::fs;
use std::time::Instant;
fn main() {
    let now = Instant::now();
    println!("{:?}",fibonacci_re(20))
    let after = Instant::now();
    println!("피보나치 재귀함수의 걸린 시간: {:?}", after.checked_duration_since(now));
    let now = Instant::now();
    println!("{:?}",fibonacci(20))
    let after = Instant::now();
    println!("반복문 피보나치 함수의 걸린 시간: {:?}", after.checked_duration_since(now));
    let now = Instant::now();
    println!("{:?}",fibonacci_dp(20));
    let after = Instant::now();
    println!("동적계획법 피보나치 함수의 걸린 시간: {:?}", after.checked_duration_since(now));
}


fn fibonacci_re(n: i32) -> i32 {
    if n < 2 {
        return n
    } else {
        return fibonacci_re(n - 1) + fibonacci_re(n - 2)
    }
}
fn fibonacci(n:usize) -> i32{
    if n == 0{
        return 0
    }
    else{
        let mut x = vec![0];
        let mut y = vec![1];
        for i in 0..n-1{
            let z = x[i] + y[i];
            x.push(y[i]);
            y.push(z);
        }
        return y[y.len()-1]
    }
}

fn fibonacci_dp(n:usize) -> i32 {
    let mut memo = vec![0; n+1];
    memo[1] = 1;    
    for i in 2..n+1{
        memo[i] = memo[i-1]+memo[i-2];
    }
    return memo[n]
}

```