---
layout: post
title: "RxMoya + Codable"
description: "RxSwift + RxMoya + Codable"
date: 2019-3-26
tags: [reactiveX, RxSwift, swift, iOS]
comments: true
share: true
---

# RxSwift + RxMoya + Codable
rxSwift를 코드로 쉽게 적용할 수 있는 방법 중 하나로 api통신 코드를 작성해 보았습니다.  
기존에 이용하던 Moya 대신 [rxMoya](https://github.com/Moya/Moya)를 이용하였으며, Swift4.0부터 JSON parsing을 처리해주는 프로토콜인 [Codable](https://developer.apple.com/documentation/swift/codable)도 같이 사용하였습니다.

## Codable로 모델 선언
Codable은 프로토콜입니다.  
이 프로토콜을 채택하여 json을 encode하거나 decode 할 수 있으며, class, struct, enum에서 모두 사용할 수 있습니다.  
저는 struct로 모델을 만들었습니다.

```swift
struct User: Codable {
    let login : String
    let avatarUrl : String
    let reposUrl : String
    
    private enum CodingKeys: String, CodingKey {
        case login
        case avatarUrl = "avatar_url"
        case reposUrl = "repos_url"
    }
}
```
기본적으로 JSON의 키와 동일해야 하며 모델의 키가 JSON의 키와 다를경우 CodingKey를 사용하여 다시 맵핑해줄 수 있습니다.

## rxMoya 설치
```ruby
pod 'Moya/RxSwift'
```

## Provider 작성
github의 user 정보를 받아오는 provider를 작성합니다.

```swift
import Moya

enum provider {
    case userInfo(name: String)
}

extension provider: TargetType {
    var headers: [String : String]? {
        return nil
    }
    
    var baseURL: URL {
        return URL(string: "https://api.github.com")!
    }
    
    var path: String {
        switch self {
        case .userInfo(let name):
            return "/users/\(name)"
        }
    }
    
    var method: Moya.Method {
        return .get
    }
    
    var sampleData: Data {
        return "".data(using: .utf8)!
    }
    
    var parameters: [String: Any]? {
        return nil
    }
    
    var task: Task {
        return .requestPlain
    }
}
```

## 요청 코드 작성
**Moya Code**
```swift
    func userInfo(name: String) {
        let provider = MoyaProvider<provider>()
        provider.request(.userInfo(name: name)) { (result) in
            switch result {
            case .success(let response):
                guard let user = try? JSONDecoder().decode(User.self, from: response.data) else { return }
                print(user.login)
            case .failure(let error):
                print(error)
            }
        }    
    }
```
**rxMoya Code**
```swift
    func userInfo(name: String) {
        let provider = MoyaProvider<provider>()
        provider.rx.request(.userInfo(name: name))
            .map(User.self)
            .asObservable()
            .subscribe(onNext: { (user) in
                print(user.login)
            }, onError: { (error) in
                print(error)
            })
            .disposed(by: disposeBag)
    }
```
rx답게 읽어보자면, provider의 rx에게 요청 `request`하고 모델화 `map` 한 다음 실행 `subscribe` 할게! 정도 되려나요? 🙃  
어쨌든 선형적으로 읽히긴 합니다. 더 고급스러운 코드를 짠다면 장점이 더욱 부각될거에요.

## UI Event binding
Github User 정보를 가져오기 위해서 사용자가 입력한 이름을 얻어내고자 합니다.  
`UITextField`와 `UIButton`에서 이벤트를 가져오는 코드로, 사용자가 이름을 입력하면 버튼이 활성화되고 아무것도 입력하지 않으면 비활성화됩니다.

**Legacy Code**   
textField의 `delegate`를 설정하고 delegate method에서 사용자가 입력한 문자열의 유효성을 검사합니다.  
(예제라서 아주 간단하게 검사했습니다...🙄)
```swift
    nameField.delegate = self
```
```swift
    func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
        getButton.isEnabled = string.count > 0 && !string.contains(" ")
        return true
    }
```
버튼이 활성화 되고 사용자가 버튼을 누르면 입력한 이름을 가지고 정보를 요청합니다.
```swift
    guard let name = nameField.text else { return }
    userInfo(name: name) // 요청함수 호출
```

**rx Code**  
위의 과정을 rxSwift로 작성한 코드입니다.  
먼저 textField의 값을 바인딩 할 수 있는 **BehaviorSubject**를 만듭니다.  
(**BehaviorSubject** : 가장 최근에 발생한 아이템을 배출, 이 후 observable에 의한 결과 아이템을 계속 배출, `초기값`이 반드시 있어야 함)  
```swift
    let name: BehaviorSubject<String> = BehaviorSubject(value: "")
```
textField의 값을 subject에 바인딩 합니다.
```swift
    func bindEvent() {
        nameField.rx.text.orEmpty
                .bind(to: name)
                .disposed(by: disposeBag)    
    }
```
그 다음, subject를 통해 사용자가 입력한 문자를 검사합니다.
```swift
    func bindEvent() {
        nameField.rx.text.orEmpty
                .bind(to: name)
                .disposed(by: disposeBag)   

        name.map(validName)
            .bind(to: nameValid)
            .disposed(by: disposeBag)
    }

    func validName(name: String) -> Bool {
        return !name.contains(" ")
    }
```
이제 get button을 활성화 하기 위해서 Bool 타입의 subject를 만들고, 입력값을 검사하여 버튼의 활성화 여부를 결정합니다.  
사실, 굳이 subject를 만들 필요까진 없어보일 수 도 있지만, 한번 써보고 싶었어요...😬
```swift
    let nameValid: BehaviorSubject<Bool> = BehaviorSubject(value: false)

    func bindEvent() {
        nameField.rx.text.orEmpty
            .bind(to: name)
            .disposed(by: disposeBag)
        
        name.map(validName)
            .bind(to: nameValid)
            .disposed(by: disposeBag)
        
        nameValid.subscribe(onNext: { [weak self] (valid) in
            self?.getButton.isEnabled = valid
        }).disposed(by: disposeBag)
    }
```
여기서 주목해야 할 것 중 하나는 `nameValid`가 `subscribe`할 때 **self**를 **weak**로 쓴다는 것입니다.  
`viewController`의 생명주기가 끝이 나더라도 이벤트 바인딩한 것이 메모리에서 해제되지 않기 때문에 그냥 self가 아니라 `weak self`를 써주어야 합니다.  
만약 `weak self`가 싫다면 아래의 방법도 가능합니다.
```swift
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        disposeBag = DisposeBag()
    }
```
viewController가 사라질 때 `disposeBag`을 다시 생성해주기만 하면 됩니다. (가방을 비워내는거죠 :D)  

## 호출
마지막으로 `userInfo()` 함수를 호출하여 값을 확인해보겠습니다.  
우선 `viewDidLoad()`에서 event binding을 준비하고, 버튼 액션이 있을 때 호출만 해주면 끝!  

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        bindEvent()
    }

    @IBAction func getButtonTouched(_ sender: Any) {
        if let name = try? name.value() {
            userInfo(name: name)
        }
    }
```

간단한 기능 구현을 통해서 rxSwift를 활용하는 방법에 대한 글이었습니다.  
잘못된 정보, 오류 코드가 있다면 피드백 환영합니다!  
모두 **Happy Coding!**🙏




