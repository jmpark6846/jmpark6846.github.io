---
title: "Stack"
date: 2022-12-24T16:44:00+09:00
tags: ["자료구조"]
summary: "LinkedList로 Stack 구현하기"
isCJKLanguage: true
---
* LIFO(Last In First Out): 마지막에 입력한 데이터가 먼저 출력된다.
* 트리 순회, 그래프 탐색, 함수 콜 스택 등에 사용된다.
* LinkedList를 사용해 구현했다.

### Node
* `Stack`을 구성하는 `Node` 클래스

```java
package dataStructure.Stack;

public class Node {
private Object data;
private Node next;
public Node(){
}

    public Node(Object data){
        this.data = data;
        this.next = null;
    }


    public Object getData(){
        return this.data;
    }

    public Node getNext(){
        return this.next;
    }

    public void setData(Object data) {
        this.data = data;
    }

    public void setNext(Node next) {
        this.next = next;
    }
}
```

### Stack

* `first`: 스택의 첫 노드
* `top`: 스택의 맨 위 노드
* `push`: 데이터 추가
* `pop`: 데이터 삭제

```java
package dataStructure.Stack;

public class Stack {
    private Node first;
    private Node top;

    public Stack(){}

    public Node getFirst() {
        return first;
    }

    public void setFirst(Node first) {
        this.first = first;
    }

    public Node getTop() {
        return top;
    }

    public void setTop(Node top) {
        this.top = top;
    }

    public void push(Object object){
        Node newNode = new Node(object);

        if(first == null){
            first = newNode;
            top = first;
        }else{
            top.setNext(newNode);
            top = newNode;
        }
    }

    public Object pop(){
        Object result;

        if(isEmpty()){
            return null;
        }

        result = top.getData();
        if(first == top){
            first.setNext(null);
            first = null;
            top = null;
            return result;
        }
        Node topPrev = getPrev(top);
        topPrev.setNext(null);
        top = topPrev;

        return result;
    }

    public Node getPrev(Node target){
        Node current = first;
        while(current.getNext() != target){
            current = current.getNext();
        }

        return current;
    }

    public boolean isEmpty(){
        return first == null;
    }

    public void print(){
        StringBuilder sb = new StringBuilder();

        Node current = first;

        if(isEmpty()){
            System.out.println("empty\n");
            return;
        }

        while(current != null){
            sb.append(current.getData().toString() + " ");
            current = current.getNext();
        }
        System.out.println(sb + "\n");
    }
}

```

### 예시
```java
Stack stack = new Stack();

stack.push(1);
stack.push(2);
stack.push(3);
stack.push(4);
stack.push(5);

stack.print();


System.out.println("popped: " + stack.pop());
stack.print();

System.out.println("popped: " + stack.pop());
stack.print();

System.out.println("popped: " + stack.pop());
stack.print();

System.out.println("popped: " + stack.pop());
stack.print();

System.out.println("popped: " + stack.pop());
stack.print();

System.out.println("popped: " + stack.pop());
stack.print();
```

실행 결과
```java
1 2 3 4 5 

popped: 5
1 2 3 4 

popped: 4
1 2 3 

popped: 3
1 2 

popped: 2
1 

popped: 1
empty

popped: null
empty

```