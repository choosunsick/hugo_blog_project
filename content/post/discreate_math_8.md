---
title: "정렬 알고리즘 총 정리"
date: 2021-07-10T17:44:40+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 정렬 알고리즘 시간 비교

Rust 내장 정렬, 버블 정렬, 삽입 정렬, 합병 정렬, 퀵 정렬 함수 간 시간 차이를 비교합니다. 버블 정렬과 삽입 정렬 간의 시간 복잡도가 n²으로 같고 합병 정렬과 퀵 정렬의 시간 복잡도가 nlog n으로 같습니다. 같은 시간 복잡도를 가지는 두 알고리즘 중 어느 알고리즘이 더 빠른지 확인 해보겠습니다.

## 시간 비교 결과

아래 문제 풀이 코드를 실행하여 제 경우에는 다음과 같은 결과가 나왔습니다.

```bash
내장 정렬 함수의 걸린 시간: Some(7.439752ms)
버블 정렬 함수의 걸린 시간: Some(20.786836236s)
삽입 정렬 함수의 걸린 시간: Some(3.842206293s)
합병 정렬 함수의 걸린 시간: Some(89.799089ms)
퀵 정렬 함수의 걸린 시간: Some(78.114541ms)
```

내장 정렬 함수가 역시 가장 빠르게 작동하는 것을 확인할 수 있습니다. 약 7.5 밀리초로 0.007초가 걸렸습니다. 가장 오래 걸린 정렬 함수는 버블 정렬입니다. 약 20초가 걸렸는데 이는 같은 시간 복잡도를 가지는 삽입 정렬이 약 3초가 걸린 것에 6배 가량 느린 것입니다. 내장 정렬 함수에 비하면 단위가 초와 밀리초로 차이나기 때문에 엄청나게 차이가 난다고 볼 수 있습니다. 버블 정렬의 경우 약 2800배, 삽입 정렬의 경우 약 400배 정도 차이가 납니다.

n log n 의 시간 복잡도를 가지는 합병 정렬과 퀵 정렬은 모두 내장 정렬과 단위가 같은 밀리초로 나옵니다. 그러나 걸린 시간에서 차이가 있습니다. 퀵 정렬이 78 밀리초로 0.078 초가 걸렸습니다. 그리고, 89 밀리초가 걸린 합병 정렬이 0.089초가 걸립니다. 여러번 반복해도 퀵 정렬의 경우 약 70 밀리초 정도 소요되고 합병 정렬의 경우 약 80 밀리초 가량이 소요되는 것을 확인할 수 있습니다. 이에 따라 퀵 정렬이 합병 정렬 보다 빠르게 작동합니다.

가장 빠른 내장 정렬 함수의 경우 퀵 정렬보다 약 10배 가량 빠르게 작동합니다. 최종적으로 정리하면 내장 정렬 함수가 가장 빠르고 그 다음이 퀵 정렬, 합병 정렬 그리고 삽입 정렬, 버블 정렬 순서로 빠르게 작동합니다. 단 삽입 정렬과 버블 정렬의 경우 내장 정렬 함수나, 퀵 정렬, 합병 정렬에 비해 압도적으로 느리게 작동합니다. 이는 알고리즘이 가지는 시간복잡도에서 차이가 나기 때문입니다. 참고로 내장 정렬 함수의 알고리즘과 관련 내용은 [링크](https://doc.rust-lang.org/std/primitive.slice.html#method.sort) 에서 찾아 보실 수 있습니다.

## 문제풀이

아래 코드를 실행하기에 앞서 rand crate 사용을 위해 cargo.toml에 [dependencies]란에 `rand = "0.8"`를 추가해주어야 합니다.

랜덤으로 샘플링된 0에서 99999까지의 숫자 리스트를 정렬하라.

```Rust
use std::fs;
use std::time::Instant;
use rand::{seq, thread_rng};

fn main(){
    
    let mut rng = thread_rng();
    let sample = seq::index::sample(&mut rng, 100000, 100000);
    let mut sample_v = sample.into_vec();
    let now = Instant::now();
    sample_v.clone().sort();
    let after = Instant::now();
    println!("내장 정렬 함수의 걸린 시간: {:?}", after.checked_duration_since(now));
    
    let mut tmp: Vec<i32> = Vec::new();
    for i in sample_v{
        tmp.push(i as i32)
    }

    let now = Instant::now();
    bubble_sort(&mut tmp.clone());
    let after = Instant::now();
    println!("버블 정렬 함수의 걸린 시간: {:?}", after.checked_duration_since(now));

    let now = Instant::now();
    insertion_sort(&mut tmp.clone());
    let after = Instant::now();
    println!("삽입 정렬 함수의 걸린 시간: {:?}", after.checked_duration_since(now));

    let now = Instant::now();
    merge_sort(&mut tmp);
    let after = Instant::now();
    println!("합병 정렬 함수의 걸린 시간: {:?}", after.checked_duration_since(now));

    let now = Instant::now();
    quick_sort(tmp);
    let after = Instant::now();
    println!("퀵 정렬 함수의 걸린 시간: {:?}", after.checked_duration_since(now));
    
}

fn bubble_sort(vector: &mut Vec<i32>) -> Vec<i32>{
    
    for _ in 0..(vector.len () - 1){
        for j in 0..(vector.len () - 1){
            if vector[j] > vector[j+1]{
                let sorted_vec_1 = vector[j+1];
                let sorted_vec_2 = vector[j];
                vector[j] = sorted_vec_1;
                vector[j+1] = sorted_vec_2;
            }
        }
    }
    return vector.to_vec()
}
    
fn insertion_sort(vector: &mut Vec<i32>) -> Vec<i32>{
    for i in 0..vector.len(){
        for j in 0..i{
            if vector[i] < vector[j]{
                let sort_vec = vector[i];
                vector.remove(i);
                vector.insert(j,sort_vec);
                break
            }
        } 
    }
    return vector.to_vec()   
}

fn merge(left: &mut Vec<i32>,right: &mut Vec<i32>) -> Vec<i32>{
    let mut sort = Vec::new();
    let mut i: usize = 0;
    let mut j: usize = 0;
    let left_size = left.len();
    let right_size = right.len();
    while left_size > i && right_size > j{
        if left[i] > right[j]{
            sort.push(right[j]);
            j += 1;
        }
        else{
            sort.push(left[i]);
            i += 1;
        }
    }
    if left_size > i{
        for k in i..left_size{
            sort.push(left[k]);
        }
    }
    else if right_size > j{
        for k in j..right_size{
            sort.push(right[k]);
        }
    }
    return sort
}

fn merge_sort(unsort:&mut Vec<i32>) -> Vec<i32>{
    let size = unsort.len();
    if size > 1{
        let mid = size / 2;
        return merge(&mut merge_sort(&mut unsort[0..mid].to_vec()),&mut merge_sort(&mut unsort[mid..size].to_vec()))
    }
    else{
        return unsort.to_vec()
    }
}

fn quick_sort(unsort:Vec<i32>) -> Vec<i32>{
    let size = unsort.len();
    let mut sort = Vec::new();
    if size > 1{
        let mid = size / 2;
        let pivot = unsort[mid];
        let mut left = Vec::new();
        let mut pivot_list = Vec::new();
        let mut right = Vec::new();
        for i in unsort{
            if i < pivot{
                left.push(i);
            }
            else if i > pivot{
                right.push(i);
            }
            else{
                pivot_list.push(i);
            }
        }
        sort.append(&mut quick_sort(left));
        sort.append(&mut pivot_list);
        sort.append(&mut quick_sort(right));
        return sort
    }
    else{
        return unsort
    }
}
```

