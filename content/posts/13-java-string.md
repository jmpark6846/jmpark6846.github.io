---
title: "Java String"
date: 2023-07-27T10:57:34+09:00
draft: false
summary: " "
tags: ["java", "String"]

---

# String(Java)

### equals()
* **내용물**을 비교하여 같은지 검사
* `==` 와 다르다. `==` 는
  * 기본 자료형일 경우 변수의 내용물이 같은지 검사
  * 객체 자료형일 경우 레퍼런스가 같은지 검사
* **문자열은 객체이기 때문에 내용물을 비교하기 위해선 `equals()`를 사용해야 한다.**


* `equals()`와 `==`를 사용해보면 결과값이 예상과 다르게 나오는 경우가 있다.
```java
String str1 = "AAA";
String str2 = "AAA";

System.out.println(str1 == str2);
System.out.println(str1.equals(str2));

=>
true
true
```
* 각자 다른 문자열 변수이기 때문에 `==`로 비교할때 `false`가 나와야 할 것 같지만 `true`이다.
* 같은 내용의 문자열을 다른 변수에 여러번 사용하는 경우 문자열이 메모리 내에서 한번만 생성되고 변수들이 같은 객체를 가리키기 때문.
* 만약 `new`를 사용하여 새로운 객체로 만들어주면 예상대로 결과가 나온다.
```java
str1 = new String("AAA");
str2 = new String("AAA");

System.out.println(str1 == str2);
System.out.println(str1.equals(str2));

=> 
false
true
```


### StringBuilder
* String으로 문자열을 조작하는 것은 느리다. String은 객체이며 불변이기 때문이다. 즉 문자열 연산을 할때마다 새로운 객체를 생성하고, 이전 객체는 가비지 컬렉터가 해제한다. 이런 방식은 비효율적이다.
* 문자열 연산에는 StringBuilder를 사용하며 String 연산에 비해 매우 빠르다. **따라서 엔터프라이즈급에서는 가급적 String 대신 StringBuilder를 사용한다.**

```java
String str1 = "AAA";
String str2 = "BBB";

// + 연산 10만번 연산 속도 계산
long start = System.currentTimeMillis();

for(int i = 0; i < 100000; i++){
    str1 += str2;
}

long end = System.currentTimeMillis();
System.out.println(end - start);


// StringBuilder append() 10만번 연산 속도 계산
StringBuilder sb = new StringBuilder(str1);

start = System.currentTimeMillis();
for(int i = 0; i < 100000; i++){
    sb.append(str2);
}
end = System.currentTimeMillis();
System.out.println(end - start);

=>
4941
4
```

* 내가 가진 2014년형 맥북으로는 `+`연산은 5초, StringBuilder `append()`는 4ms 정도로 상당히 차이가 많이 났다.