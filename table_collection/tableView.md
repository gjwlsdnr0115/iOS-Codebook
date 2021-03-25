# TableView
## Basic
- 스토리보드에서 각 셀의 기본 템플릿 존재
    - 그냥 텍스트만 있을 경우 basic으로 설정
- dataSource, delegate 두개 구현 해야 한다

Cell 리턴하는 메서드는 최대한 간단하게 구현 - 16ms 이하
```
extension TableViewBasicsViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return list.count
        
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = list[indexPath.row]
        return cell
    }
    
    
}
```

## Multi Sections
- cell 종류의 개수만큼 스토리보드에서 prototype cell 추가
- tableView의 style, backgroundcolor 모두 설정 권장

원하는 셀의 타입이 없는 경우 직접 만들어야한다
- UITableViewCell 상속
- awakeFromNib에서 외형 설정
```
import UIKit

class SwitchTableViewCell: UITableViewCell {

    override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code
        
        let v = UISwitch(frame: .zero)
        accessoryView = v
    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)

        // Configure the view for the selected state
    }
}
```

Section 구분 필요한 경우
```
func numberOfSections(in tableView: UITableView) -> Int {
        list.count
}
```
필수 delgate methods
```
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return list[section].items.count
}
    
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let target = list[indexPath.section].items[indexPath.row]
        
    let cell = tableView.dequeueReusableCell(withIdentifier: target.type.rawValue, for: indexPath)
    cell.textLabel?.text = target.title
    switch target.type {
    case .disclosure:
        cell.imageView?.image = UIImage(systemName: target.imageName ?? "")
    case .switch:

        // 스위치를 넣은 accessoryView는 optional이기 때문에 - 스위치가 있는 경우에 isOn 설정을 해야한다
        if let switchView = cell.accessoryView as? UISwitch {
            switchView.isOn = target.on
        }
    case .action:
        break
    case .checkmark:
        cell.accessoryType = target.on ? .checkmark : .none
    }
        
    return cell
}
```
Header and footer
```
func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
    return list[section].header
}
    
func tableView(_ tableView: UITableView, titleForFooterInSection section: Int) -> String? {
    return list[section].footer
}
```
cell 안에 있는 스위치/버튼에 메서드 할당하기
```
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let target = list[indexPath.section].items[indexPath.row]
        
    let cell = tableView.dequeueReusableCell(withIdentifier: target.type.rawValue, for: indexPath)
    cell.textLabel?.text = target.title
    switch target.type {
    case .disclosure:
        cell.imageView?.image = UIImage(systemName: target.imageName ?? "")
    case .switch:
        if let switchView = cell.accessoryView as? UISwitch {
            switchView.isOn = target.on

            // remove target
            switchView.removeTarget(nil, action: nil, for: .valueChanged)
            // add new target
            if indexPath.section == 1 && indexPath.row == 0 {
                switchView.addTarget(self, action: #selector(toggleHideAlbum(_:)), for: .valueChanged)

            }
        }
    case .action:
        break
    case .checkmark:
        cell.accessoryType = target.on ? .checkmark : .none
    }
        
    return cell
}
```
Cell 터치 했을 때 구현
- indexPath의 section과 row를 통해 개별 셀에 접근
```
extension MultiSectionTableViewViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        
        if indexPath.section == 3 && indexPath.row == 0 {
            showActionSheet()
            tableView.deselectRow(at: indexPath, animated: true)  // 선택 해제
        }
        
        if indexPath.section == 4 {
            if let cell = tableView.cellForRow(at: indexPath) {
                list[indexPath.section].items[indexPath.row].on.toggle()
                cell.accessoryType = list[indexPath.section].items[indexPath.row].on ? .checkmark : .none
            }
        }
    }
}

```
화면 넘어갔다가 다시 올 때 선택 해제
```
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
        
    if let selected = listTableView.indexPathForSelectedRow {
        listTableView.deselectRow(at: selected, animated: true)
    }
}
```

## Separator
- separator
- color
- inset
    - 기본값: 왼쪽 - 15, 오른쪽 - 0
- 개별 셀의 separator 따로 설정 가능

viewDidLoad에서 설정
```
listTableView.separatorStyle = .singleLine
listTableView.separatorColor = UIColor.systemBlue
listTableView.separatorInset = UIEdgeInsets(top: 0, left: 0, bottom: 0, right: 0)
listTableView.separatorInsetReference = .fromCellEdges
```
개별 cell에 직접 설정
- cell은 재사용 되기 때문에 초기화 해야한다
```
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        
    if indexPath.row == 1 {
        cell.separatorInset = UIEdgeInsets(top: 0, left: 30, bottom: 0, right: 0)
    } else if indexPath.row == 2 {
        cell.separatorInset = UIEdgeInsets(top: 0, left: 0, bottom: 0, right: 30)
    } else {
        // cell은 재사용 되기 때문에 초기화 해야한다
        cell.separatorInset = listTableView.separatorInset
    }
        
    cell.textLabel?.text = list[indexPath.row % list.count]
    return cell
}
```

