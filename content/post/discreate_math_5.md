---
title: "합병 정렬(merge sort)"
date: 2021-07-07T17:16:43+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 합병 정렬의 시간복잡도

합병 정렬의 시간복잡도는 최선의 경우와 최악의 경우 평균인 경우가 전부 같습니다. 전부 nlog n이 됩니다. 어떻게 해서 n log n 이 되는지 살펴보겠습니다. 합병 정렬은 데이터의 개수를 2의 배수 개씩 최종적으로 1개가 될때까지 나누어집니다. 이 과정을 보면 처음에 나눌때 n개에서 n/2개가 2개가 생기고 이후 n/4개가 4개가 되며 나아가면, 1개씩 n개가 됩니다. 데이터의 개수가 n개에서 1개씩 n개가 되는 과정은 합병 정렬에서 나누어질때와 합쳐질 때 발생하게 됩니다. 데이터가 합쳐지는 과정에서 정렬이 이루어지면서 kn(k는 데이터가 나누어지거나 합쳐지는 과정의 길이) 만큼의 복잡도가 소요됩니다. 이에 따라서 시간복잡도 식은 (2^k)*(n/(2^k)) +kn이 됩니다.

k 값은 최대 2를 밑으로 하는 log n 값을 가질 수 있습니다. 데이터가 8개인 경우를 가정해보면 알 수 있습니다. 먼저 처음으로 나누어질때 4개가 2개씩 나누어지면서 k=1이 됩니다. 두번째에는 2개가 4개씩 나누어지면서 k=2가 됩니다. 마지막으로 1개가 8개씩 나누어지면서 k=3이 되는데 이 3이라는 값은 2를 밑으로 하는 log 8 값과 동일한 값이 됩니다. 이어서 위의 식에 있는 k에 2를 밑으로 하는 log n 값을 대입하여 정리하면 최종적으로 n+n*log n(밑은 2)가 나오고 빅오를 적용하면 시간복잡도는 n log n이 됩니다.

## 문제풀이

429 쪽 예제 9: 합병 정렬을 이용하여 리스트 8,2,4,6,9,7,10,1,5,3을 오름차순으로 정렬하라  

```Rust
fn main(){
    let mut tmp: Vec<i32> = vec![8,2,4,6,9,7,10,1,5,3];
        
    println!("{:?}",merge_sort(&mut tmp));
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
```