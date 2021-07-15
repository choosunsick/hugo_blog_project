---
title: "파이썬과 Rust 프로그래밍 코드 성능 비교"
date: 2021-07-14T17:33:06+09:00
draft: False
tags: ["이산수학","Rust","Python"]
categories: ["이산수학","Rust","Python"]
---

파이썬과 Rust 프로그래밍 간 내장 정렬, 버블, 삽입, 합병, 퀵 정렬 알고리즘을 구현하고 서로 작동 시간을 비교해 봅니다. 공정한 비교를 위해 10만개의 정렬되지 않은 텍스트 데이터를 만들고 저장한 다음 파이썬과 Rust에서 각각 읽어 온 다음 정렬을 진행하였습니다.

## 파이썬과 러스트 시간 비교 결과

밑에 구현 코드를 작동한 결과를 토대로 비교합니다.
첫 번째 결과가 파이썬 코드의 작동 시간이고 두 번째 결과가 Rust 코드의 작동시간입니다.

```bash

----------------------------------------------------------------------------------- benchmark: 3 tests ----------------------------------------------------------------------------------
Name (time in ms)          Min                 Max                Mean            StdDev              Median               IQR            Outliers      OPS            Rounds  Iterations
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
test_sort              18.2152 (1.0)       23.5398 (1.0)       19.3755 (1.0)      1.3161 (1.40)      18.8257 (1.0)      1.5104 (1.08)          6;3  51.6116 (1.0)          51           1
test_quick_sort       303.8617 (16.68)    309.0250 (13.13)    305.5513 (15.77)    2.0085 (2.13)     304.8247 (16.19)    1.6705 (1.19)          1;1   3.2728 (0.06)          5           1
test_merge_sort       469.5359 (25.78)    471.8063 (20.04)    470.4240 (24.28)    0.9423 (1.0)      470.5774 (25.00)    1.4021 (1.0)           1;0   2.1257 (0.04)          5           1
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

```bash
내장 정렬 함수의 걸린 시간: Some(5.88405ms)
버블 정렬 함수의 걸린 시간: Some(20.083627604s)
삽입 정렬 함수의 걸린 시간: Some(3.957112085s)
합병 정렬 함수의 걸린 시간: Some(91.605143ms)
퀵 정렬 함수의 걸린 시간: Some(77.738477ms)
```

위의 벤치마크 표를 해석해보겠습니다. 먼저 파이썬의 내장 정렬 함수는 평균적으로 19 밀리초가 걸리며 최대 23밀리초까지 작동시간이 걸립니다. 반면 러스트의 경우 5 밀리초가 걸립니다. 파이썬 보다 Rust의 내장 정렬 함수가 4배 정도 더 빠르게 작동합니다. 그 다음은 버블 정렬과 삽입 정렬을 파이썬 코드와 Rust 코드 간 비교를 해야합니다. 그러나 버블 정렬과 삽입 정렬의 경우 파이썬에서 작동하는데 30분 이상의 시간이 소요 되었기 때문에 시간 비교가 무의미 합니다.

합병 정렬 함수의 시간 비교를 해보자면 파이썬의 경우 평균적으로 470 밀리초가 걸렸습니다. 합병 정렬의 경우 최대 최소 시간과 약 1 밀리초 정도밖에 차이가 나지 않습니다. 반면 Rust의 합병 정렬의 경우 91 밀리초가 걸렸습니다. 내장 정렬 함수 때와 마찬가지로 약 5배 가량 러스트가 더 빠르게 작동합니다. 다음으로 퀵 정렬은 파이썬 코드의 경우 평균적으로 305 밀리초가 걸립니다. 최대 309 밀리초까지 걸릴 수 있습니다. Rust의 퀵 정렬의 경우 77 밀리초가 걸렸습니다. 퀵 정렬의 경우도 파이썬 코드 보다 Rust 코드가 약 4배 가량 더 빠르게 작동합니다. 결론적으로 성능 면에서 보자면 모든 부분에서 Rust 프로그래밍 코드가 파이썬 프로그래밍 코드에 비해 약 4배 정도 더 우수하다는 것을 알 수 있습니다.

## 정렬 알고리즘 파이썬 구현

기존에 소개한 버블, 삽입, 합병, 퀵 정렬 알고리즘을 파이썬으로 구현합니다. 그리고 정렬되지 않은 10만개의 숫자 리스트를 불러와 작동 시간을 검사합니다. 아래 코드를 실행하시려면 “pytest-benchmark” 패키지를 설치하셔야 됩니다. 설치 방법은 다음과 같습니다: `pip3 install pytest-benchmark` 패키지를 설치한 다음 아래 코드를 benchmark.py 파일로 저장합니다. 이후 테스트 실행 방법은 pytest --benchmark-only benchmark.py를 터미널에서 입력해주시면 됩니다.

```python
import csv

