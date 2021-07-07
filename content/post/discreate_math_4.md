---
title: "삽입 정렬(insertion sort)"
date: 2021-07-06T11:40:52+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 삽입 정렬의 시간복잡도

삽입 정렬의 시간복잡도는 입력 자료의 구성에 따라 달라집니다. 최선의 경우 입력 자료가 정렬되어 있는 경우로 첫 번째 인덱스와 각 인덱스 간 n-1 (n은 배열의 크기)번의 비교만 이루어지기에 시간 복잡도는 n이 됩니다. 그러나 최악의 경우는 입력 자료가 역순으로 배치되어 있을 때 입니다. 이 때의 시간 복잡도는 n²이 되는데 버블 정렬과 마찬가지로 첫 번째 인덱스부터 n-1 번 이어서 나머지 인덱스들의 비교횟수는 n-2 ... 1 번 비교하게 됩니다. 이에 따라 이 비교 횟수의 합은 (n*(n-1))/2 가 되고 시간복잡도는 n²이 됩니다.

## 문제풀이

233 쪽 예제 5: 3,2,4,1,5,를 오름차순으로 정렬하라.

```Rust
fn main(){
    let mut tmp: Vec<i32> = vec![3,2,4,1,5];
        
    println!("{:?}",insertion_sort(&mut tmp));
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
```