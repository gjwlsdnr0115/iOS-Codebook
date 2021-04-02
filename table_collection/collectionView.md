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

Supplementary View
- Header, Footer
- storyboard에서 추가할 수 있고
- 만약 동일한 view를 여러 collectionView에 사용한다면 custom class 생성 후 사용

Custom Class
```
class HeaderCollectionReusableView: UICollectionReusableView {
        
    @IBOutlet weak var sectionTitleLabel: UILabel!
}
```
or
```
class FooterCollectionReusableView: UICollectionReusableView {
    var sectionFooterLabel: UILabel!
    
    private func setup() {
        let lbl = UILabel(frame: bounds)
        lbl.translatesAutoresizingMaskIntoConstraints = false
        lbl.font = UIFont.boldSystemFont(ofSize: 20)
        addSubview(lbl)
        
        if #available(iOS 11.0, *) {
            lbl.leadingAnchor.constraintEqualToSystemSpacingAfter(leadingAnchor, multiplier: 1.0).isActive = true
        } else {
            lbl.leadingAnchor.constraint(equalTo: leadingAnchor).isActive = true
        }
        lbl.centerYAnchor.constraint(equalTo: centerYAnchor).isActive = true
        
        sectionFooterLabel = lbl
    }
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }
}
```
만약 스토리보드가 아닌 코드로 추가한다면
```
listCollectionView.register(FooterCollectionReusableView.self, forSupplementaryViewOfKind: UICollectionElementKindSectionFooter, withReuseIdentifier: "footer")
```

Flow Layout 통해서 supplementary view 속성 바꾸기
```
if let layout = listCollectionView.collectionViewLayout as? UICollectionViewFlowLayout {
    layout.sectionHeadersPinToVisibleBounds = true  // sticky
    layout.footerReferenceSize = CGSize(width: 50, height: 50)  // 코드로 register 한 view는 따로 크기 지정해야 한다
}
```
dataSource
```
func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, at indexPath: IndexPath) -> UICollectionReusableView {

    // multiple supplementary views
    switch kind {
    case UICollectionElementKindSectionHeader:
        let header = collectionView.dequeueReusableSupplementaryView(ofKind: kind, withReuseIdentifier: "header", for: indexPath) as! HeaderCollectionReusableView
        header.sectionTitleLabel.text = list[indexPath.section].title
        return header
    case UICollectionElementKindSectionFooter:
        let footer = collectionView.dequeueReusableSupplementaryView(ofKind: kind, withReuseIdentifier: "footer", for: indexPath) as! FooterCollectionReusableView
        footer.sectionFooterLabel.text = list[indexPath.section].title
        return footer
    default:
        fatalError("Unsupported kind")
    }    
}
```
동적으로/개별 section 크기 지정
- UICollectionViewDelegateFlowLayout
```
extension SupplementaryViewViewController: UICollectionViewDelegateFlowLayout {

    // header 크기 지정
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, referenceSizeForHeaderInSection section: Int) -> CGSize {
        
    }
    
    // footer 크기 지정
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, referenceSizeForFooterInSection section: Int) -> CGSize {
        
    }
}
```

추가적인 api
```
listCollectionView.supplementaryView(forElementKind: String, at: IndexPath)
listCollectionView.indexPathsForVisibleSupplementaryElements(ofKind: String)
listCollectionView.visibleSupplementaryViews(ofKind: String)
```

## Selection
- multi-selection / selection 비활성화는 코드를 통해서만 설정 가능

Single selection
```
listCollectionView.allowsSelection = true
listCollectionView.allowsMultipleSelection = false
```

Delegate methods

```
func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
    let color = list[indexPath.item].color
    view.backgroundColor = color
        
    // 선택 시 셀의 색 바꾸기
    if let cell = collectionView.cellForItem(at: indexPath) {
        if let imageView = cell.contentView.viewWithTag(100) as? UIImageView {
            imageView.image = checkImage
        }
    }
}
```

- 셀 선택하기 직전에 호출
- true 리턴해야 실제로 선택된다
```
func collectionView(_ collectionView: UICollectionView, shouldSelectItemAt indexPath: IndexPath) -> Bool {
        
    // 현재 선택되어 있는 indexPaths
    guard let list = collectionView.indexPathsForSelectedItems else {
        return true
    }        
    return !list.contains(indexPath)
}
```
- 선택 해제
- true 리턴해야 실제로 선택 해제
- ex) 선택 해제 전에 사용자가 특성 행동 해야한다면 false 리턴하고 경고 메세지 띄우기
```
func collectionView(_ collectionView: UICollectionView, shouldDeselectItemAt indexPath: IndexPath) -> Bool {
        return true
    }
```
- 선택 해제 후 UI 업데이트 할때 활용
```
func collectionView(_ collectionView: UICollectionView, didDeselectItemAt indexPath: IndexPath) {

}
```