def bubble_sort(sample):
    for i in range(len(sample)-1):
        for j in range(len(sample)-1):
            if sample[j] > sample[j+1]:
                sample[j],sample[j+1] = sample[j+1],sample[j]
    return sample

def insertion_sort(unsort):
    size = len(unsort)
    for i in range(size):
        for j in range(i):
            if unsort[i] < unsort[j]:
                unsort.insert(j,unsort.pop(i))
                break
    return unsort

def merge(left,right):
    sort = []
    i, j = 0, 0
    left_size = len(left)
    right_size = len(right)
    while left_size > i and right_size > j:
        if left[i] > right[j]:
            sort.append(right[j])
            j += 1
        else:
            sort.append(left[i])
            i += 1
    
    if left_size > i:
        sort.extend(left[i:])
    elif right_size > j:
        sort.extend(right[j:])
    return sort

def merge_sort(unsort):
    size = len(unsort)
    if size > 1:
        mid = size // 2
        return merge(merge_sort(unsort[:mid]),merge_sort(unsort[mid:]))
    else:
        return unsort

def split(pivot,unsort):
    pivot_list = []
    left, right = [], []
    for x in unsort:
        if x < pivot:
            left.append(x)
        elif x > pivot:
            right.append(x)
        else:
            pivot_list.append(x)
    return (left, right)

def quick_sort(unsort):
    size = len(unsort)
    if size > 1:
        pivot = unsort[size//2] #가장 첫 번째를 pivot으로 사용
        left,right = split(pivot,unsort)
        return quick_sort(left)+[pivot]+quick_sort(right)
    else:
        return unsort


with open("temp_100000.txt","rt") as file: 
    cln = csv.reader(file)
    tmp = [row for row in cln]

temp  = tmp[0]
test = [int(i) for i in temp]


def fun_sort():
    return sorted(test)

def fun_mergesort():
    return merge_sort(test)

def fun_quicksort():
    return quick_sort(test)

def test_sort(benchmark):
    benchmark(fun_sort)

def test_merge_sort(benchmark):
    benchmark(fun_mergesort)

def test_quick_sort(benchmark):
    benchmark(fun_quicksort)
```

## 정렬 알고리즘 Rust 구현

```Rust
use std::fs;
use std::time::Instant;
use rand::{seq, thread_rng};

fn main(){
    let data = fs::read_to_string("temp_100000.txt").expect("Unable to read file");
    
    let temp_data: Vec<&str> = data
       .split(|c| c == ',')
        .collect();

    let numbers: Result<Vec<i32>, _> = temp_data
        .iter()
        .map(|x| x.parse())
        .collect();

    let mut tmp = &mut numbers.unwrap();
    
    let now = Instant::now();
    tmp.clone().sort();
    let after = Instant::now();
    println!("내장 정렬 함수의 걸린 시간: {:?}", after.checked_duration_since(now));
    
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
    quick_sort(tmp.to_vec());
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
