---
layout: post
toc: true
title: "[iOS] RequestInterceptor를 통한 자동로그인(토큰 재발급)을 구현해보자!"
categories: iOS
tags: [iOS]
author:
  - bowon han
---

진행 중인 프로젝트에서 로그인을 구현하게 되었습니다 😊

access token은 구현해본 경험이 있었지만, refresh token을 통해 토큰 갱신하는 것은 처음이라 미리 완벽하게 계획하고 구현하고자 하였습니다.

## 아이디어 💡
***

일단 제가 생각했던 로그인 플로우는 다음과 같았습니다. 

![](/images/ios-RequestInterceptor-1.png)


토큰은 KeyChain에 저장하기로 하였습니다.

앱 실행 시 KeyChain에 저장된 토큰이 있는지 확인하고, 토큰이 존재한다면 유효한지 확인합니다. <br>
유효하지 않다면 refresh token으로 재발급이 가능한지 확인하여 자동로그인 구현을 하고자 하였습니다. 


## 고민 😭
***

사용자가 앱을 사용하던 중 토큰이 만료되는것을 대비하여, <br>
네트워크 로직에 모든 API 로직에 반환된 status 코드를 확인하여 **401 반환 시에 재발급 API를 호출**하고,<br>
**원래의 API를 재호출하는 로직을 추가**하는 것은 비효율적이고, 복잡한 과정이 될것이라고 생각이 들었습니다. <br> 

그렇게 알아보던 중 **RequestInterceptor**라는 것을 알게 되었습니다. <br>
이것을 통해 토큰의 재발급 과정을 간단하게 구현할 수 있었습니다🍀


## RequestInterceptor 구현 ⭐️
***

RequsetInterceptor는 Alamofire에서 지원하는 프로토콜입니다.

![](/images/ios-RequestInterceptor-2.png)

**RequestAdaptor**와 **RequestRetrier**를 채택하고 있습니다. 

### RequestAdaptor ✔️
RequestAdaptor의 adapt메서드는 request를 하기 전 어떤 특정 작업을 하고자할때 사용합니다. 
![](/images/ios-RequestInterceptor-3.png)

요청을 보내기 전 헤더에 토큰을 넣어서 헤더값을 세팅하는 등의 과정을 주로합니다. 

저는 Refresh token을 통해 access token이 재발급되었을 떄 해당 메서드가 실행되도록 구현하였습니다. 
```swift 
func adapt(_ urlRequest: URLRequest, for session: Session, completion: @escaping (Result<URLRequest, Error>) -> Void) {
    /// 토큰이 재발급 되었을 때 
    /// 재발급 받은 토큰으로 다시 Bearer 토큰 값 세팅 
    if isTokenRefreshed {
        var modifiedRequest = urlRequest
        guard let accessToken = KeyChain.loadAccessToken(key: TokenType.accessToken.rawValue) else { return }
        modifiedRequest.setValue("Bearer \(accessToken)", forHTTPHeaderField: "Authorization")
        
        isTokenRefreshed = false
        completion(.success(modifiedRequest))
    } else {
        completion(.success(urlRequest))
    }
}
```

### RequestRetrier✔️
RequestRetrier의 retry 메서드는 요청을 재시도해야하는지 여부를 반환하는 메서드입니다. 

![](/images/ios-RequestInterceptor-4.png)

completion으로 RetryResult를 넘겨 재시도 유형에 대해 알려줍니다. 

![](/images/ios-RequestInterceptor-5.png)

따라서 프로젝트에서는 통신이 실패했을 때, 즉 401을 반환했을 때 retry 하도록 구현하였습니다. 
```swift 
func retry(_ request: Request, for session: Session, dueTo error: Error, completion: @escaping (RetryResult) -> Void) {
    guard let response = request.task?.response as? HTTPURLResponse, response.statusCode == 401, let pathComponents =
            request.request?.url?.pathComponents,
            !pathComponents.contains("reissue")
    else {
        completion(.doNotRetryWithError(error))
        return
    }

    APIService.postReissue.performRequest { [weak self] result in
      switch result {
      case .success(let result):
          guard let result = result as? CommonResponseDtoSignInResDto
          else {
              completion(.doNotRetry)
              return
          }
          
          let keyChainResult = KeyChain.saveTokens(accessKey: result.result.accessToken, refreshKey: result.result.refreshToken)
          
          if keyChainResult.accessResult == true && keyChainResult.refreshResult == true {
              self?.isTokenRefreshed = true
              
              guard request.request != nil else {
                  completion(.doNotRetry)
                  return
              }
              
              completion(.retry)
          } else {
              self?.isTokenRefreshed = false
              completion(.doNotRetry)
          }
      case .failure(let error):
          self?.isTokenRefreshed = false
          self?.changeLoginViewController()
          completion(.doNotRetryWithError(error))
          print("⚠️refreshToken 만료")
      }
  }
}
```

retry 메서드에서는 토큰 재발급 API 를 호출하게 되는데 해당 API 도 401 을 반환할 수 있어 계속 adapt, retry 가 무한으로 반복될 가능성이 있습니다. 

따라서 토큰 재발급 API 의 반복 호출을 막기 위해 guard let 구문을 통해 해당 path에서 reissue 이라는 String 이 존재한다면 정지하도록 구현하였습니다.

저의 프로젝트에서는 RequestInterceptor를 채택하는 class를 만들어 구현하였습니다. 


### 사용할때는? 🤷🏻‍♀️
해당 interceptor를 사용할 때는 AF.request() 요청시에 파라미터로 삽입해주면 됩니다!

```swift
AF.request(request, interceptor: APIInterceptor.shared)
    .validate(statusCode: 200..<300)
    .responseJSON { response in 
    ...} 
```


## 참고 📜
***
[https://ios-development.tistory.com/730](https://ios-development.tistory.com/730)