- highlight 전에 호출
- true 리턴해야 highlight
- false 리턴하면 highlight 선택 모두 안됨
    
```
func collectionView(_ collectionView: UICollectionView, shouldHighlightItemAt indexPath: IndexPath) -> Bool {
    return true
}
```
    
- highlight 된 이후
```
func collectionView(_ collectionView: UICollectionView, didHighlightItemAt indexPath: IndexPath) {
    if let cell = listCollectionView.cellForItem(at: indexPath) {
        cell.layer.borderWidth = 6
    }
}
```

- highlight 해제 후 호출
```
func collectionView(_ collectionView: UICollectionView, didUnhighlightItemAt indexPath: IndexPath) {
    if let cell = listCollectionView.cellForItem(at: indexPath) {
        cell.layer.borderWidth = 0.0
    }
}
```
코드로 셀 선택
- nil 전달하면 모든 선택 해제
```
let targetIndexPath = IndexPath(item: item, section: 0)
listCollectionView.selectItem(at: targetIndexPath, animated: true, scrollPosition: .centeredHorizontally)
```
셀 선택 해제
```
listCollectionView.deselectItem(at: IndexPath, animated: Bool)
```
or
```
listCollectionView.selectItem(at: nil, animated: true, scrollPosition: .left)
        
// 스크롤 초기화
let firstIndexPath = IndexPath(item: 0, section: 0)
listCollectionView.scrollToItem(at: firstIndexPath, at: .left, animated: true)
```

## Editing
- 따로 편집모드 없음
- 편집에 필요한 UI 직접 구현
- api - insert, move, delete, reload

섹션에서 셀 insert/delete
- 반드시 배열에서 data 먼저 이동한 후 셀 이동
```
extension EditViewController: UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        if indexPath.section == 0 {
            selectedList.remove(at: indexPath.item)
            collectionView.deleteItems(at: [indexPath])
        } else {
            let deleted = colorList[indexPath.section-1].colors.remove(at: indexPath.item)
//            collectionView.deleteItems(at: [indexPath])
            
            let targetIndexPath = IndexPath(item: selectedList.count, section: 0)
            selectedList.append(deleted)
//            collectionView.insertItems(at: [targetIndexPath])
            // 이 메소드로 한번에 하는것도 가능
            collectionView.moveItem(at: indexPath, to: targetIndexPath)
        }
    }
}

```
Section 초기화
```
func emptySelectedList() {
    selectedList.removeAll()
    let targetSection = IndexSet(integer: 0)
    listCollectionView.reloadSections(targetSection)
}
```
Section 추가
```
func insertSection() {
    // 데이터 추가
    let sectionData = MaterialColorDataSource.Section()
    colorList.insert(sectionData, at: 0)

    // 섹션 추가    
    let targetSection = IndexSet(integer: 1)
    listCollectionView.insertSections(targetSection)
}
```
Section 삭제
```
func deleteSecondSection() {
    // 데이터 삭제
    colorList.remove(at: 0)

    // 섹션 삭제
    let targetSection = IndexSet(integer: 1)
    listCollectionView.deleteSections(targetSection)
    }
```
Section 이동
```
func moveSecondSectionToThird() {
    // 데이터 이동
    let target = colorList.remove(at: 0)
    colorList.insert(target, at: 1)
        
    // 섹션 이동
    listCollectionView.moveSection(1, toSection: 2)
}
```

## Reordering
- Pan gesture recognizer를 통해 구현
    - 스토리보드에서 object를 scene 독에 추가하고 collectionView와 아웃렛 연결
    - IBAction도 연결
```
@IBAction func handlePanGesture(_ sender: UIPanGestureRecognizer) {
    let location = sender.location(in: listCollectionView)
        
    switch sender.state {

    // 샐 이동 시작
    case .began:
        if let indexPath = listCollectionView.indexPathForItem(at: location) {
            listCollectionView.beginInteractiveMovementForItem(at: indexPath)
        }
    // 셀 이동 중
    case .changed:
        listCollectionView.updateInteractiveMovementTargetPosition(location)
    // 셀 이동 완료
    case .ended:
        listCollectionView.endInteractiveMovement()
    default:
        listCollectionView.cancelInteractiveMovement()
    }
}
```
Datasource에 메소드 구현하여 실제 데이터의 이동도 설정
- pan gesture 정상적으로 끝난 후 호츌
- 바뀐 위치에 따라 내부 데이터 이동 구현
```
func collectionView(_ collectionView: UICollectionView, moveItemAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {
    let target = list[sourceIndexPath.section].colors.remove(at: sourceIndexPath.item)
    list[destinationIndexPath.section].colors.insert(target, at: destinationIndexPath.item)
}
```

