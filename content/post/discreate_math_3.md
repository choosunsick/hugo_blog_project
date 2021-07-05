---
title: "이산 수학 정리 3 - 버블정렬"
date: 2021-07-05T17:26:55+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 버블 정렬(bubble sort)

버블 정렬 시간복잡도는 최선의 경우, 최악의 경우, 평균적인 경우 모두 일정합니다. 어떤 경우에도 버블 정렬은 마지막 인덱스까지 두 개의 원소를 비교하기에 항상 n²(n은 배열의 크기) 만큼의 순회가 필요하기 때문입니다. 시간복잡도를 계산해봅시다. 먼저 배열의 각 인덱스를 비교할 때 첫번째 인덱스의 경우 나머지 인덱스들과의 비교를 위해 n-1 번의 비교가 필요합니다. 이후 두번째, 세번째 등 나머지 인덱스에 대해서 비교해나가면 n-2, n-3 ... 1 까지의 비교횟수가 늘어나게 되며 이것의 총합은 (n*(n-1))/2 가 됩니다. 이에 따라 시간복잡도는 n²이 됩니다.

## 풀 문제

232 쪽 예제 4: 버블정렬을 사용하여 3,2,4,1,5,를 오름차순으로 정렬하라.

```Rust
fn main(){
    let mut tmp: Vec<i32> = vec![3,2,4,1,5];
        
    println!("{:?}",bubble_sort(&mut tmp));
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

```