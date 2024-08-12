---
layout: post
toc: true
title: "[iOS] swift cornerRadius와 shadow 동시에 주기"
categories: iOS
tags: [TIL, iOS]
author:
  - bowon han
---

현재 진행 중인 프로젝트에서 **그림자와 둥근 테두리를 동시에 가진 view**를 구현해야했다. 

하지만 **두가지를 동시에** 하려고 하면 적용되지 않는 것을 볼 수 있다 ‼️

## 왜 안되는가? 🤷🏻‍♀️
***

cornerRadius를 주기 위해서는 다음과 같은 코드가 필요하다.

```swift

    self.layer.cornerRadius = 15
    self.layer.masksToBounds = true

    // or

    self.layer.cornerRadius = 15
    self.layer.clipsToBound = true

```

view의 layer 부분에 cornerRadius를 주고, 
**masksToBounds**또는 **clipsToBound**를 통해 바깥 영역을 잘라내는 원리이다. 

여기서 문제가 생긴다. 

**masksToBounds**또는 **clipsToBound**를 통해 바깥 영역을 잘라내기 때문에,
해당 view에 그림자를 주게 되면 **그림자는 그 바깥 부분**이기 때문에 잘려나가게 된다.

## 해결 👍🏻
***

view를 두개를 사용하면 된다‼️

현재 나의 프로젝트에서는 customView 내부에 또 다른 innerCustomView가 들어가는 구조이다.

![](/images/ios-cornerRadius-and-shadow-1.png)

이런식으로 view가 구성된다. 
<br>

따라서 customView(**외부 View**)에 **shadow**를 주고,

innerCustomView를 stackView(**내부 View**)로 감싸 **stackView에 cornerRadius**를 주면 된다. 

```swift

    self.layer.shadowColor = UIColor(hexCode: "c4c4c4").cgColor // 그림자 색
    self.layer.shadowOpacity = 0.7 
    self.layer.shadowRadius = 15 
    self.layer.shadowOffset = CGSize(width: 1.0, height: 1.0)
    
    self.stackView.layer.cornerRadius = 15
    self.stackView.layer.masksToBounds = true
    self.stackView.backgroundColor = .white

    // stackView margin 주기
    self.stackView.layoutMargins = UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10) 
    self.stackView.isLayoutMarginsRelativeArrangement = true

```

그런데 아직 해결하지 못한 것은 ,, 

```swift

    self.layer.shadowPath = UIBezierPath(rect: CGRect(x: self.bounds.origin.x - 1.5, 
                                                    y: self.bounds.origin.y + 10, 
                                                    width: self.bounds.width + 3, 
                                                    height: self.bounds.height - 7)).cgPath

```
**그림자가 나올 영역을 지정**해주는 **shadowPath**를 집어넣으면 그림자가 보이지 않는다.. 

진행중인 프로젝트 다른 곳에서도 **collectionViewCell에 cornerRadius+shadow를 함께** 주면서 **shadowPath를 지정**해주고 있는데 

거기선 잘 작동하는것이 왜 여기서는....

암튼,, shadowPath에 대해서도 더 찾아보고 알아내면 업데이트 해야겠다!!!!


## 참고 📜
***
- [https://woongsios.tistory.com/54](https://woongsios.tistory.com/54)

