---
layout: post
toc: true
title: "[iOS] coordinator 패턴 알아보기"
categories: iOS 
tags: [coordinator, iOS]
author:
  - bowon han
---

오늘의 주제는 iOS에서의 coordinator pattern입니다!


## 왜 사용하는가🤷🏻‍♀️?
***
먼저 왜 이 패턴이 나오게 되었는지를 생각해보려고 합니다!


이 패턴을 처음으로 소개한 Khanlou는 

>**viewController가 massive해지는 이유는 flow 로직, business 로직이 모두 얽혀있기 때문이다.**


라고 생각하였습니다. 

### 👇🏻 기존의 사용하던 화면 전환 방법  

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
	// 1
    let object = self.dataSource[indexPath]
    // 2
    let detailViewController = DetailViewController.init(object: object)
    //3   
    self.navigationController?.pushViewController(detailViewController, animated: true)
}
```

위 코드는 tableView에서 **하나의 cell을 클릭했을 시 다른 화면으로 전환** 되는 코드입니다. 기존의 사용하던 화면 전환 방법을 볼 수 있습니다.

1. 먼저 **indexPath**에 접근하여 객체 생성합니다.
2. 다음으로 **현재 VC**에서 **detailViewController**를 생성합니다.
3. 마지막으로 현재 VC는 이제, **부모** 즉 **navigationController**에게 해야할 일을 지시합니다.
<br>

#### 이때 발생하는 문제 ‼️
현재 VC는 flow상 **다음 넘어갈 VC가 어떤 것인지 알고 있습니다**.

이는 VC가 **flow 로직까지 담당**한다는 의미이며 **VC가 너무 많은(massive) 일을 하고 있음**을 볼 수 있습니다. 또한, 어플의 크기가 커질 수록 복잡해지며 코드의 가독성 또한 떨어집니다.


따라서 coordinator 패턴이 나오게 된 것은 **VC가 담당하던 flow 로직을 분리**하기 위함임을 알 수 있습니다.


## 사용방법 👀
***
본격적으로 **coordinator 패턴**에 대해 알아보겠습니다.

**‼️coordinator 패턴의 구현 방법은 앱의 구조에 따라 달라질 수 있습니다.**

![](/images/ios-coordinator-1.png)
**👆🏻 위 사진은 제가 하나의 프로젝트를 진행하면서 만들게 된 어플의 구조 입니다.**

해당 어플을 예로 들어 설명하겠습니다.
저희의 프로젝트에서는 **TabBarController**를 사용하였고, 로그인, 회원가입의 인증과정이 필요한 어플이었습니다. 

TabBar 내에는 총 2개의 VC(**MainVC, SettingVC**)가 존재합니다. 

> MainVC에서의 흐름은 다음과 같습니다. 

	1. MainVC 에서 버튼을 클릭 -> RegisterImageVC
	2. RegisterImageVC 에서 버튼을 클릭 -> ImageResultVC
    3.tabBar의 Setting 항목 클릭 시 ->settingVC
<br>

구조를 먼저 보면 일단 **하나의 AppCoordinator**가 필요합니다.

하위 coordinator들이 상위 coordinator에게 **해야할 일을 알리며**, **coordinator가 flow로직을 담당**하게 되는 구조입니다. 

쉽게 설명하기 위해 **TabBarCoordinator와 AuthCoordinator를 상위 coordinator**, 
그 **아래 coordinator들을 하위 coordinator**라고 칭하겠습니다.

### ✔️ coordinator 프로토콜
먼저 **coordinator 프로토콜**을 만들어야합니다. 
**모든 coordinator들은 coordinator 프로토콜을 채택**하여 따르게 됩니다. 
```swift
protocol Coordinator : AnyObject {
    var parentCoordinator: Coordinator? { get set }
    var childCoordinator: [Coordinator] { get set }

    var navigationController : UINavigationController { get set }

    func start()
}

