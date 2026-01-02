---
layout: post
toc: true
title: "[iOS] SwiftUI의 StateObject와 ObservedObject"
categories: iOS 
tags: [iOS, SwiftUI]
author:
  - bowon han
---



**(1) ObservedObject란?**
- 공식 문서 설명


> A property wrapper type that subscribes to an observable object and invalidates a view whenever the observable object changes.

번역하면 
- observable Object를 구독하고 해당 객체가 변경될때마다 초기화하는 property wrapper이다. 

**예시** 
```swift
final class SubViewModel: ObservableObject {
  @Published var isOn = false
  
  func toggle() {
    isOn.toggle()
  }
}
```
- 뷰에서 보여질 이벤트를 담은 뷰모델을 관찰 가능한 객체로 만든다.
- `@Published` property wrapper로 선언된 변수가 변경될 시 외부로 변경사항을 알릴 수 있다. 
	- 해당 변수 상태가 변경되면 외부(**뷰**)에도 해당 내용이 바로 알려짐


```swift
struct SubView: View {
    @ObservedObject var viewModel = SubViewModel()  

    var body: some View {
        VStack {
            Text("- 자식 뷰 -")
                .font(.system(size: 30, weight: .bold))
            Button(action: {
                viewModel.toggle()
            }, label: {
                HStack {
                    Image(systemName: viewModel.isOn ? "lightbulb.fill" : "lightbulb")
                        .resizable()
                        .scaledToFit()
                        .frame(width: 40)
                        .foregroundStyle(.yellow)

                    Text(viewModel.isOn ? "켜졌습니다!" : "꺼졌습니다.")
                        .font(.system(size: 20, weight: .medium))
                        .foregroundStyle(.black)
                }
            })
        }
    }
}
```
- 위 `SubViewModel`의 인스턴스를 `@ObservedObject`로 참조하였다. 
	- 따라서 `viewModel.isOn` 의 상태에 따라 뷰가 바뀐다. 


![](/images/swiftui-stateobject-observedobejct1.gif)


**그러나**


> 부모 뷰 내부에 `@ObservedObject`가 포함된 뷰가 있다면, 부모뷰의 상태가 변경될때 마다 해당 객체가 초기화되는 문제가 발생한다. 


---

**(2) ObservedObject의 문제점**
- 위에서 언급한 "부모뷰 내부에 `@ObservedObject`가 포함된 뷰" 의 상황을 예시를 들어보자!


```swift
struct ParentView: View {
    @State var isOn = false

    var body: some View {
        VStack {
            Text("- 부모 뷰 -")
                .font(.system(size: 30, weight: .bold))

            Button(action: {
                isOn.toggle()
            }, label: {
                HStack {
                    Image(systemName: isOn ? "lightbulb.fill" : "lightbulb")
                        .resizable()
                        .scaledToFit()
                        .frame(width: 40)
                        .foregroundStyle(.yellow)

                    Text(isOn ? "켜졌습니다!" : "꺼졌습니다.")
                        .font(.system(size: 20, weight: .medium))
                        .foregroundStyle(.black)
                }
            })
            Divider()
            SubView()
        }
    }
}
```

- 첫 예시에서 보여준 `SubView`가  `ParentView` 에 포함되어있다. 
	- `SubView`에는 `@ObservedObject var viewModel = SubViewModel()`가 포함되어, 
	- 현재 부모뷰(`ParentView`) 내부에  `@ObservedObject`가 포함된 뷰(`SubView`)가 들어있다. 

해당 앱을 실행하면 

![](/images/swiftui-stateobject-observedobejct2.gif)



- 보이는것 처럼 `SubView`의 전구가 켜져있을때 `ParentView`의 전구 상태가 바뀌면 `SubView`의 전구도 함께 꺼진다. 
- 왜일까?
	- 위에서 말했던 `부모뷰의 상태가 변경될때 마다 해당 객체가 초기화되는 문제`가 발생하는 것이다. 
	- `ParentView`의 전구 상태 변경 -> 부모뷰의 상태 변경 
	- 해당 객체(`@ObservedObject`로 선언된 객체)가 초기화 -> `SubView`의 viewModel 객체가 초기화된다 -> `SubViewModel`의 isOn 변수가 false로 초기화 
	- 이와 같은 과정으로 위와 같은 결과가 나타난다. 


---

**(3) @StateObject 사용**
- 이러한 문제를 `@StateObject`를 사용하여 해결할 수 있다. 
- `@StateObject`는 첫 호출 시에만 초기화 되기 때문에 뷰를 다시 그릴때마다 초기화하는 `@ObservedObject`의 문제를 해결할 수 있다!

`@ObservedObject`를 `@StateObject`로 변경하여 실행하면 

![](/images/swiftui-stateobject-observedobejct3.gif)

문제없이 동작하는 것을 확인할 수 있다. 