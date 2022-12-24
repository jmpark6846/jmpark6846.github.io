---
title: "Binary Search Tree"
date: 2022-12-24T19:37:28+09:00
draft: true
categories: ["자료구조"]
summary: "이진 탐색 트리 구현"
---
* 이진 트리로서 각 노드가 다음과 같은 조건을 만족한다.   
  -> 각 노드의 키 값이 왼쪽 서브트리에 있는 노드들의 키값들 보다 크고, 오른쪽 서브트리에 있는 노드들의 키값들보다 작다.  

* 이진 탐색을 트리로 구현한 자료구조이다.
* 균등한 트리일 경우 O(log(n))이 소요되지만, 편향된 트리일 경우가 최악의 케이스로 O(n)이 소요된다.
* 균등하게 트리를 유지하기 위해 AVL트리, 레드블랙트리, B-트리가 있다.  
   
### Node
* 트리의 노드

```java
public class Node{
    private int key;
    private Object value;

    private Node left;
    private Node right;

    public Node(int key, Object value){
        this.key = key;
        this.value = value;
    }

    // Getter, Setter 생략
}
```

### Binary Search Tree
* 구조의 특징은 트리 전체와 부분이 모두 트리 형태이다.
* 따라서 탐색, 삽입, 삭제, 순회 연산에서 탐색 과정을 재귀함수를 활용하여 구현할 수 있다.  


* `root`: 트리의 루트 노드
* `get`: 노드 탐색
* `put`: 노드 삽입
* `putItem`: 실제 노드 삽입 연산을 수행한다.
* `delete`: 노드 삭제
* `deleteItem`: 실제 노드 삭제 연산을 수행한다.
* `getMin`: 해당 노드를 가진 트리 중 최소 키를 가진 노드 구한다.
* `delMinimum`: 해당 노드를 루트 노드로 하는 트리에서 최소 값을 가진 노드를 제거한다.
* `traverse`: 중위순회하여 최소값부터 최대값까지 출력한다.


```java

public class BinarySearchTree {
    private Node root;

    public Object get(int key){
        Node current = root;

        while(current != null){
            if(current.getKey() == key){
                return current.getValue();
            }

            if(current.getKey() > key) {
                current = current.getLeft();
            }else{
                current = current.getRight();
            }
        }
        return null;
    }

    public void put(int key, Object value){
        root = putItem(root, key, value);
    }

    public Node putItem(Node node, int key, Object value){
        if(node == null){
            return new Node(key, value);
        }
        if(key < node.getKey()){
            node.setLeft(putItem(node.getLeft(), key, value));
        }else if(key > node.getKey()){
            node.setRight(putItem(node.getRight(), key, value));
        }else{
            node.setValue(value);
        }
        return node;
    }

    public void delete(int key){
        root = deleteItem(root, key);
    }

    public Node deleteItem(Node node, int key){
        // 삭제하고자 하는 키의 노드가 없음
        if(node == null){
            return null;
        }

        if(key < node.getKey()){
            node.setLeft(deleteItem(node.getLeft(), key));
        }else if(key > node.getKey()){
            node.setRight(deleteItem(node.getRight(), key));
        }else{
            if(node.getRight() == null){
                return node.getLeft();
            }

            if(node.getLeft() == null) {
                return node.getRight();
            }

            Node target = node;

            node = getMin(target.getRight());
            node.setRight(delMinimum(target.getRight()));
            node.setLeft(target.getLeft());
        }
        return node;
    }
    public Node getMin(Node node){
        if(node.getLeft() == null){
            return node;
        }
        return getMin(node.getLeft());
    }

    public Node delMinimum(Node node){
        if(node.getLeft() == null){
            return node.getRight();
        }
        node.setLeft(delMinimum(node.getLeft()));
        return node;
    }

    public void traverse(Node node){
        if(node == null){
            return;
        }
        traverse(node.getLeft());
        System.out.printf("%d ",node.getKey());
        traverse(node.getRight());
    }

    public Node getRoot() {
        return root;
    }

    public void setRoot(Node root) {
        this.root = root;
    }
}

```
