---
title: "선형 탐색"
date: 2021-07-04T13:15:12+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 선형 탐색

선형 탐색의 시간 복잡도는 최상의 경우 1이며 찾는 원소가 첫 번째에 있는 경우, 최악의 경우 n으로 찾는 원소가 마지막에 있는 경우 입니다. 평균적인 시간 복잡도는 가장 자주 사용되는 최악인 경우의 시간 복잡도를 따라갑니다. 그렇기 때문에 시간 복잡도는 n이 됩니다.

230 페이지의 예제3 다음 리스트에서 19를 찾아보자.
리스트:[1, 2, 3, 5, 6, 7, 8, 10, 12, 13, 15, 16, 18, 19, 20, 22]

```Rust
use std::time::Instant;
fn main(){
    let mut tmp: Vec<i32> = vec![1, 2, 3, 5, 6, 7, 8, 10, 12, 13, 15, 16, 18, 19, 20, 22];
    
    println!("{}",linear_search(& mut tmp,19));
    println!("{}",linear_search_re(& mut tmp,0,16,19));
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

```
