# Delegate Pattern
## 개념
- 객체가 다른 객체를 자신의 대리자로 지정
- 자신이 제공하는 일부 기능을 대리자가 수행하도록 위임
- ex) TableView, CollectionView
- ~delegate, ~dataSource로 된 속성 존재
- delegate에게 자신의 메소드 호출 (protocol)

## 코드
**delegate 지정**
```
// ViewController 내에서
TableView.delegate = self
TextField.delegate = self
```
**delegate protocol 구현**
```
extension TableViewViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return list.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        
        cell.textLabel?.text = list[indexPath.row]
        return cell
    }
}

extension TableViewViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        print(list[indexPath.row])
    }
}
```
**protocol 직접 구현**
- 메소드의 이름은 delegate의 이름으로 시작하는게 관례
- 첫 파라미터는 호출하는 객체

```
//SenderDelegate.swift

protocol SenderDelegate {
    func sender(_ vc: UIViewController, didInput value: String?)
    func senderDidCancel(_ vc: UIViewController)
}
```

```
//SenderViewController.swift

class SenderViewController {
    var delegate: SenderDelegate?
}


//RecieverViewController.swift

class RecieverViewController {
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if let vc = segue.destination.children.first as? SenderViewController {
            vc.delegate = self
        }
    }
}

extension RecieverViewController: SenderDelegate {
    func sender(_ vc: UIViewController, didInput value: String?) {
        //
    }
    
    func sesnderDidCancel(_ vc: UIViewController) {
        //
    }
}
```