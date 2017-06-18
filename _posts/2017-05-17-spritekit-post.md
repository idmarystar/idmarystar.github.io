---
layout: post
title: "SpriteKit으로 게임 만들기"
description: "애플워치와 아이폰에서 모두 사용할 수 있는 간단한 게임을 만들게 되었습니다."
date: 2017-05-17
tags: [game, swift, spritekit, applewatch, watchgame]
comments: true
share: true
---

애플워치와 아이폰에서 모두 플레이 할 수 있는 간단한 게임을 만들 기회가 생겨서 spriteKit을 사용해본 경험을 정리해 보고자 합니다.

랜덤하게 발생하는 아이템을 각각 왼쪽과 오른쪽으로 올바르게 보내면 스코어가 쌓이는 게임으로, 약간의 애니메이션과 콤보모드가 있습니다. 

이미 Unity로 개발이 되어 다른 OS에서 서비스 되고 있는 게임이지만, 우리는 애플워치에서 구동되도록 해야하기 때문에 Unity로 개발된 버전을 그대로 가져다 쓸 수 없었습니다. 

우선, SpriteKit과 Unity에 대해 간략하게 비교해 보았습니다.

## SpriteKit vs. Unity
#### SpriteKit
* iOS에 내장 : 다른 라이브러리나 dependencies, 플러그인이 필요가 없다.
* swift와 xcode에 능숙하다면 배우기 쉽다.
* 애플이 작성했기 때문에 어려움없이 모든 디바이스에 적용할 수 있다. (패드, 워치, 티비)
* 무료이다.
* 왜 때문인지 발생하는 frame Drop...

#### Unity
* Cross-platform
* layout 편집이 쉽다.
* 게임을 위한 asset을 구입할 수 있다.
* 좀 더 복잡한 게임에 어울린다.

프로젝트 초반에는 SpriteKit이 생소해서 UIKit과 섞어서 작성을 했는데, Scene위에 ViewController의 요소들이 올라오기 때문에 적절하지 않았습니다.(몇번을 고쳤다는...) 

## SKSpriteNode 생성
```swift
let player = SKSpriteNode(imageNamed: "player")
player.position = CGPoint(x: size.width * 0.1, y: size.height * 0.5)
addChild(player)
```



