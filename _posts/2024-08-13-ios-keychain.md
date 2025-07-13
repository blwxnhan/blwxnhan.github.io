---
layout: post
toc: true
title: "[iOS] KeyChain"
categories: iOS 
tags: [TIL, iOS]
author:
  - bowon han
---

오늘은 **KeyChain** 에 대해서 알아보겠습니다! 

## Key Chain Service란? 🤷🏻‍♀️
***
사용자를 대신해서 안전하게 작은 데이터 조각을 저장할 수 있는 디바이스 내부 암호화된 데이터 베이스입니다!
<br>

KeyChain은 여러 형태의 데이터를 **암호화된 데이터베이스에 저장할 수 있는 메커니즘 제공**하고, <br>
비밀번호에 한정되지 않고 신용카드 정보, 인증서 등 관리하는 암호화키 저장 시에도 사용합니다.

오늘은 KeyChain의 구성요소와, <br>
프로젝트에서 로그인을 구현하며, 서버에서 받아온 토큰 값을 저장하는 방법에 대해 자세히 알아보겠습니다!


## 구성요소 🍀
***
### KeyChainItem
- 비밀번호, 암호키 정보 저장 시에 KeyChain Item으로 패키징
- 데이터, item의 접근을 관리하고 검색할 수 있도록 하기 위한 공개 Attributes를 함께 패키징
- KeyChain 서비스는 데이터와 속성의 저장을 처리하고 암호화도 처리
    - 이후 권한 있는 프로세스는 keychain 서비스를 아이템을 찾고 데이터를 복호화하는데 사용


### KeyChains
- 전체 키체인을 생성하고 관리하는것
- iOS에서는 앱이 하나의 keychain에만 접근이 가능


### 아이템 클래스의 키와 값
- 키체인 아이템의 각 클래스는 적용되는 속성을 규정
- 시스템이 암호화 여부를 결정함

|**Item Class Value**|---|
|---|---|
|kSecClassGenericPassword|일반 비밀번호|
|kSecClassInternetPassword|인터넷 비밀번호|
|kSecClassCertificate|인증서|
|kSecClassKey|암호화 키|
|kSecClassIdentity|ID 항목|
 

***
- 암호화 키에 대해 알아보자!
    - kSecAttrLabel: 아이템의 label을 나타내는 문자열의 값
    - kSecAttrApplicationLabel: 아이템의 애플리케이션 label을 나타내는 값


## KeyChain 데이터 저장, 읽기, 삭제 ⭐️
***
### KeyChain 저장 😊
```swift 
    private static func saveToken(token: String, key: String) -> Bool {
        if let data = token.data(using: .utf8) {
            let query: [String: Any] = [
                kSecClass as String: kSecClassGenericPassword,
                kSecAttrAccount as String: key,
                kSecValueData as String: data
            ]

            ///...
        }
    }
```

- 첫번째: 아이템 클래스의 키와 값
- 두번째: 아이템 클래스의 속성
- 네번째: 데이터를 Data 형태로 저장

키체인 서비스가 이 쿼리를 받아서 아이템으로 저장합니다.

```swift
    # 저장해주는 메서드
    let status = SecItemAdd(query, nil)

    if status == errSecSuccess {
        print("저장 성공")
    } else {
        print(status)
        print("저장 실패")
    }
```


### KeyChain 읽기 😊
```swift
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecReturnData as String: kCFBooleanTrue!,
        kSecMatchLimit as String: kSecMatchLimitOne /// 중복되는 경우, 하나의 값만 불러오기
    ]
    
    # 결과를 불러와서 저장할 변수
    var data: AnyObject?

    let status = SecItemCopyMatching(query as CFDictionary, &data)

    # 결과확인
    if status == errSecSuccess, let tokenData = data as? Data,
        let token = String(data: tokenData, encoding: .utf8) {
        
        if key == TokenType.accessToken.rawValue {
            print("AccessToken 불러오기 성공")
        // ...
```

```SecItemCopyMatching``` 메서드를 통해 데이터를 찾아 data에 저장합니다.

### KeyChain 삭제 😊
```swift
 private static func deleteToken(key: String) -> Bool {
    let query: NSDictionary = [
        kSecClass: kSecClassGenericPassword,
        kSecAttrAccount: key
    ]
    # 삭제 메서드
    let status = SecItemDelete(query)

    //...
 }
```

```SecItemDelete``` 메서드를 통해 저장된 데이터를 삭제합니다.



## 참고 📝
***
- [https://modelinspring.tistory.com/95](https://modelinspring.tistory.com/95)