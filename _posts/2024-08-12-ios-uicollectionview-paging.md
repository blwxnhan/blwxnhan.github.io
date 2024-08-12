---
layout: post
toc: true
title: "[iOS] UICollectionView Pagination(페이징)"
categories: iOS 
tags: [coordinator, iOS]
author:
  - bowon han
---

현재 SNS 기능이 포함된 프로젝트를 진행중입니다 😊


현재 많이들 사용하는 SNS는 엄청나게 많은 내용을 담고 있고, 정보들을 끊임없이 보여주죠?.. 
<br>
❌~~그래서헤어나오지못함~~❌
<br>

이렇게 많은 정보들을 한번에 모두 받아올 수는 없겠죠. <br>
그래서 화면에 보이는 정보들을 조금씩 끊어서 가져와야합니다!
<br>

## 아이디어 💡
***
아래롤 스크롤을 할 시에 컨텐츠의 마지막에 다다르면, 다음 페이지 정보들을 받아오도록 구현해보려고 합니다! 
<br>

**UICollectionViewDelegate**는 **UIScrollViewDelegate**를 채택하기 때문에 ```scrollViewDidScroll```를 사용할 수 있습니다 👍🏻


## 구현 ⭐️
***

```swift 
extension CommunityViewController: UICollectionViewDataSource, UICollectionViewDelegate {
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        /// 1️⃣
        if scrollView.contentOffset.y > scrollView.contentSize.height - scrollView.bounds.height { 
            
        /// 2️⃣
            if isInfiniteScroll && hasNextPage {  
                isInfiniteScroll = false
                
                fetchSnaps(pageNum: pageNum) {
                    self.isInfiniteScroll = true
                }
            }
        }
    }
}
```

### 1️⃣ 서버 요청 시점
![](/images/ios-uicollectionview-paging-1.png)

일단 ```scrollView.contentOffset.y```는 scrollview의 원점에서 contentview(총 컨텐츠)의 원점까지 얼마나 떨어졌는냐를 나타냅니다. 
사용자가 스크롤 한 후의 지점입니다!.

```scrollView.contentSize.height - scrollView.bounds.height``` <br>
는 scrollView가 길어질 수 있는 총 높이, 즉 총 컨텐츠의 길이에서, scrollview의 높이를 뺀 값입니다. <br>

이것이 사용자가 스크롤 한 후의 지점보다 작다면?


🍀 더 이상 스크롤 할 부분이 없다는 뜻! 즉, **컨텐츠를 모두 보았다는 뜻이죠** 🍀
<br>이때 다음 단계를 진행하도록 하였습니다!


### 2️⃣ 서버 요청 조건
다음은 ```if isInfiniteScroll && hasNextPage``` 부분입니다. 

일단, hasNextPage 변수에는 서버에서 전달받은 값으로, 다음 페이지가 존재하는지에 대한 bool값입니다!
<br>

그리고 isInfiniteScroll 변수는 이미 content view의 바닥에 닿아 서버에 요청을 보냈음을 의미하는 변수입니다. <br>

해당 변수가 없다면, 여러번 서버에게 데이터를 요청하게 되므로, 요청을 보낸 후 false로 변경하도록 하였습니다. 

```swift
fetchSnaps(pageNum: pageNum) {
    self.isInfiniteScroll = true
}
```
서버에 요청하는 부분에는 escaping closure를 통해 네트워크 요청이 끝났을 시 다시 isInfiniteScroll 변수 값을true로 변경하도록 하였습니다‼️

## 참고 📜
***
- [https://devyul.tistory.com/109](https://devyul.tistory.com/109)
- [https://gyuios.tistory.com/139](https://gyuios.tistory.com/139)

