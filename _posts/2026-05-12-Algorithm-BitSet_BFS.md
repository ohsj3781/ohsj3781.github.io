---
title: "[Alogrithm] BitSet BFS(Breadth First Search) (C/C++)"
date: 2026-05-12 17:14:42 +0900
categories: [Algorithm, algorithm]
tags: [breadth first search, cpp]     # TAG names should always be lowercase
description: ""
math: true
---
> cpp에서 구현하는 방법입니다. 타 언어에서는 구현 해 보지 않았습니다. 

## BFS(Breadth-First Search)
BFS 알고리즘은 그래프를 탐색 하는데 있어서 깊이가 아닌 너비를 우선 하여 탐색 하는 알고리즘이다.
일반적으로는 queue를 이용하여 구현한다. 

## BitSet BFS 
이러한 BFS 알고리즘을 변형시켜 BitSet을 이용하여 BFS를 구현 할 수 있다.
BitSet에다가 같은 Depth를 가진 Node들을 저장시켜서 최적화 한다.
다만 제약 조건이 존재한다. **간선에 가중치가 없어야 한다.** 즉 간선은 연결 됬는지 안됬는지만 확인한다.

### 최적화 이유
BitSet BFS가 일반적인 BFS보다 최적화 되는 이유는 같은 Depth에 있는 노드들을 동시에 탐색 하는 효과가 일어나기 때문이다. BFS알고리즘의 특성상 n'th depth의 노드들을 탐색 한다면 n-1'th 노드는 모두 다 탐색 되어있고, n+1'th 노드들은 n'th depth의 노드들이 다 탐색 된 다음에 탐색 된다. 이러한 성질을 활용 하여서 n'th depth에 있는 노드들을 동시에 저장 한 후, n'th depth의 노드의 edge들을 이용하여 n+1'th depth의 노드들을 탐색 하게 된다면 실제 일어나는 operation들을 줄일 수 있다.

### 추가적인 최적화 이유
std::Bitset은 내부적으로 컴퓨터의 Word Size단위로 연산 하게 된다. 즉 64bit, Linux GCC 환경에서는 64개의 노드들을 동시에 처리 할 수 있게된다. 따라서 O(N^2)알고리즘을 BitSet BFS를 이용하게 된다면 O(N^2/64)의 시간 복잡도를 가지고 처리 할 수 있게된다.

## 변형 제안
만약 간선의 가중치가 양수이고 예를들어 가중치의 최댓값이 10이라면 dp를 사용하지 않고 BitSet BFS를 이용하여서 최적화가 가능하다는 생각이 들었다. 조건에 따라 다르겠지만 조금 더 빨라질거 같다는 느낌이다.

## BitSet BFS 코드
``` cpp
int BitSetBFS(int src) {
  std::bitset<MAX_N> visited, cur, next;
  cur.set(src); // 현재 탐색할 노드 집합 (현재 depth의 레이어)

  int cc_sum = 0, depth = 0;
  while (cur.any()) {
    visited |= cur; // 방문한 노드 마킹
    ++depth;        // 거리 증가
    next.reset();   // 다음 탐색 노드 집합 초기화

    for (int i = 0; i < N; ++i) {
      // 만약 현재 i'th 노드를 탐색 중이라면 다음 탐색 노드에 현재 노드와 연결된
      // edges를 추가시킨다.
      if (cur[i]) {
        next |= edges[i];
      }
    }

    // 이미 방문한 노드들은 제외 시킨다.
    next &= ~visited;
    cc_sum += next.count() * depth;

    cur = next;
  }

  return cc_sum;
}
```