extension Coordinator {
    func childDidFinish(_ coordinator : Coordinator){
        for (index, child) in childCoordinator.enumerated() {
            if child === coordinator {
                childCoordinator.remove(at: index)
                break
            }
        }
    }
}
```

Coordinator 프로토콜에는 **childDidFinish**라는 메서드를 두어 childCoordinator에 존재하는 요소들을 삭제하여 **메모리 leak을 방지**할 수 있도록 합니다. 

이는 AuthCoordinator에서 TabBarCoordinator로 넘어가는 상황에 사용할 수 있습니다. 

### ✔️ AppCoordinator 생성

```swift
final class AppCoordinator : Coordinator {
    var parentCoordinator: Coordinator?
    var childCoordinator: [Coordinator] = []
    
    var navigationController: UINavigationController

    func start() {
        startAuthCoordinator()
    }
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
        navigationController.isNavigationBarHidden = true
    }
    
	// 두가지 coordinator 시작점
    func startAuthCoordinator() {
        let authCoordinator = AuthCoordinator(navigationController: navigationController)
        childCoordinator.removeAll()
        authCoordinator.parentCoordinator = self
        childCoordinator.append(authCoordinator)
        authCoordinator.start()
    }
    
    func startMainTabbarCoordinator() {
        let tabBarCoordinator = TabBarCoordinator(navigationController: navigationController)
        childCoordinator.removeAll()
        tabBarCoordinator.parentCoordinator = self
        tabBarCoordinator.start()
    }
}
```

저희 프로젝트에서는 전체적으로 **앱을 사용하는 부분**과 **인증(로그인,회원가입) 부분**이 있었기 때문에 TabBarCoordinator, AuthCoordinator로 분리하여 구현하였습니다.

따라서 AppCoordinator에서는 두가지 Coordinator를 시작할 수 있는 메서드를 만들어두었습니다. 메서드 내부에서는 

- **자식 coordinator를 모두 삭제**
- 호출하고하자는 coordinator의 부모 coordinator를 자기 자신으로 지정
- coordinator 호출

순서로 진행됩니다. 
<br>

start메서드는 **해당 coordinator의 시작점**이므로, 앱을 켜자마자 인증(로그인, 회원가입) 화면을 나타나게하기 위해 startAuthCoordinator를 호출해주었습니다.

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?


    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        if let windowScene = scene as? UIWindowScene {
            let window = UIWindow(windowScene: windowScene)
            self.window = window

            let navigationController = UINavigationController()
            self.window?.rootViewController = navigationController
            
            let coordinator = AppCoordinator(navigationController: navigationController)
            coordinator.start()
            
            self.window?.makeKeyAndVisible()
        }
  	 ...
    }
}
```
 AppCoordinator의 시작은 **sceneDelegate**에서 불리게 됩니다. 


  
### ✔️ TabBarCoordinator(상위 coordinator) 구현
  다음으로 TabBarCoordinator를 구현해줍니다. 
  ```swift
  final class TabBarCoordinator : Coordinator {
    var parentCoordinator: Coordinator?
    var childCoordinator: [Coordinator] = []
    
    var navigationController: UINavigationController
    // coordinator 시작점
    func start() {
        goToHomeTabbar()
    }
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
        navigationController.isNavigationBarHidden = true
    }
    
    // tabBar 세팅 및 그에 따른 coordinator 세팅
    func goToHomeTabbar() {
        let tabbarController = CustomTabBarController()
        
        let mainNavigationController = UINavigationController()
        let mainCoordinator = MainCoordinator(navigationController: mainNavigationController)
        
        mainCoordinator.parentCoordinator = parentCoordinator
        
        let settingNavigationController = UINavigationController()
        let settingCoordinator = SettingCoordinator(navigationController: settingNavigationController)
        
        settingCoordinator.parentCoordinator = parentCoordinator
         
        
        tabbarController.viewControllers = [mainNavigationController,
                                            settingNavigationController]
        navigationController.pushViewController(tabbarController, animated: true)
        navigationController.isNavigationBarHidden = true
        
        parentCoordinator?.childCoordinator.append(mainCoordinator)
        parentCoordinator?.childCoordinator.append(settingCoordinator)
        
        
        mainCoordinator.start()
        settingCoordinator.start()
    }
}

  ```
 
 TabBarCoordinator를 시작하게 되면 **tabBar의 요소를 세팅**하고 각 **화면의 coordinator를 연결**하는 과정을 진행합니다.
 
 
