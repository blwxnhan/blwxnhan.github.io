---
layout: post
toc: true
title: "[iOS] collectionView cell 선택하게 할 수 있는 방법..!"
categories: iOS
tags: [TIL, iOS]
author:
  - bowon han
---

오늘은 **UICollectionViewListCell의 accessory** 기능에 대해 알아볼 것입니다. 

## 구현 계기 🤷🏻‍♀️
***
진행하는 프로젝트에서 쭉 나열되어있는 list 중 **몇가지를 선택**하는 것을 구현해야했습니다. 

![](/images/ios-uicollecionviewlistcell-1.png)

요런 느낌으로 진행하고 싶었습니다. 

일단 List자체는 **collectionView로 구현**을 하였습니다.

다른 어플에서 **버튼을 눌렀을 때 저 선택버튼이 나타나도록**하는 부분이 있었던 것 같아, 방법을 찾아보기로 하였습니다. 

버튼이 나올때 갑자기 딱 나오는게 아니라, 미끄러지면서..? 나왔던 것같은데... 처음엔 방법을 찾지 못해, 직접 애니메이션을 줘야하나..? 생각을 하던 중! 

[이 글](https://sujinnaljin.medium.com/ios-uicellaccessory-종류-알아보기-335d3f4a1f3c)을 발견하게 되었습니다!

## 구현 방법 😊
***
일단 해당 글에서는 UICollectionViewListCell의 ```UICellAccessory``` 에 대해서만 자세하게 다루고 있습니다. 

제가 필요했던 accessory는 

![](/images/ios-uicollectionviewlistcell-2.png)

바로 이것입니다!

해당 accessory는 displayed의 기본값이 ```.whenEditing```으로 따로 파라미터 없이 생성할 시 ```collectionView.isEditing = true``` 편집모드를 true로 바꿔주어야 보이게 됩니다. 

제가 딱 원하던... 것을 발견한 것입니다!!

따라서 저는 
```swift
let cellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, ListModel> {
    (cell, indexPath, list) in
    var content = cell.defaultContentConfiguration()

    content.image = UIImage(systemName: "star")
    content.text = "Favorites"

    cell.contentConfiguration = content
    cell.accessories = [.multiselect()]
}

self.dataSource = UICollectionViewDiffableDataSource<Section, ListModel>(collectionView: self.clothListCollectionView) {
    (collectionView, indexPath, list) -> UICollectionViewListCell? in
    return self.clothListCollectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: list)
}
```
평소 셀을 등록할 때 사용하던 ```register(_, forCellWithReuseIdentifier:)```가 아닌 ```CellRegistration```을 사용하였습니다. 

collectionView diffableDataSource로 구현하였고, ```.multiselect()``` 하나의 accessory만을 사용하였습니다. 

**여러 개의 accessory를 동시에 사용**해도 된다고 하네요!

![](/images/ios-uicollectionviewlistcell-3.png) 

예시로 구현해보았습니다.

제가 원하던대로 버튼을 클릭하면, 미끄러지듯이! 원래의 요소들이 살짝 슬라이드되면서 select 버튼이 나타나네요 👍🏻


아직 공부 중에 있어 .. **UICollectionViewListCell**에 대해서는 차차 업데이트 하도록 하겠습니다!

## 참고 📄
***
[https://sujinnaljin.medium.com/ios-uicellaccessory-종류-알아보기-335d3f4a1f3c](https://sujinnaljin.medium.com/ios-uicellaccessory-종류-알아보기-335d3f4a1f3c)