- 셀 이동 직전 호출
- true 리턴해야 실제로 이동 가능
```
func collectionView(_ collectionView: UICollectionView, canMoveItemAt indexPath: IndexPath) -> Bool {
    return true
}
```

## Prefetching Data
- 스크롤 성능 위해
- cell fetching & data fetching
- storyboard에서 prefetching 활성화

Cell prefetching
```
extension PrefetchingViewController: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return list.count
    }

    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath)
        
        if let imageView = cell.viewWithTag(100) as? UIImageView {
            if let image = list[indexPath.row].image {
                imageView.image = image
            } else {
                imageView.image = nil
                downloadImage(at: indexPath.row)
            }
        }
        
        return cell
    }
}

// Cell prefetching
extension PrefetchingViewController: UICollectionViewDelegate {
    
    // cell 화면에 표시되기 직전에 호출
    func collectionView(_ collectionView: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt indexPath: IndexPath) {
        if let imageView = cell.viewWithTag(100) as? UIImageView {
            if let image = list[indexPath.row].image {
                imageView.image = image
            } else {
                imageView.image = nil

            }
        }
    }
}
```
Data Prefetching
- prefetching 하다가 도중에 취소될 수도 있기 때문에 cancelDownload도 구현해야 한다 - 스크롤 빨리 할때
```
extension PrefetchingViewController: UICollectionViewDataSourcePrefetching {
    func collectionView(_ collectionView: UICollectionView, prefetchItemsAt indexPaths: [IndexPath]) {
        for indexPath in indexPaths {
            downloadImage(at: indexPath.item)
        }
    }
    
    func collectionView(_ collectionView: UICollectionView, cancelPrefetchingForItemsAt indexPaths: [IndexPath]) {
        for indexPath in indexPaths {
            cancelDownload(at: indexPath.item)
        }
    }
}
```

Download image 코드 예시
```
extension PrefetchingViewController {
    func downloadImage(at index: Int) {

        // 이미지 이미 다운 되었는지 확인
        guard list[index].image == nil else {
            return
        }
        
        // 동일한 이미지 다운로드 하는 작업이 존재하는지 확인
        let targetUrl = list[index].url
        guard !downloadTasks.contains(where: { $0.originalRequest?.url == targetUrl }) else {
            return
        }
        
        let task = URLSession.shared.dataTask(with: targetUrl) { [weak self] (data, response, error) in
            if let error = error {
                print(error.localizedDescription)
                return
            }
            
            if let data = data, let image = UIImage(data: data), let strongSelf = self {
                strongSelf.list[index].image = image
                let reloadTargetIndexPath = IndexPath(row: index, section: 0)
                DispatchQueue.main.async {
                    if strongSelf.listCollectionView.indexPathsForVisibleItems.contains(reloadTargetIndexPath) == .some(true) {
                        strongSelf.listCollectionView.reloadItems(at: [reloadTargetIndexPath])
                    }
                }
                
                strongSelf.completeTask()
            }
        }
        task.resume()
        downloadTasks.append(task)
    }
    
    
    func completeTask() {
        downloadTasks = downloadTasks.filter { $0.state != .completed }
    }
    
    // 다운 취소 메소드
    func cancelDownload(at index: Int) {
        let targetUrl = list[index].url
        guard let taskIndex = downloadTasks.index(where: { $0.originalRequest?.url == targetUrl }) else {
            return
        }
        let task = downloadTasks[taskIndex]
        task.cancel()
        downloadTasks.remove(at: taskIndex)
    }
}
```

Refresh Control
- control & action 선언
```
lazy var refreshControl: UIRefreshControl = { [weak self] in
    let control = UIRefreshControl()
    control.tintColor = self?.view.tintColor
    return control
}()
    
    
@objc func refresh() {
    DispatchQueue.global().async { [weak self] in
        guard let strongSelf = self else { return }
        strongSelf.list = Landscape.generateData()
        strongSelf.downloadTasks.forEach { $0.cancel() }
        strongSelf.downloadTasks.removeAll()
        Thread.sleep(forTimeInterval: 2)
            
        DispatchQueue.main.async {
            strongSelf.listCollectionView.reloadData()
            strongSelf.listCollectionView.refreshControl?.endRefreshing()
        }
    }
}
```
- viewDidLoad에서 설정
```
listCollectionView.refreshControl = refreshControl
refreshControl.addTarget(self, action: #selector(refresh), for: .valueChanged)
```

## 추가
- IndexPath.item vs IndexPath.row
    - tableView에서는 row, collectionView에서는 item 사용
- 실제 storyboard에서 셀 크기 조절할 때는 collectionView에다가 해야한다
    - cell에다가 하는 설정은 그냥 디자인을 위한 프로토타입 셀에만 적용됨
- Get random Int
```
let item = Int(arc4random_uniform(UInt32(list.count)))
```