### ✔️ MainCoordinator(하위 coordinator) 구현
```swift
final class MainCoordinator : Coordinator {
    var parentCoordinator: Coordinator?
    var childCoordinator: [Coordinator] = []
    
    var navigationController: UINavigationController

    func start() {
        presentMainVC()
    }
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
        navigationController.isNavigationBarHidden = true
    }
}

```

### ✔️ SettingCoordinator(하위 coordinator)구현
```swift
final class SettingCoordinator : Coordinator {
    var parentCoordinator: Coordinator?
    var childCoordinator: [Coordinator] = []
    
    var navigationController: UINavigationController

    func start() {
        presentSetting()
    }
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
        navigationController.isNavigationBarHidden = true
    }
}

```

이렇게 기본적인 coordinator 구현을 마쳤습니다. 이제는 flow 로직을 대신하는 coordinator에게 VC가 **"이제 버튼을 눌렀어 ~~ "** 하고 알려주는 대리자, **MainNavigation**을 구현해보겠습니다. 이를 **delegate 패턴**으로 구현해줄 것입니다. 

### ✔️ MainNavigation 구현
```swift
protocol MainNavigation : AnyObject {
    func presentMainVC()
    func presentRegisterVC()
}

final class MainViewController : BaseViewController {
    weak var coordinator : MainNavigation?
    
    init(coordinator: MainNavigation) {
        self.coordinator = coordinator
        super.init()
    }
    
    ...
    
    private func tabCameraButton() {
        coordinator?.presentRegisterVC()
    }
    
    ...
```

이후 이 **VC가 보내는 것을 MainCoordinator가 알아야**하므로
```swift
extension MainCoordinator : MainNavigation, RegisterImageNavigation, ImageResultNavigation {
    func presentMainVC() {
        let mainVC = MainViewController(coordinator: self)
        navigationController.pushViewController(mainVC, animated: true)
    }
    ...
```

MainVC의 coordinator를 **자기 자신**으로 정해줍니다. 

같은 방법으로 **SettingNavigation**을 구현하여 VC와 Coordinator를 연결해줍니다.
<br>

이렇게 하면 coordinator 패턴의 구현은 끝이 나게 됩니다. 

## 마무리하며..✨
***
> **coordinator 패턴의 정답은 없다 !**

많은 레퍼런스를 보면서 다양한 구조의 coordinator 패턴이 존재한다는 것을 확인했습니다. 

내가 하고자했던 앱의 구조는 **TabbarController**와 **NavigationController**를 함께 사용해야했습니다. 이렇듯 복잡한 프로젝트에 coordinator 패턴을 접목시키기 위해 정보를 찾아보며, 또한 기본적인 coordinator 패턴의 법칙들을 이해하며 공부하였습니다. 

기본적으로 **coordinator**는 

- VC가 화면 전환 요청을 할 경우 coordinator는 화면을 생성한다.
- parentCoordinator,childCoordinator 프로퍼티를 통해 계층관리를 한다. 
- start 메서드를 사용해 시작하고자하는 화면을 시작한다. 

이러한 규칙을 갖습니다. 
<br>

공부하면서 정리한 글이라 오류가 많을 수 있습니다!

글을 읽어주셔서 감사합니다😊


## 참고 📋
***
[https://zeddios.medium.com/coordinator-pattern-bf4a1bc46930](https://zeddios.medium.com/coordinator-pattern-bf4a1bc46930)

[https://khanlou.com/2015/01/the-coordinator/](https://khanlou.com/2015/01/the-coordinator/)

[https://khanlou.com/2015/10/coordinators-redux/](https://khanlou.com/2015/10/coordinators-redux/)


