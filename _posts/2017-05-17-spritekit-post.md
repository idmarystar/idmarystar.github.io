---
layout: post
title: "SpriteKit으로 게임 개발하기"
description: "iOS와 WatchOS에서 구동 가능한 게임 개발하기"
date: 2017-05-17
tags: [game, swift, spritekit, applewatch, watchgame]
comments: true
share: true
---

애플워치와 아이폰에서 모두 플레이 할 수 있는 간단한 게임을 만들 기회가 생겨서 spriteKit을 사용해본 경험을 정리해 보고자 합니다.

![큰 이미지](/images/20170517-1.jpg)
[Judgement Day - Heaven or Hell 다운로드하기](https://appsto.re/kr/JxHojb.i)

랜덤하게 발생하는 아이템을 각각 왼쪽과 오른쪽으로 올바르게 보내면 스코어가 쌓이는 게임으로, 약간의 애니메이션과 콤보모드가 있습니다. 

이미 Unity로 개발이 되어 다른 OS에서 서비스 되고 있는 게임이지만, 이번 프로젝트에서는 애플워치에서 구동되도록 해야하기 때문에 Unity로 개발된 버전을 그대로 가져다 쓸 수 없었습니다. 

우선, SpriteKit과 Unity에 대해 간략하게 비교해 보았습니다.

## SpriteKit vs. Unity
#### SpriteKit
* iOS에 내장 : 다른 라이브러리나 dependencies, 플러그인이 필요가 없다.
* swift와 xcode에 능숙하다면 배우기 쉽다.
* 애플이 작성했기 때문에 어려움없이 모든 디바이스에 적용할 수 있다. (패드, 워치, 티비)
* 무료이다.
* 아주 빠른 터치에서 Frame Drop이 종종 발생

#### Unity
* Cross-platform
* layout 편집이 쉽다.
* 게임을 위한 asset을 구입할 수 있다.
* 좀 더 복잡한 게임에 어울린다.

프로젝트 초반에는 SpriteKit이 생소해서 UIKit과 섞어서 작성을 했는데, Scene위에 ViewController의 요소들이 올라오기 때문에 적절하지 않았습니다.(몇번을 고쳤다는...) 

## SKSpriteNode 생성
SpriteKit에서 아이템들은 모두 spriteNode로 작성할 수 있습니다. 
```swift
let player = SKSpriteNode(imageNamed: "player")
player.position = CGPoint(x: size.width * 0.1, y: size.height * 0.5)
addChild(player)
```

## SKAction
생성한 node에 액션을 적용할 수 있습니다.
```swift
SKAction.move(to::duraion:) // 객체가 움직이는 것을 제어하고 시간과 속도를 변경할 수 있다.
SKAction.removeFromParent() // 부모노드로부터 노드를 지울 수 있다. 제 때 제거하지 않으면 리소스 낭비를 불러온다.
SKAction.sequence(_:) // 시퀀스 동작을 통해 한번에 하나씩 순서대로 수행되는 작업을 연결할 수 있다.
```

## Touch
location(in:) 및 previousLocation(in:) 메서드를 통해 SKNode의 좌표계내에서 터치 좌표를 얻어올 수 있습니다. watchOS에서는 다른 방식으로 터치 좌표를 얻어야 합니다.
```swift
override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    guard let touch = touches.first else {
        return
    }
    let touchLocation = touch.location(in: self)
}
```

## Collision Detection and Physics
SpriteKit의 물리엔진에 대한 간단한 정리입니다.

#### physics world
물리계산을 실행하기 위한 시뮬레이션 공간입니다.

physics world 중 하나는 기본적으로 scene에 설정되어 있으며, 중력과 같은 속성을 구성할 수 있습니다.

#### physics body
충돌 감지를 위해 각 sprite에 도형을 연결하고 속성을 설정합니다.

단순한 모양이라도 퍼포먼스가 충분하기 때문에 physics body가 sprite와 똑같을 필요는 없습니다.

#### physics Category

physics body에 설정할 수 있는 속성 중 하나로 자신이 속한 그룹을 나타내는 bitMask라고 할 수 있습니다.

category를 통해 physics body가 충돌할 때 어떤 종류의 sprite인지 알 수 있습니다.

```swift
struct PhysicsCategory {
  static let None      : UInt32 = 0
  static let All       : UInt32 = UInt32.max
  static let Monster   : UInt32 = 0b1       
  static let Projectile: UInt32 = 0b10      
}
```

category는 하나의 32비트 정수이며, 비트마스크 역할을 합니다.

정수의 32비트 각각이 하나의 카테고리를 나타내므로 최대 32개의 카테고리를 가질 수 있습니다.

#### contact delegate

두 physic body가 충돌할 때 이를 알려주는 contact delegate를 설정할 수 있습니다.

contact delegate에서 객체의 category를 확인하고 각각에 맞는 액션을 수행합니다.

## PhysicsBody의 생성과 속성

#### isDynamic

true이면 Sprite를 동적으로 설정 (물리엔진이 몬스터의 움직임을 제어하지 않고, 이전에 작성한 코드대로 움직이는것을 의미)

#### pinned

물리체가 부모노드에 고정되는지 여부를 묻는 값, 기본이 false이므로 설정하지 않으면 화면밖으로 사라짐(움직이는 물리체라면 상관없음)

#### categoryBitMask

미리 설정한 카테고리로 지정.

#### contactTestBitMask

sprite가 무엇에 충돌했는지 컨택리스너에게 알려주기 위함

#### collisionBitMask

물리엔진의 바운스 같은 컨택응답을 설정.

#### usesPreciseCollisionDetection

true이면 빠르게 움직이는 두 물체가 충돌했을 때 그냥 통과하는것을 방지한다.

phsicsBody는 아래와 같이 적용할 수 있습니다.

```swift
func addMonster() {
...
    monster.physicsBody = SKPhysicsBody(rectangleOf: monster.size) 
    monster.physicsBody?.isDynamic = true 
    monster.physicsBody?.categoryBitMask = PhysicsCategory.Monster 
    monster.physicsBody?.contactTestBitMask = PhysicsCategory.Projectile 
    monster.physicsBody?.collisionBitMask = PhysicsCategory.None 
...
}
```
처음 진행하는 게임프로젝트였고, spriteKit을 처음 적용해보았기 때문에 생각보다 시간이 많이 걸렸고, 애플워치 때문에 spriteKit으로 구현은 했지만, 막상 애플워치에서 무리없이 구동된다고 하기에는 무리가 있었습니다. 복잡하고 안정성을 요하는 게임에는 아무래도 Unity가 적절하지 않나 생각합니다.

[Judgement Day - Heaven or Hell 다운로드하기](https://appsto.re/kr/JxHojb.i)




