---
layout: post
toc: true
title: "[iOS] swift 연산 프로퍼티(get,set)"
categories: iOS
tags: [TIL, iOS]
author:
  - bowon han
---

## 연산 프로퍼티란?🤷🏻‍♀️
***

> 저장 프로퍼티와 달리 **직접 저장공간을 갖지 않고**, **다른 저장 프로퍼티**의 값을 읽어 연산을 실행하거나, 프로퍼티로 전달받은 값을 다른 프로퍼티에 저장한다. 

**즉, 다른 저장 프로퍼티가 꼭 존재해야한다.**

직접 어떤 값을 저장하는 것이 아니므로, **선언 시 반드시! 자료형을 명시**해야한다.

```swift

class Cloth {
    // 반드시 다른 저장 프로퍼티가 먼저 존재하고 이것을 사용해야함.
    var clothName: String = "pretty shirt"

    // 연산 프로퍼티
    var name: String {
        get{
            return clothName
        }
        set(clothName) {
            self.clothName = clothName + "은(는) 옷장에 있습니다."
        }
    }
}
```

## 연산 프로퍼티 사용4️⃣
***

저장 프로퍼티 사용할때처럼 사용하면 된다. 

```swift
let shirt: Cloth = .init()

// get 접근
print(shirt.name) // pretty shirt

// set 접근
shirt.name = "long shirt"
print(shirt.clothName) // long shirt은(는) 옷장에 있습니다.

```
### set😊

set의 파라미터는 하나만 존재하고, 생략이 가능하다. 
생략하면 파라미터에 **newValue** 로 접근이 가능하다. 

연산 프로퍼티에서 set은 생략할 수 있다.

하지만 get은 생략할 수 없다!!


### get😊
연산 프로퍼티 사용시 set없이 **get만 선언**해도 된다. 

```swift
class Cloth {
    // 반드시 다른 저장 프로퍼티가 먼저 존재하고 이것을 사용해야함.
    var clothName: String = "pretty shirt"

    // 연산 프로퍼티
    var name: String {
        get{
            return clothName
        }
    }
}
```

이렇게 사용하고 축약하면 get구문을 없애서 사용할 수 있다. 

```swift
class Cloth {
    // 반드시 다른 저장 프로퍼티가 먼저 존재하고 이것을 사용해야함.
    var clothName: String = "pretty shirt"

    // 연산 프로퍼티
    var name: String {
        return clothName
    }
}
```

## 참고 📜
***

[https://babbab2.tistory.com/119](https://babbab2.tistory.com/119)