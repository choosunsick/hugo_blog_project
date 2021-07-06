---
title: "이진 탐색(binary_search)"
date: 2021-07-04T21:12:31+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 이진 탐색의 시간복잡도

이진 탐색의 시간 복잡도는 최상의 경우 1이며 찾는 원소가 중간에 있는 경우입니다. 최악의 경우의 연산 횟수를 계산해보면, 이진 탐색은 한번 비교할 때마다 데이터가 반으로 줄어듭니다. 이에 따라 데이터 개수가 n일 때 부터 1이 될 때까지 k번 비교하여 (n/2)^k 번이 됩니다. 이제 k에 대하여 식을 정리해보면 2^k = n이 되고 양쪽에 밑을 2로 하는 log를 붙여주면 k는 2를 밑으로 하는 log n이 됩니다. 이에 따라 이진 탐색의 시간 복잡도는 log n 이 됩니다.

참고로 이진탐색 알고리즘의 경우 사용할 때 정렬된 리스트에서만 사용해야 한다는 주의점이 존재합니다.

선형탐색에서 풀었던 문제와 같은 문제를 풀어보겠습니다.
230 쪽 예제3: 다음 리스트에서 19를 찾아보자.

리스트:[1, 2, 3, 5, 6, 7, 8, 10, 12, 13, 15, 16, 18, 19, 20, 22]

```Rust
fn main(){
    let mut tmp: Vec<i32> = vec![1, 2, 3, 5, 6, 7, 8, 10, 12, 13, 15, 16, 18, 19, 20, 22];
    
    println!("{}",binary_search(& mut tmp,19));
    println!("{}",binary_search_re(& mut tmp,0,16,19));
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