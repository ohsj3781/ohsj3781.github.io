---
title: "[Alogrithm] LCA(Lowest Common Ancestor) (C/C++)"
date: 2025-10-09 09:57:00 +0900
categories: [Algorithm, algorithm]
tags: [Lowest common ancestors]     # TAG names should always be lowercase
description: ""
math: true
---
> 위글은 위키피디아의 [Lowest common ancestor](https://en.wikipedia.org/wiki/Lowest_common_ancestor)글과 [Tarjan's off-line lowest common ancestors algorithm](https://en.wikipedia.org/wiki/Tarjan%27s_off-line_lowest_common_ancestors_algorithm)글을 참고하여 작성하였습니다.

## LCA(Lowest Common Ancestors)
LCA 알고리즘은 Tree나 DAG내부의 두 Node 간의 최소 공통 조상을 효과적으로 빠르게 찾아 내는 알고리즘이다. 

## 일반적인 LCA알고리즘
가장 먼저 생각 할수 있는 알고리즘은 Tree내 각 Node들의 Parent Node와 Level을 저장 한 후, 임의의 노드 A,B를 선택 한 다음 Node의 Level을 동일 하게 설정한 후, 서로 다르면 각 노드의 Level을 감소 시키며, A,B의 값이 동일 해 지면 해당 노드가 A,B의 LCA가 되는 알고리즘이다.

### LCA알고리즘 1
```cpp
/* level[i] = level of i'th node */
std::vector<int> level;
/* parent[i] =  parent of i'th node */
std::vector<int> parent;

const int LCA(const int A, const int B) {

    /* To set level[A] >= level[B] */
    if (level[A] < level[B]) {
        std::swap(A, B);
    }

    /* Set level[A] and level[B] same */
    while (level[A] != level[B]) {
        A = parent[A];
    }

    while (A != B) {
        A = parent[A];
        B = parent[B];
    }

    return A;
}
```
하지만 이런식으로 코드를 작성하게 되면, 트리의 depth를 h라고 했을 때, O(h)만큼의 시간 복잡도가 발생한 다는 문제가 존재하며, 최악의 경우 트리 전체 노드 갯수를 n이라 했을때 O(n)의 시간 복잡도가 걸린다는 문제가 발생한다.

## 개선된 LCA알고리즘
위에 설명한 문제를 개선하기 위하여 한가지 성질을 이용한다. 임의의 노드 A,B,C가 있다 가정하자. 이때, A의 n'th parent = B이고, B의 n'th parent = C라 가정했을때, A의 2n'th parent = B의 n'th parent라는 성질을 이용한다.

위의 성질을 이용하면 각 노드들이 2의 제곱인 parent만 저장해도 노드의 모든 Parent들에 접근 할 수 있다는 결론이 나오게 된다. {왜 이런 성질이 성립되는지는 2진수의 성질을 생각해보면 된다.}

### LCA 알고리즘 2
```cpp
/* level[i] = level of i'th node */
std::vector<int> level;
/* parent[i][j] = 2^j'th parent of i'th node */
std::vector<std::vector<int>> parent;
/* routes[i][j] =  node 'i' can access to node 'j' */
std::vector<std::vector<int>> routes;

void TraverseTree(const int now_node, const int parent_node) {
    parent[now_node][0] = parent_node;
    for (int i = 1; i < parent[now_node].size(); ++i) {
        parent[now_node][i] = parent[parent[now_node][i - 1]][i - 1];
    }

    for (int i = 0; i < routes[now_node].size(); ++i) {
        const int child_node = routes[now_node][i];

        /* child_node != parent_node */
        if (child_node ^ parent_node) {
            TraverseTree(child_node, now_node);
        }
    }

    return;
}

const int LCA(const int A, const int B) {

    /* To set level[A] >= level[B] */
    if (level[A] < level[B]) {
        std::swap(A, B);
    }

    /* Set level[A] and level[B] same */
    if (level[A] != level[B]) {
        for (int i = parent[A].size() - 1; i >= 0; --i) {
            if (level[parent[A][i]] >= level[B]) {
                A = parent[A][i];
            }
        }
    }

    int return_value = A;
    if (A != B) {
        for (int i = parent[A].size() - 1; i >= 0; --i) {
            if (parent[A][i] != parent[B][i]) {
                A = parent[A][i];
                B = parent[B][i];
            }
            return_value = parent[A][i];
        }
    }

    return return_value;
}
```

## Tarjan's offline Lowest Common Ancestors Algorithm
하지만 위에 개선된 알고리즘은 메모리를 많이 사용한다는 문제 점이 존재한다. 따라서 Tarjan's offline LCA알고리즘을 적용가능한 상황에서는 해당 알고리즘을 적용하는 것이 더 좋다.

### Tarjan's 코드
```cpp
/* query[i][j] = get LCA of i'th node and j'th node */
std::vector<std::vector<int>> query;
/* routes[i][j] =  node 'i' can access to node 'j' */
std::vector<std::vector<int>> routes;
std::vector<int> sets;
std::vector<bool> visitied;

const int Getset(const int now_node) {
    if (sets[now_node] == now_node) {
        return now_node;
    }
    return sets[now_node] = Getset(sets[now_node]);
}

void UnionSet(const int child_node, const int now_node) {
    sets[Getset(child_node)] = Getset(now_node);
}

const int TarjanLCA(const int now_node, const int parent_node) {
    sets[now_node] = now_node;

    for (int i = 0; i < routes[now_node].size(); ++i) {
        const int child_node = routes[now_node][i];

        /* parent_node != child_node */
        if (parent_node ^ child_node) {
            TarjanLCA(child_node, now_node);

            UnionSet(child_node, now_node);
        }
    }

    visitied[now_node] = true;

    for (int i = 0; i < query[now_node].size(); ++i) {
        const int searched_node = query[now_node][i];
        if (visitied[searched_node]) {
            const int LCA = Getset(searched_node);
        }
    }

    return;
}
```