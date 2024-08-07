---
layout: post
toc: true
title: "[iOS] Pull Down Button에 대해 알아보자"
categories: iOS
tags: [TIL, iOS]
author:
  - bowon han
---

## 궁금증 🙋🏻‍♀️
***
진행하고 있는 프로젝트에서 사용자의 옷에 대한 정보를 받아와서 보여주고,
해당 리스트에 대한 삭제와 수정을 할 수 있게 UI를 구성해야했다. 

![](/images/ios-pulldownbutton-1.png)

(제가 디자인해서 좀 .. 뭐가 없습니다 .. 😭)

<br>
어떻게 할지 고민하다가 맨 오른쪽에 버튼을 두고 해당 **버튼을 눌렀을 때, 메뉴바가 나타나도록** 구현해야겠다고 생각을 하였다. 
<br>

![](/images/ios-pulldownbutton-2.png) 

대충 이런 느낌으로?

<br>
그래서 찾아본 결과 !

pull down button 이라는 것을 발견했다.

[이글](https://sujinnaljin.medium.com/ios-pull-down-button-과-pop-up-button-f0f85d650b51)에서 보면 **pop up button**과 **pull down button**을 아주 잘 설명해주고 있다!<br>
(자세한 부분은 해당 글에서 보시길 바랍니닷!)

<br>
내가 구현하고자 하는 menu를 만들기 위해 **pull down button**을 사용하기로 하였다. <br>
현재 필요한 menu 요소는 두가지로, **수정, 삭제**를 넣을 것이다.

<br>
pull down button은 button의 프로퍼티 중 
<br>

**`menu != nil && changesSelectionAsPrimaryAction = false && showsMenuAsPrimaryAction = true`**
 
조건을 만족하는 구성이다. 

![](/images/ios-pulldownbutton-3.png)

button에 menu를 넣기 위해서는 UIMenu를 추가해주어야 한다!


## 구현 방법 🤷🏻‍♀️
***

이번엔 구현 방법에 대해 알아볼 것이다.

일단 menu에 요소추가를 위해 UIAction 배열을 만들어 menu에 넣어줄 요소를 구성한다.

```swift
    private lazy var items: [UIAction] = {
        return [UIAction(title: "삭제",
                         image: UIImage(systemName: "trash"),
                         handler: { _ in print("삭제버튼")}),

                UIAction(title: "수정",
                         image: UIImage(systemName: "pencil"),
                         handler: { _ in print("수정버튼")})
                ]
    }()
```

<br>
이후 UIMenuElement를 받는 **childern 프로퍼티에 items를 넣어준다.**

```swift
    private lazy var menu: UIMenu = { 
        return UIMenu(
            title: "", 
            options: [], 
            children: items
        ) 
    }()
```


<br>
이것을 button.menu에 넣어주면 **`menu != nil`** 조건을 만족할 수 있고,<br>
짧게 눌렀을 때에도 menu창을 볼 수 있도록 **showsMenuAsPrimaryAction**도 true로 지정하면 모든 조건을 만족하게 된다. 

```swift
    private lazy var imageSettingButton: UIButton = {
        let button = UIButton()
        button.menu = menu
        button.showsMenuAsPrimaryAction = true
        
        return button
    }()
```

<br>
이렇게 내가 원하는 menu 버튼을 구현하였다! 

![](/images/ios-pulldownbutton-4.png)


<br>

## 참고 📜
***

[https://sujinnaljin.medium.com/ios-pull-down-button-과-pop-up-button-f0f85d650b51](https://sujinnaljin.medium.com/ios-pull-down-button-과-pop-up-button-f0f85d650b51)

