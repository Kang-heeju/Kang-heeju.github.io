---
title: "자료구조(Data Structure)- 2: Non-Linear Data Structure"
last_modified_date: "2024-03-30 10:32:13"
parent: 알고리즘 개념
grand_parent: Algorithm
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">자료구조-2: Non-Linear Data Structure(트리, 그래프)</div>

{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

저번 시간에는 자료구조의 선형 자료구조(linear data structure)에 대해 알아봤습니다. 

[자료구조(Data Structure) - 1: Linear Data Structure](https://bestdeveloperever.tistory.com/3)



이번 시간에는 non-linear 한 자료구조에 대해 알아보겠습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbYDmx7%2FbtsCLo3CXQM%2FwZwYsJh8sEaeu2mmnRFQdk%2Fimg.png)



## 1. 그래프(Graph)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbXA5CF%2FbtsC1HJM2Lf%2FU4xMN7KNfFKQC6UyZSbwuK%2Fimg.png)



그래프는 노드(vertices)와 간선(edge)들로 이루어져 있는 자료구조이다. 

간선들은 노드를 연결하는 데 사용된다. 

그래프의 종류에는 두가지가 있다. (directed, undirected)



### 1-1. 무방향 그래프(undirected graph)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbRRLMU%2FbtsC6qfStGg%2FpqJWfsnnekGVvFZf0RSpTk%2Fimg.png)



말 그대로 방향성이 없는 그래프 구조를 말한다. 두 정점을 연결하는 간선에 방향이 존재하지 않는다. 간선의 개수는 N\*(N - 1) / 2



### 1-2. 방향 그래프(directed graph)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FO5SbR%2FbtsC89RZcq7%2F9hZ2HkXJ3yJBTE2RAw53l1%2Fimg.png)



간선에 방향이 있는 그래프 구조를 말한다. 간선의 개수는 N\*(N - 1)



## 1-3. 가중 그래프(Weighted graph)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbCI06E%2FbtsC4JNzHml%2FPIBQuOzkaXfv3WxRE30JQK%2Fimg.png)



각각의 간선이 어떠한 값(value)를 갖는다. 



## 2. 그래프 구현 방법


### 2-1. Adjacency Matrix

 배열을 활용하여 1차원 배열과 2차원 배열을 하나씩 만든다. 1차원 배열은 노드를 저장하는 데 사용하고, 2차원 배열은 간선을 저장하는 데 사용한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcuXwhS%2FbtsC6pnKIIA%2Fmgpekb2ktgkaajG5ZQ9jD1%2Fimg.png)



- 장점

 두 노드간 연결성을 빠르게 확인할 수 있다. 

 간선이 많은 밀집도가 높은 그래프에 활용하기 좋다. (ex. complete graph)



- 단점

 비어있는 간선에도 메모리를 할당해야 하므로 낭비되는 메모리가 많다. 



### 2-2. Adjacency List

linked list 기반으로 구현한다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FPVdoe%2FbtsC1E7sjAV%2FnSOLMCkmwIEr2fOuSPKJv1%2Fimg.png)



첫번째 값은 노드를 나타내고, 이에 이어지는 Linked list에는 간선과 가중치(weighted value)가 들어간다. 



- 장점



 간선이 얼마 없는 희소 그래프에 좋다.

 메모리 필요할 때 필요한 만큼만 할당할 수 있어 메모리 사용량이 많지 않다.

 특정 꼭짓점에 인접한 노드를 찾기 쉽다. 



## 3. 트리(Tree)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FHzYvx%2FbtsC0Xe105p%2FCw3xgihk2bir5XV8dES3Z0%2Fimg.png)



그래프의 일종으로, 노드들이 나무처럼 연결된 비선형 계층적 자료구조이다. 

트리 내에 하위 트리가 존재하고, 그 하위 트리 안에는 또다른 하위 트리가 존재하는 재귀적 자료구조이기도 하다. 



### 3-1. 트리 관련 용어

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F8WMBO%2FbtsC1HQD7KD%2Fhj6rTcSgAPVNYzHnSybEQK%2Fimg.png)



