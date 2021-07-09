---
title: "퀵 정렬(quick sort)"
date: 2021-07-09T15:57:20+09:00
draft: False
tags: ["이산수학","Rust"]
categories: ["이산수학","Rust"]
---

3월부터 공부한 이산수학 알고리즘 개념들을 정리합니다. 교재는 Rosen의 이산수학 8판 입니다.

## 퀵 정렬의 시간복잡도

퀵 정렬은 평균적으로 nlog n의 시간복잡도를 가집니다. 퀵 정렬 역시 합병 정렬과 유사하게 나누고 다시 합치는 분할 정복 방식으로 정렬이 이루어집니다. 단 퀵 정렬의 경우 2개 씩 나누어지지만 pivot 값에 따라 나누어지는 개수가 절반씩 나누어지는 것이 아니라 1개와 나머지가 나오는 경우가 발생할 수 있습니다.

먼저 절반씩 나누어지는 경우를 살펴보면 처음에 n/2가 2개 생기게 됩니다. 두 번째에 n/4가 4개 나타나게 됩니다. 이렇게 계속 진행하여 1개씩 n개가 될 때까지 진행합니다. 이 나누는 과정을 일반화하면 n/(2^k)가 됩니다. 각 나누는 과정에서 비교하여 정렬하는데 걸리는 복잡도는 n번 발생하게 됩니다.k 값은 합병 정렬에서와 마찬가지로 2를 밑으로 하는 log n 값을 가지게 됩니다. 이는 분할 정복 방식의 특징이기도 합니다. 이에 따라 비교 비용 n을 곱해서 최종적으로 nlog n의 시간복잡도를 가지게 됩니다.

물론 최악의 경우 즉 분할이 절반씩 되는 경우가 아닌 1개와 나머지로 되는 경우가 있습니다. 이것은 데이터가 미리 정렬되어 있거나 역순으로 정렬된 경우 입니다. 이 때의 분할의 깊이는 n이 되는데 그 이유는 1개와 나머지로 나누면 1개씩 n개가 될 때까지 비용이 n이기 때문입니다. 비교 비용은 n번으로 같기에 최악의 경우 퀵정렬의 시간복잡도는 n²이 됩니다. 보통 최악의 경우가 가지는 시간복잡도를 따라가지만 최악의 경우가 2가지 경우 밖에 없기에 평균적인 시간복잡도인 nlog n이 퀵 정렬의 시간복잡도가 됩니다.

## 문제풀이

435쪽 문제 50: 퀵 정렬을 이용하여 3,5,7,8,1,9,2,4,6을 정렬하라.

```Rust
fn main(){
    let mut tmp: Vec<i32> = vec![3,5,7,8,1,9,2,4,6];
        
    println!("{:?}",quick_sort(tmp));
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
