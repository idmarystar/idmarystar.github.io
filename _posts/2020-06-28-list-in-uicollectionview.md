---
layout: post
title: "List in UICollectionView"
description: "WWDC2020에서 소개된 UICollectionView의 새로운 스타일에 대해 정리합니다."
date: 2020-06-28
tags: [WWDC, WWDC2020, UICollectionView, list, swift]
comments: true
share: true
---
 
이번 WWDC에서는 UICollectionView의 새로운 스타일이 소개되었습니다.
그 중 마치 UITableView처럼 보여지는 List에 대해 정리해 보았습니다. 
이 글은 WWDC2020의 [List in UICollectionView Video](https://developer.apple.com/videos/play/wwdc2020/10026)를 번역한 글입니다.  

![큰 이미지](/images/wwdc-collectionview-1.png)

## Lists
CollectionView의 Lists가 무엇인지 알아보도록 하겠습니다.   

Lists는 iOS14에서 제공되는 UICollectionView의 layout중의 하나로 UITableView와 비슷한 모양을 하고 있습니다.   

Lists는 작년에 도입한 Compositional Layout위에 만들어졌습니다. 따라서 매우 유연하고 Custom이 쉽습니다.   
특히 Lists의 self-sizing지원을 크게 향상시켰는데 Lists를 사용하게 되면 기본동작으로 self-sizing이 됩니다.  
즉, 더이상 AutoLayout을 수동으로 계산하지 않아도 ciollectionView가 처리해 준다는 것을 의미합니다.   

만약 수동조정이 필요한 경우 `preferredLayoutAttributesFittingAttributes:`를 overwriting 하여 조정할 수 있습니다.   

![큰 이미지](/images/wwdc-collectionview-2.png)
이것은 이모지를 나열한 Collection View입니다.   
이모지가 그룹별로 정렬되어 있어 접었다 펼 수 있으며 하단에는 UITableView와 매우 비슷한 모양이 있습니다. (이 셀에서는 스와이프하여 좋아하는 이모지를 표시할 수도 있습니다.)  
이 컬렉션 뷰는 Lists와 Composition Layout을 조합한 단일한 컬렉션 뷰 입니다.   
이것이 어떻게 동작하는지 깊이 들어가기 전에 List의 Componests를 먼저 살펴보겠습니다.    

## List Configuration
![큰 이미지](/images/wwdc-collectionview-3.png)
iOS14에서는 `UICollectionLayoutListConfiguration`이라는 새로운 타입을 제공합니다.     
이 것은 컬렉션뷰에서 Lists를 작성하기 위해 필요한 새로운 타입입니다.   
`UICollectionLayoutListConfiguration` 은 `NSCollectionLayoutSection`과 `UICollectionViewCompositionalLayout`을 기반으로 구축되었으며, iOS13에서 도입된 기존의 compositional layout system의 일부입니다.   

### Appearance
- `.plain`
- `.grouped`
- `.insetGrouped`
- `.sidebar`
- `.sidebarPlain`
List Configuration은 우리가 이미 알고 있는 tableView의 스타일과 동일한 모양(`.plain` `.grouped` `.insetGrouped`)을 제공하고 있으며 `.sidebar` `.sidebarPlain`이라는 Lists전용의 두가지 새로운 스타일도 제공합니다.
`.sidebar` `.sidebarPlain`을 사용한다면 iPadOS14에서 multi column app을 만들 수도 있습니다. (사진 참고)
![큰 이미지](/images/wwdc-collectionview-4.png)

### Separator, Headers, Footers
List Configuration은 일반적인 모양외에도 separator를 숨기거나 보이는 옵션을 제공하며 headers, footers를 구성할 수 있도록 해줍니다.

## How To Create a List
List를 만드는 간단한 방법은 우선 `UICollectionLayoutListConfiguration`을 작성하여 모양을 잡아준 다음 이 configuration을 사용하여 `UICollectionViewCompositionalLayout`을 만드는 것입니다.

```
    let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

우리는 .insetGrouped 스타일을 사용하여 UITalbleView의 inset group과 똑같아 보이게 할 수 있습니다. UITableView와 매우 유사하기 때문에 이 방법을 사용하는 것을 추천합니다.
하지만 section마다 셋업해주는 것이 좀 더 강력한 방법입니다. 이것을 `per-section setup`이라고 합니다.

```
    let layout = UICollectionViewCompositionalLayout() {
        [weak self] sectionIndex, layoutEnvironment in
        guard let self = self else { return nil }
        if sectionIndex == 0 {
            let section = self.createGridSection()
            return section
        } else {
            let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
            let section = NSCollectionLayoutSection.list(using: configuration, layoutEnvironment: layoutEnvironment)

            return section
        }
    }
```

per-section setup은 compositional layout을 작성하지 않고 대신 `per-section setup`을 사용하여 NSCollectionLayoutSection을 생성합니다. (하지만 `.insetGrouped`와 동일한 구성입니다.)

이 코드는 compositional layout의 기존 section provider의 initializer내에서 사용될 수 있으며, collection view의 모든 섹션에 호출되어 특정 섹션에 고유한 각각의 layout을 반환할 수 있습니다.

`per-section setup`을 잘 사용한다면 우리는 쉽게 종횡 스크롤이 함께 있는 컬렉션뷰를 구성할 수 있을 것입니다. (orthogonal scrolling)

## Headers and Footers
이제 List section에서 hedaer와 footer를 어떻게 구성하는지 살펴보겠습니다. 

CollectionView List의 header와 footer는 UITableView에서 쓰던 것과는 조금 다릅니다.
UICollectionView의 header와 footer는 명시적으로 활성화해야하며 이를 위해서 두가지 방법이 있습니다.

### Register ad Supplementary Views
// 코드 삽입 
첫번째 방법은 header와 footer를 supplementary view로 등록하는 것입니다.  
이렇게 구성하면 header혹은 footer가 렌더링될 때 collection view에게 supplementary view를 제공해야 합니다. 이를 위해서 diffalble datasource의 supplementaryViewProvider를 사용해도 되지만 UICollectionView의 delegate를 사용할 수도 있습니다.   
callback내에서 `elementKindSectionHeader` 혹은 `elementKindSectionFooter`에 대한 elementKind를 체크하고 적절한 view를 구성하여 리턴할 수 있습니다.

💡 supplementary view callback에서 nil을 리턴하면 안됩니다.  

```
    var configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    configuration.headerMode = sectionHasHeader ? .supplementary : .none
```
만약 일부 섹션에만 헤더가 필요하다면 per-section configuration을 사용할 수 있으며 특정 섹션에 헤더를 표시해야 하는지 여부에 따라 모드를 설정합니다(supplementary or none).  

### HeaderMode to firstItemInSection
```
    var configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    configuration.headerMode = .firstItemInSection
```
두번째 방법은 header에서만 사용할 수 있는 방법으로 headerMode를 firstItemInSection로 설정하여 사용하는 것입니다.
즉, collection view의 특정 섹션의 첫번째 셀을 헤더처럼 보이도록 구성하는 것입니다.
이 모드는 계층적 데이터 구조와 새로운 section snapshot APIs를 사용할 때 권장되지만 이 모드를 사용하는 경우 data source의 첫번째 아이템이 더 이상 섹션의 실제 컨텐츠를 나타내는 것이 아니므로 data source에서 이 부분을 인지하고 있어야 합니다. 

## List Cell
iOS14에서 UICollectionViewCell의 subclass인 UICollectionViewListCell을 소개하고자 합니다.
이 셀은 컬렉션 뷰의 compositional 특성을 유지하면서 List Section과 UICollectionViewCell을 함께 사용할 수 있습니다. 따라서 원하는 디자인을 구현하기 위해서는 API를 선택해서 사용하면 됩니다.

- Separators
- Indentation
- Swipe Actions
- Accessories
- Default Content Configurations

List Cell에서는 Indentation과 Separator의 삽입을 위해 좀 더 세분화된 지원을 제공하고 있으며 Swipe Action은 UITableView와 달리 Cell의 기능으로 추가되었습니다.
또한 Accessories API도 대폭 개선되었습니다. 

### Separators
![큰 이미지](/images/wwdc-collectionview-5.png)
image와 label 그리고 그 아래 separator를 렌더링하는 일반적인 레이아웃 예제입니다.  
그러나 알다시피 여기서 보여지는 레이아웃은 실제로는 올바르지 않습니다.  
separator는 셀에서 primary contents와 일치해야 하는데 이 예제에서는 primary contents가 imageView가 아니라 label입니다. 
따라서 leading side(측면)에서는 separator가 label의 가장자리와 일치하도록 삽입되어야 합니다.
![큰 이미지](/images/wwdc-collectionview-6.png)

UITableView에서는 이것을 separator inset이라고 하는 point 기반의 값을 제공하여 수행합니다. 
label의 X offset을 수동으로 계산하는 메서드가 이미 있었고 동일한 메서드를 사용하여 동일한 값으로 separator inset을 구성할 수 있었기 때문에 이 API가 도입되었을 때는 이것이 매우 쉬운일이었습니다.
그러나 safe area insets, layout margins, dynamic font sizes, SF Symbols가 있는 modern AutoLayout에서는 더 이상 쉬운일이 아니었습니다.
![큰 이미지](/images/wwdc-collectionview-7.png)

오늘날 우리는 dynamic font와 SF를 사용하여 이 같은 값들을 언제든지 변경할 수 있는 매우 동적인 환경을 갖추고 있습니다. 심볼 이미지의 크기도 사용자의 기본 font크기에 따라 변경될 수 있으며 그런 다음 레이블의 위치가 맞춰집니다. 따라서 레이블이 어디에서 나올지 미리 알기 어렵습니다.

### Separator Layout Guide
![큰 이미지](/images/wwdc-collectionview-8.png)
List Cell에서 새로운 개념인 Separatro Layout Guide라는 것을 소개합니다.
이 레이아웃 가이드는 기존 레이아웃 가이드와 달리 레이아웃 가이드에 제약을 가할 수  있습니다.  

separator layout guide를 설정하는 가장 쉬운 방법은 먼저 셀의 레이아웃을 구성하고 셀이 원하는 방식으로 표시되면 단일 제약 조건을 추가하는 것입니다.

![큰 이미지](/images/wwdc-collectionview-9.png)
separator layout guides의 leading anchor를 label의 leading anchor 또는 셀에 있는 주요 컨텐츠에 제약을 줍니다. 그런 다음 list cell과 list section을 함께 사용하면 셀의 주요 컨텐츠와 separator가 자동으로 정렬됩니다.  
시스템에서 제공하는 content configuration을 사용하는 경우 자동으로 이루어지므로 걱정할 필요는 없지만 Cell을 custom하는 경우 알아둬야 할 필요가 있습니다. 

## Swipe Actions
Swipe Action은 UITableView와 달리 Cell의 기능이 되었습니다.  
스와이프 액션은 셀의 컨텐츠와 함께 구성합니다. 따라서 imageView 또는 labe을 구성할 때마다 측면 또는 후면의(leading or trailing) 스와이프 액션 구성을 설정할 수 있습니다.

```
    let markFavorite = UIContextualAction(style: .normal, title: "Mark as Favorite") {
        [weak self] (_, _, completion) in
        guard let self = self else { return }
        self.markItemAsFavorite(with: item.identifier) // 셀의 indexPath를 캡쳐하지 않도록 한다. 
        completion(true)
    }
    cell.leadingSwipeActionsConfiguration = UISwipeActionsconfiguration(actions: [markFavorite])
```

셀과 레이아웃의 통신이 필요해지면서 list configuration으로 구성된 섹션에서 렌더링 된 경우에만 스와이프 액션이 지원됩니다. 만약 동적인 스와이프 액션 API가 필요하다면 leading 혹은 trailing 스와이프 액션 configuration getter를 overwrite한 다음 셀을 스와이프하려는 구성을 만들 수 
있습니다.

💡Note: 스와이프 액션의 액션 핸들러에서, 구성중인 셀의 indexPath를 캡처하지 않도록 합니다.

indexPath는 안정적인 식별자가 아닙니다.  
셀의 indexPath는 컨텐츠가 삽입되거나 삭제될 때 변경되기 때문에 특정 셀을 반드시 갱신하는 것은 아닙니다. 따라서 사용자가 스와이프 하는 액션을 트리거할 때 저장된 indexPath를 사용하여 셀의 데이터 모델을 가져오는 경우(fetch) 실제로는 다른 셀의 데이터를 조작하게 될 수 있습니다.  
이것은 잘못된 데이터를 delete 할 수 있기 때문에 delete action에서 특히 위험합니다. 대신, 데이터 모델을 직접 캡처하거나 이 셀의 컨텐츠를 식별하는데 사용할 수 있는 안정적인 식별자를 캡쳐해야 합니다. 
Diffable Data source와 안정적인 item 식별자 그리고 iOS14의 새로운 cell registration type은 이에 적합하다고 할 수 있습니다.  

## Accessories. 
UITableView의 accessories API는 상당히 제한적이었습니다.  
List Cell은 많은 액세서리 타입을 제공합니다. 셀의 leading과 trailing 모두에 대해 액세서리를 구성할 수 있으며 같은면에 여러 액세서리를 구성할 수도 있습니다.  

![큰 이미지](/images/wwdc-collectionview-10.png)
또한 List cell에서는 기능을 활성화할 수 있습니다.
- **Reorder** 
  re-ordering mode로 자동 설정이 되고 필요한 re-ordering 콜백도 구현할 수 있습니다.
- **Delete**
  자동으로 trailing swipe action이 수행됩니다.
- **Outline disclosure**
  셀이 자동으로 data source와 통신하여 셀의 하위항목을 확장하거나 축소시킵니다. 

### Accessories - How the API Works
셀의 액세서리를 구성하려면 List Cell의 단일 액세서리 속성을 UICellAccessories 배열로 설정하기 만하면 됩니다. 
```
    cell.accessories = [
        .disclosureIndicator(),
        .delete()
    ]
```

이 예제에서는 disclosure indicator와 delete 액세서리로 셀을 구성해 보겠습니다.  
시스템은 disclosure indicator가 항상 셀의 맨 끝에 있어야 하는 반면 delete accessory는 항상 셀의 맨 앞에 표시되는 것을 알고 있습니다. 따라서 UIKit은 액세서리를 올바른 순서대로 자동 정렬하여 적절한 면에 표시합니다.   
게다가 시스템은 disclosure indicator는 항상 보여야 하지만, delete accessory 컬렉션 뷰가 편집모드에 있을 때만 보여진다는 것도 알고 있습니다. 따라서 UIKit은 기존 편집 모드에 들어가거나 나올때 delete accessory를 자동으로 추정할 수 있습니다.  

이처럼 시스템에서 기본값을 많이 제공하고 있지만 대부분 Custom도 가능합니다. 예를들어 편집 모드가 아닌 경우에만 disclosure indicator를 표시하고 싶다면  displayed 매개 변수를 whenNotEditing 으로 설정하면 됩니다.  
```
    cell.accessories = [
        .disclosureIndicator(displayed: .whenNotEditing),
        .delete()
    ]
```


이것은 UIKit이 모든 상태를 처리하는 매우 선언적인 API입니다.
List는 모듈화 되어있고 매우 유연하며 많은 부분 사용자 정의가 가능한 레이아웃이라고 할 수 있습니다. 

긴 글 읽어주셔서 감사합니다. 🙏

🙌 의역과 오역이 있을 수 있습니다. 코멘트 주시면 반영하겠습니다. Happy Coding!


