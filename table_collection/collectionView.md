# Collection View
## Basic
- DataSource
- Delegate
- Layout

DataSource
- 섹션 개수
```
func numberOfSections(in collectionView: UICollectionView) -> Int {
    return list.count
}
```
- 셀 개수
```
func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    return list[section].colors.count
}
```
- 셀 설정
```
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath)
        
    cell.contentView.backgroundColor = list[indexPath.section].colors[indexPath.row]
        
    return cell
    }
```

## Flow Layout
- collectionView 내부의 view의 배치를 담당
- Layout 간 전환 가능 (애니메이션 가능)
- 속성
    - `minimumInteritemSpacing`
    - `minimumLineSpacing`
    - `sectionInset`
- 개별 셀 / 동적으로 속성 바꾸려면 delegate 메소드를 통해서 한다

코드로 구현
```
// 접근할 때는 downcasting 해줘야 한다
listCollectionView.collectionViewLayout as? UICollectionViewFlowLayout
```
storyboard에서도 설정 가능
```
if let layout = listCollectionView.collectionViewLayout as? UICollectionViewFlowLayout {
    // cell size
    layout.itemSize = CGSize(width: 100, height: 100)
    // cell 여백
    layout.minimumInteritemSpacing = 10
    layout.minimumLineSpacing = 10
            
    layout.sectionInset = UIEdgeInsets(top: 40, left: 40, bottom: 20, right: 20)
}
```
Change scroll direction
```
@objc func toggleScrollDirection() {
    guard let layout = listCollectionView.collectionViewLayout as? UICollectionViewFlowLayout else {
        return
    }
        
    // animation
    listCollectionView.performBatchUpdates ({
        layout.scrollDirection = layout.scrollDirection == .vertical ? .horizontal : .vertical
    }, completion: nil)

}
```
animation 설정
```
listCollectionView.performBatchUpdates ({
        layout.scrollDirection = layout.scrollDirection == .vertical ? .horizontal : .vertical
    }, completion: nil)
```
Cell 크기 직접 설정
- 크기를 리턴하는 delegate method가 가장 먼저 호출된다
    - layout 미리 설정
    - 다른 delegate method으로 layout 변경한다면 별도로 가져와야한다

 UICollectionViewDelegateFlowLayout

 - 셀 배치 전 크기 설정할 때 호출
 - 전체 설정 보다 우선 순위 높다
 ```
 func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
    guard let layout = collectionViewLayout as? UICollectionViewFlowLayout else { return CGSize.zero }
        
        // 전체 화면의 높이 가져오기
        var bounds = collectionView.bounds
        bounds.size.height += bounds.origin.y
        
        // 사용될 화면의 높이 넓기 계산 (여백 제외)
        var width = bounds.width - (layout.sectionInset.left + layout.sectionInset.right)
        var height = bounds.height - (layout.sectionInset.top + layout.sectionInset.bottom)
        

        // 개별 셀의 높이 넓이 설정
        switch layout.scrollDirection {
        case .vertical:
            height = (height - (layout.minimumLineSpacing * 4)) / 5
            if indexPath.item > 0 {
                width = (width - (layout.minimumInteritemSpacing * 2)) / 3
            }
            
        case .horizontal:
            width = (width - (layout.minimumLineSpacing * 2)) / 3
            if indexPath.item > 0 {
                height = (height - (layout.minimumInteritemSpacing * 4)) / 5
            }
        }
        
        // 소수점은 없애야 한다
        return CGSize(width: width.rounded(.down), height: height.rounded(.down))
        
    }
 ``` 
 - 다른 delegate 메소드로 layout을 동적으로 바꿨다면 별도로 값을 가져와야 한다
 ```
 let lineSpacing = self.collectionView(collectionView, layout: collectionViewLayout, minimumLineSpacingForSectionAt: indexPath.section)
let itemSpacing = self.collectionView(collectionView, layout: collectionViewLayout, minimumInteritemSpacingForSectionAt: indexPath.section)
let sectionInset = self.collectionView(collectionView, layout: collectionViewLayout, insetForSectionAt: indexPath.section)
 ```
Delegate로 layout 변경
- cell 여백값이 필요할 때 마다 호출
- 여기서 리턴하는게 여백 값
- 개별 셀 지정은 안되지만 섹션별로 지정은 가능
```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumInteritemSpacingForSectionAt section: Int) -> CGFloat {
    return 5
}
```
- lineSpacing 설정
```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumLineSpacingForSectionAt section: Int) -> CGFloat {
    return 5
}
```
- Section의 여백 설정
```
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, insetForSectionAt section: Int) -> UIEdgeInsets {
    return UIEdgeInsets(top: 5, left: 5, bottom: 5, right: 5)
}
```

## Custom Cell
- tableView와는 달리 accessoryView 없다
클래스 생성
```
class ColorCollectionViewCell: UICollectionViewCell {
    
    @IBOutlet weak var colorView: UIView!
    @IBOutlet weak var hexLabel: UILabel!
    @IBOutlet weak var nameLabel: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        colorView.clipsToBounds = true
        colorView.layer.cornerRadius = colorView.bounds.width / 2
    }
}
```

dataSource에서 설정
```
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath) as! ColorCollectionViewCell
        
    let target = list[indexPath.item]
    cell.colorView.backgroundColor = target.color
    cell.hexLabel.text = target.hex
    cell.nameLabel.text = target.title
        
    return cell
}
```
Cell의 정보 segue로 다른 VC에 전달
```
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    // sender을 셀로 casting
    guard let cell = sender as? ColorCollectionViewCell else { return }

    // cell 정보 가져오기    
    guard let indexPath = listCollectionView.indexPath(for: cell) else {
        return
    }
        
    let target = list[indexPath.item]
    segue.destination.view.backgroundColor = target.color
    segue.destination.title = target.title
}
```

Cell 정보 가져오는 methods etc.
```
listCollectionView.visibleCells
listCollectionView.cellForItem(at: IndexPath)
listCollectionView.indexPath(for: UICollectionViewCell)
listCollectionView.indexPathForItem(at: CGPoint)
```

Self-Sizing
```
if let layout = listCollectionView.collectionViewLayout as? UICollectionViewFlowLayout {
    layout.estimatedItemSize = UICollectionViewFlowLayoutAutomaticSize
}
```
## 추가
- IndexPath.item vs IndexPath.row
    - tableView에서는 row, collectionView에서는 item 사용
- 실제 storyboard에서 셀 크기 조절할 때는 collectionView에다가 해야한다
    - cell에다가 하는 설정은 그냥 디자인을 위한 프로토타입 셀에만 적용됨