- Root: 맨 상위의 노드, beginning node라고도 한다. 트리는 하나의 루트 노드만 가진다.

- Parent Node: 부모 노드. 한 노드의 바로 상단에 연결되어 있는 노드

- Child Node: 자식 노드. 한 노드의 바로 하단에 연결되어 있는 노드

- Leaf Node: 자식 노드를 갖지 않는 노드

- Sub-tree: 하위 트리. 트리 내부에서 트리 구조를 갖는 또 다른 트리를 말한다.

- Depth: 루트에서 어떤 노드에 도달하기 위해 거쳐야 하는 간선의 수



### 3-2. 이진 트리(Binary Tree)

최대 2개의 자녀 노드만을 가지는 트리. 루트 노드로부터 특정 노드까지의 unique한 경로가 존재한다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpGwwc%2FbtsC4KsdKu2%2FFeBSXL9WSMg6CuW1c7k0dK%2Fimg.png)

이진 트리 예시



### 3-3. 이진 트리 종류



**(1) 정 이진 트리(Full Binary Tree)**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqUfcO%2FbtsC4ES5deq%2FYgIrFnZYOW5McCEkSuWGc0%2Fimg.png)



모든 노드가 0 또는 2개의 자녀 노드를 가진다. 



**(2) 완전 이진 트리(Complete Binary Tree)**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb9wXOQ%2FbtsC36h4IVc%2F8zoxcKgN8rdZ8uqmKzW2h0%2Fimg.png)



높이가 h일 때, h -1 까지는 정 이진 트리이고, 레벨 h는 왼쪽부터 노드가 채워진 트리이다. 

노드의 개수는 2^(h -1) -1 <= n <= 2^h - 1



**(3) 포화 이진 트리(Perfect Binary Tree)**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbbvSjl%2FbtsC7tpNSQK%2FRiBd2hjKGydY9Mmwx01LEK%2Fimg.png)

모든 leaf node가 같은 높이에 있고 leaf node가 아닌 노드는 모두 2개의 자녀 노드를 갖는 트리이다.



### 3-4. 트리 순회

트리에 존재하는 노드들을 방문하는 방법에는 3가지가 있다. 정말 많이 사용하는 개념으로 중요하다!!

**(1) 중위 순회: InOrder Traversal**

왼쪽 하위 트리 -> root -> 오른쪽 하위 트리 순서로 순회한다. 

알파벳 순서대로 print할 때 주로 사용한다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlASYD%2FbtsC9bClKFh%2FRVsHkMHVqF1HESuWrBbwb0%2Fimg.png)

```cpp

void InOrder(TreeNode* tree, QueType& inQue){ 
  if (tree != NULL) {
    InOrder(tree->left, inQue);
    inQue.Enqueue(tree);
    InOrder(tree->right, inQue); 
  } 
}
```



**(2) 후위 순회: PostOrder Traversal**

왼쪽 하위 트리 -> 오른쪽 하위 트리 -> root

노드들을 delete할 때 사용하기 좋다. 



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FnE3QS%2FbtsC7sYJgTA%2FqKLDK8zrWn51OLVNn2OK90%2Fimg.png)



```C++
void PostOrder(TreeNode* tree, QueType& postQue) { 
  if (tree != NULL) { 
    PostOrder(tree->left, postQue); 
    PostOrder(tree->right, postQue);
    postQue.Enqueue(tree);
  }
}
```



**(3) 전위 순회: PreOrder Traversal**

root -> 왼쪽 하위 트리 -> 오른쪽 하위 트리

tree copy할 때 사용하기 좋다. 

DFS(Depth-first Search) 의 원리 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbgFZlt%2FbtsC6T3hWK7%2Fjt7P4K6dxpDQALtKTd9Ick%2Fimg.png)



```cpp
void PreOrder(TreeNode* tree, QueType& preQue) { 
  if (tree != NULL) { 
    preQue.Enqueue(tree); 
    PreOrder(tree->left, preQue); 
    PreOrder(tree->right, preQue); 
  } 
}
```