## Cell
cell 정보에 접근하려면
```
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    
    if let cell = tableView.cellForRow(at: indexPath) {
        print(cell.textLabel?.text ?? "none")
    }
}
```
cell로부터 indexPath에 접근
```
if let indexPath = listTableView.indexPath(for: cell) {
    if let vc = segue.destination as? DetailViewController {
        vc.value = list[indexPath.row]
    }
}
```
Cell 정보 다른 viewController로 전달
- cell 선택해서 새로운 화면 전환되기 직전 호출
- sender - 선택된 셀
```
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let cell = sender as? UITableViewCell {
        if let indexPath = listTableView.indexPath(for: cell) {
            if let vc = segue.destination as? DetailViewController {
                vc.value = list[indexPath.row]
            }
        }
    }
}
```
cell 정보 가져오는 추가적인 메소드
```
listTableView.indexPathForRow(at: CGPoint)
listTableView.indexPathsForRows(in: CGRect)
listTableView.visibleCells
listTableView.indexPathsForVisibleRows
```

## AccessoryView
- Disclosure Indicator
    - 주로 새로운 화면을 push 형태로 보여줄 때 사용
- Detail Button
    - 상세 정보 모달로 표시
    - 버튼 독립적으로 이벤트 처리 가능 - delegate method
- Detail Disclosure Button
- Checkmark

System AccessoryView
```
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        
    switch indexPath.row {
    case 0:
        cell.textLabel?.text = "Disclosure Indicator"
        cell.accessoryType = .disclosureIndicator
    case 1:
        cell.textLabel?.text = "Detail Button"
        cell.accessoryType = .detailButton
    case 2:
            
        cell.textLabel?.text = "Detail Disclosure Button"
        cell.accessoryType = .detailDisclosureButton
    case 3:
        cell.textLabel?.text = "Checkmark"
        cell.accessoryType = .checkmark
    default:
        cell.textLabel?.text = "None"
        cell.accessoryType = .none
    }
    return cell
}
```
Custom AccessoryView
- 코드로만 구현 가능
- cellForRow에서 하면 overhead 발생
- 새로운 셀 클래스를 만들고 초기화 시점에 한번만 구현되게 한다

Cell class 선언
- Custom accessory는 UIView 상속한 모든 뷰로 구성 가능
- 일반, 편집 모두 두가지 따로 가능 - `editingAccessoryType`
- accessoryView 사용하는 경우 accessoryType은 무시, nil인 경우에만 신경 쓴다

```
override func awakeFromNib() {
    super.awakeFromNib()
    // Initialization code
        
    let v = UIImageView(image: UIImage(systemName: "star"))
        accessoryView = v
//        editingAccessoryType
    }
```
Detail Button 이벤트 처리
```
func tableView(_ tableView: UITableView, accessoryButtonTappedForRowWith indexPath: IndexPath) {
    performSegue(withIdentifier: "modalSegue", sender: nil)
}
```

## Self Sizing
- Autolayout을 이용해 self sizing
- 만약 셀들이 고정된 크기여야 한다면 끄는게 낫다
    - scrolling 하는데 시간 단축

코드로 구현
```
listTableView.rowHeight = UITableViewAutomaticDimension
listTableView.estimatedRowHeight = UITableViewAutomaticDimension
```
각 셀마다 다른 높이 구현
```
extension SelfSizingCellViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        if indexPath.row == 0 {
            return 100
        }
        return UITableViewAutomaticDimension
    }
    
    // 이거까지 같이 해주면 스크롤 성능 향상
    func tableView(_ tableView: UITableView, estimatedHeightForRowAt indexPath: IndexPath) -> CGFloat {
        if indexPath.row == 0 {
            return 100
        }
        return UITableViewAutomaticDimension
    }
}
```
## Custom Cell
- 셀 높이 직접 정해줘야 한다

