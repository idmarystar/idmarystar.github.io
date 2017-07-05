---
layout: post
title: "iMessage App 만들기"
description: "iMessage Game App으로 게임대결 신청하기"
date: 2017-05-17
tags: [game, swift, spritekit, iMessage, iMessageApp, iOS, judgementDay]
comments: true
share: true
---

iOS 10 부터 아이폰 사용자들은 iMessage App을 좀 더 다양하게 사용할 수 있게 되었습니다.     
스티커를 전송하거나, 작업을 같이 하거나, 게임 대결을 신청할 수도 있는데, 그 중 게임대결 신청하기는 게임센터의 친구초대 기능을 대신한다고 할 수 있습니다.

## iMessage App
### 독립적으로 생성 가능
iMessage App은 기존 프로젝트에 타겟을 추가해서 생성할 수도 있지만 독립적으로 생성할 수도 있습니다.  
그래서 타겟을 설정할 때에도 Application Extension이 아니라 Application 항목에서 찾을 수 있습니다.

## Message bubble
상대방에게 메시지를 전송할 때 말풍선을 만듭니다.
말풍선에는 caption, image, imageTitle, trailing caption이 있으며 MSMessageTemplateLayout을 통해 요소를 생성합니다.  

[MSMessageTemplateLayout Document](https://developer.apple.com/documentation/messages/msmessagetemplatelayout)

```swift
let layout = MSMessageTemplateLayout()
layout.image = UIImage(named: "image.png")
layout.imageTitle = "image title"
layout.caption = "caption"

let message = MSMessage()
message.shouldExpire = true // 메시지를 읽은 후 사라지게 한다.
message.layout = layout
```

## MessagesAppViewController
MSMessagesAppPresentationStyle에는 두가지 타입이 있습니다.  
iMessage 대화하기 창에서 (키보드가 자리한 위치에) 작은창으로 보여주는 Compact,

### Compact View
![큰 이미지](/images/20170705/20170705-1.png)

전체화면으로 볼 수 있는 Expanded가 있습니다.
### Expanded View
![큰 이미지](/images/20170705/20170705-2.png)

두가지 스타일이 변경되기 전에 willTransition 메서드가 호출되며, 전달되는 인수인 presentationStyle에 맞는 뷰를 띄울 수 있습니다.
```swift
override func willTransition(to presentationStyle: MSMessagesAppPresentationStyle) {
    guard let conversation = activeConversation else {
        fatalError("Expected the active conversation")
    }
    
    self.presentViewController(for: conversation, with: presentationStyle)
}

func presentViewController(for conversation: MSConversation, with presentationStyle: MSMessagesAppPresentationStyle) {
    let viewController: UIViewController
    
    if presentationStyle == .compact {
        viewController = self.compactViewController!
    } else {
        viewController = self.instantiateExpandedVC()
    }
    
    self.addChildViewController(viewController)
    view.addSubview(viewController.view)
    viewController.didMove(toParentViewController: self)  
}

func presentExpandedViewController(with presentationStyle: MSMessagesAppPresentationStyle) {
    guard let conversation = activeConversation else {
        fatalError("Expected the active conversation")
    }
    
    self.presentViewController(for: conversation, with: presentationStyle)
    requestPresentationStyle(.expanded) // expanded 스타일을 띄웠으나 전체창으로 열리지 않을 때 추가
}
```

## Session
세션을 통해 메시지를 만들고 업데이트 합니다.
메시지를 업데이트하여 세션을 유지할 수도 있고 새로운 메시지를 만들어 새로운시 세션으로 만들 수도 있습니다.
세션이 없다면 각각의 단일 메시지로 표시 됩니다.

```swift
let session = conversation?.selectedMessage?.session ?? MSSession()
let message = MSMessage(session: session)
```

## Conversation
Conversation은 현재 선택된 메시지, 대화 참여자의 UUID를 가져올 수 있으며 텍스트, 스티커, 첨부파일을 보낼 수 있습니다.

```swift
let conversation = activeConversation
let session = conversation?.selectedMessage?.session ?? MSSession()

let layout = MSMessageTemplateLayout()
layout.image = UIImage(named: "devil.png")
layout.imageTitle = "image title"
layout.caption = "caption"
layout.subcaption = "\(String(describing: conversation?.localParticipantIdentifier))"

let message = MSMessage(session: session)
message.layout = layout
message.summaryText = "Sent Message"

conversation?.insert(message, completionHandler: nil)
```

## Game Scene 띄우기
iMessage App의 MSMessagesAppViewController도 UIViewController를 상속 받았으므로 별다른 어려움 없이 기존의 프로젝트와 동일하게 Scene을 로드할 수 있었습니다.
아직까지는 별다른 제약조건을 찾지 못했기 때문에 어느정도 복잡한 UI도 적용가능하리라 봅니다.

iMessage App은 기존 게임센터의 친구초대 기능과 달리, 사용자가 국한되는것이 아니라 내 연락처에 있는 모든 사용자에게 게임 대결을 신청할 수 있기 때문에 어느정도 사용자가 늘어나는 것도 기대해 볼 수 있을 듯 합니다.



