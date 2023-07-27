---
title: "StackView 안에 있는 TableView가 제대로 표시되지 않는 문제"
date: 2023-02-14T23:50:12+09:00
draft: false
summary: " "
tags: ["swift", "ios"]
---

## 문제 
스택뷰 안에 테이블뷰가 있고, 테이블뷰가 인터넷에서 fetch해온 데이터로 다이나믹하게 세팅될 경우 셀이 나타나지 않는다.


## 원인
* 스택뷰는 서브뷰의 크기를 압축해서 표시한다. 데이터가 없는 테이블은 압축되어 높이 0으로 로드된다.
* 테이블뷰는 높이가 지정되지 않으면 셀을 표시할 필요가 없기 때문에 아무것도 표시하지 않는다. 따라서 셀이 전혀 만들어지지 않고, 셀을 요청하는 `cellForRowAt` 메소드도 호출되지 않는다.
* 동적으로 정해지는 컨텐츠의 크기를 어떻게 테이블의 크기로 지정해줄 것인가?


## 해결방법
* 테이블뷰의 height constraint를 동적인 크기로 직접 지정한다.
* 동적인 크기는 다음과 같이 구한다. 
    * 테이블의 height constraint를 매우 큰 값으로 설정해서 우선 스크린에 로드 한 다음, 보이는 셀들의 크기를 더한 뒤 크기의 합을 height constraint의 constant로 지정하여 테이블의 크기를 조정해준다.