tag로 label 연결
```
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "customCell", for: indexPath)
        
    let target = list[indexPath.row]

    // 태그 통해 접근
    if let dateLabel = cell.viewWithTag(100) as? UILabel {
         dateLabel.text = "\(target.date), \(target.hoursFromGMT)시간"
    }

    if let locationLabel = cell.viewWithTag(200) as? UILabel {
        locationLabel.text = target.location
    }

    if let ampmLabel = cell.viewWithTag(300) as? UILabel {
        ampmLabel.text = target.ampm
    }

    if let timeLabel = cell.viewWithTag(400) as? UILabel {
        timeLabel.text = target.time
    }
        
    return cell
}
```
customCell Class 만들어서 연결 - 권장
```
class TimeTableViewCell: UITableViewCell {
    
@IBOutlet weak var dateLabel: UILabel!
    @IBOutlet weak var locationLabel: UILabel!
    @IBOutlet weak var ampmLabel: UILabel!
    @IBOutlet weak var timeLabel: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code
    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)

        // Configure the view for the selected state
    }

}
```
```
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "customCell", for: indexPath) as! TimeTableViewCell
        
        let target = list[indexPath.row]
        
        cell.dateLabel.text = "\(target.date), \(target.hoursFromGMT)시간"
        cell.locationLabel.text = target.location
        cell.ampmLabel.text = target.ampm
        cell.timeLabel.text = target.time
        
        return cell
    }
```
## Nib 파일로 cell 만들기
- identifier 스토리보드에서 따로 설정 안해도 됨
- 여러 tableView에서 사용 가능
- tableView에 register 하는것은 보통 viewDidLoad에서 한다
```
override func viewDidLoad() {
    super.viewDidLoad()
        
    let cellNib = UINib(nibName: "SharedCustomCell", bundle: nil)
    listTableView.register(cellNib, forCellReuseIdentifier: "SharedCustomCell")
}
```
## Section Header

일반적인 헤더
```
func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
    return list[section].title
}
```

tableView에 Custom header 등록
```
listTableView.register(UITableViewHeaderFooterView.self, forHeaderFooterViewReuseIdentifier: "header")
```
헤더에 표시할 텍스트 설정
```
func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
    let headerView = tableView.dequeueReusableHeaderFooterView(withIdentifier: "header")
        
    headerView?.titleLable.text = list[section].title        

        
    return headerView
}
```
그 외에 속성 바꿀 때
- 헤더를 표시하기 직전에 호출
- 해더에 포함된 레이블 속성 바꿀때
```
func tableView(_ tableView: UITableView, willDisplayHeaderView view: UIView, forSection section: Int) {
    if let headerView = view as? UITableViewHeaderFooterView {
    headerView.textLabel?.textColor = .systemBlue
    headerView.textLabel?.textAlignment = .center

        if headerView.backgroundView == nil {
        let v = UIView(frame: .zero)
        v.backgroundColor = .secondarySystemFill
        v.isUserInteractionEnabled = false
        headerView.backgroundView = v
        }
    }
}
```

Custom Class로 헤더 구현
```
class CustomHeaderView: UITableViewHeaderFooterView {

    @IBOutlet weak var titleLable: UILabel!
    @IBOutlet weak var countLable: UILabel!
    @IBOutlet weak var customBackgroundView: UIView!

    override func awakeFromNib() {
        super.awakeFromNib()
        
        countLable.text = "0"
        countLable.layer.cornerRadius = 30
        countLable.clipsToBounds = true
        backgroundView = customBackgroundView
    }
    
}
```
TableView에 레지스터
```
override func viewDidLoad() {
    super.viewDidLoad()
        
    let headerNib = UINib(nibName: "CustomHeader", bundle: nil)
    listTableView.register(headerNib, forHeaderFooterViewReuseIdentifier: "header")

}
```
Downcasting 하여 메소드 구현
```
func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
    let headerView = tableView.dequeueReusableHeaderFooterView(withIdentifier: "header") as! CustomHeaderView
        
    headerView.titleLable.text = list[section].title
    headerView.countLable.text = "\(list[section].countries.count)"
        

        
    return headerView
}
```
## Section Index Title
- 오른쪽에 있는 섹션 바로가기

delegate method
```
func sectionIndexTitles(for tableView: UITableView) -> [String]? {
    return list.map { $0.title }
//    return stride(from: 0, through: list.count, by: 2).map { list[$0].title }
    }
```
만약 일부 섹션만 보이게 했다면 이 메서드에서 맞게 구현해줘야 한다
```
func tableView(_ tableView: UITableView, sectionForSectionIndexTitle title: String, at index: Int) -> Int {
    return index*2
}
```

## 추가
- Downcasting
    - subclass로 casting하는 것
    - `as?`
        - 항상 optional 리턴함
        - 만약 casting 못했으면 `nil` 리턴
    - `as!`
        - 강제로 casting하고 force unwrapping