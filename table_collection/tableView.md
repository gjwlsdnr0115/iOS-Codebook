# table
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