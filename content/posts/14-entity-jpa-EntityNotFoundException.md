---
title: "[JPA]엔티티 조회 시 존재하지 않을때 예외사항 처리"
date: 2023-07-30T09:59:07+09:00
draft: false
summary: "existsById()로 EntityNotFoundException 처리하기"
tags: ["java", "jpa"]

---


### 엔티티 조회 시 없을때 예외사항 처리
* `getReferenceById()` 는 엔티티가 없을때 `EntityNotFoundException을` 발생시킨다.
* 주의할점은 레코드가 존재하지 않는다는 것을 발견한 즉시 발생시키는 것이 아니라, 그 이후 객체에 접근했을때 발생시킨다는 것이다. 따라서 원하는 시점에 exception이 발생되지 않아 처리되지 못할 수 있다.
* 이럴땐 `existsById()`로 먼저 검사 후 `getReferenceById()` 결과값을 리턴해주자.


