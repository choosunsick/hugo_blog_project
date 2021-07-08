---
title: "재귀 알고리즘"
date: 2021-07-08T17:06:49+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 재귀 알고리즘의 장점과 단점

선형 탐색과 이진 탐색 알고리즘은 재귀를 통해서도 구현될 수 있습니다. 재귀적 알고리즘과 기존의 알고리즘 간 시간 복잡도는 달라지지 않습니다. 하지만, 재귀 알고리즘의 경우 기존 반복문에 비해 내부 메모리 공간을 사용하기에 더 느리게 작동하는 경우가 많습니다. 조건을 잘 못 설정하면 무한 재귀가 발생하여 메모리 제한에 걸려 stack overflow 에러가 발생할 위험성도 있습니다. 이런 단점 말고 장점도 존재합니다. 재귀 알고리즘을 사용하면 반복문에 비해 코드 가독성이 좋아지는 점과 변수를 적게 사용하여 side effect가 없다는 장점이 있습니다.

## 시간 비교 결과

아래 문제 풀이 코드를 실행하면 제 경우에는 다음과 같은 결과가 나왔습니다.

78920
Some(84.178µs)
78920
Some(43.776µs)
78920
Some(1.461µs)
78920
Some(1.219µs)

첫 번째는 재귀적 선형 탐색 알고리즘의 결과로 84 마이크로 초가 나와 가장 오래 걸렸습니다. 그리고 반복문을 사용한 선형 탐색 알고리즘이 43 마이크로초가 걸리는 것으로 재귀에 비해 약 2배 정도 빠르다는 것을 알 수 있습니다. 이어서 재귀적 이진 탐색 알고리즘이 1.4 마이크로 초가 걸린 것으로 기존 선형 탐색 알고리즘들 보다 압도적으로 빠른 것을 확인할 수 있습니다. 반복문을 통한 이진 탐색 알고리즘의 경우 1.2 마이크로초로 재귀의 경우와 0.2 마이크로초 차이지만 이 중에서 가장 빠른 탐색 알고리즘입니다. 참고로 이진 탐색 알고리즘의 경우 정렬된 리스트에서만 작동합니다.

## 문제풀이

이제 실제로 반복문에 비해 재귀 알고리즘이 더 느린지 두 알고리즘 간 시간 차이를 알아봅시다.

문제는 임의로 만들어 보았습니다.

1에서 100000까지 정렬된 리스트에서 78921을 찾아라.

```Rust
use std::time::Instant;
fn main(){
    let mut tmp: Vec<i32> = Vec::new();
    for i in 1..100000{
        tmp.push(i)
    }

    let now = Instant::now();
    println!("{}",linear_search_re(& mut tmp,0,100000,78921));
    let after = Instant::now();
    println!("{:?}", after.checked_duration_since(now));
    
    let now = Instant::now();
    println!("{}",linear_search(& mut tmp,78921));
    let after = Instant::now();
    println!("{:?}", after.checked_duration_since(now));

    let now = Instant::now();
    println!("{}",binary_search_re(& mut tmp,0,100000,78921));
    let after = Instant::now();
    println!("{:?}", after.checked_duration_since(now));
    
    let now = Instant::now();
    println!("{}",binary_search(& mut tmp,78921));
    let after = Instant::now();
    println!("{:?}", after.checked_duration_since(now));

}


fn linear_search_re(v:&mut Vec<i32>, i:usize,j:usize,x:i32) -> usize {
    let tmp = v[i];
    if tmp == x{
        return i
    }
    else if i == j{
        return 0
    }
    else{
        return linear_search_re(v,i+1,j,x)
    }
}

fn linear_search(v:&mut Vec<i32>, x:i32) -> usize{
    let mut i = 1;
    let n = v.len();
    while i <= n && x != v[i]{
        i += 1;
    }
    if i <= n{
        let location = i;
        return location
    }
    else{
        let location = 0;
        return location
    }
}
fn binary_search_re(v:&mut Vec<i32>, i:usize,j:usize,x:i32) -> usize{
    let m: usize = (i+j)/2;
    let tmp = v[m];
    if v[m] == x{
        return m
    }
    else if x < v[m] && i < m{
        return binary_search_re(v,i,m-1,x)
    }
    else if x > v[m] && j > m{
        return binary_search_re(v,m+1,j,x)
    }
    else{
        return 0
    }
}

fn binary_search(v:&mut Vec<i32>,x:i32) -> usize{
    let mut i = 1;
    let mut j = v.len();

    while i < j{
        let m: usize = (i+j)/2;
        if x > v[m]{
            i = m + 1;
        }
        else{
            j = m;
        }
    }
    if x == v[i]{
        let location = i;
        return location
    }
    else{
        let location = 0;
        return location
    }
}
```
