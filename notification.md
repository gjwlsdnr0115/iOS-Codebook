# Notification
- Notification
    - 객체들이 주고 받는 메세지
    - 특정 이벤트에 대한 observer 등록
    - 시각적으로 표시 X
- Local Notification
    - 알람처럼 지정된 시간에 사용자에게 알림을 전달
- Remote Notification
    - 일반적인 푸쉬 알람
    - 외부 서버에서 전달되는 알림

## Notification
Post Notification
- object는 notification을 보내는 객체 전달 ex) `self`
- 받는 쪽에서 누가 보냈는지 확인할 때 이거를 참고한다
- notification을 보낸 thread와 동일한 thread애서 observer가 처리한다 - UI 업데이트 없는지 조심
```
NotificationCenter.default.post(name: NSNotification.Name.NewValueDidInput, object: nil, userInfo: ["NewValue": "dummy"])
```
- `NSNotification.Name`에 extension 추가할 수 있다
```
extension NSNotification.Name {
    static let NewValueDidInput = NSNotification.Name("NewValueDidInputNotification")
}
```

Add Observer
- 객체로 등록
```
// Notification으로 온 데이터 처리하는 메소드
@objc func process(notification: Notification) {
    guard let value = notification.userInfo?["NewValue"] as? String else {
        return
    }
    valueLabel.text = value
    }
    
override func viewDidLoad() {
    super.viewDidLoad()
        
    // Observer 등록
    NotificationCenter.default.addObserver(self, selector: #selector(process(notification:)), name: NSNotification.Name.NewValueDidInput, object: nil)
    }
```

iOS 9 이전에 충돌 막으려면 따로 등록 해제 해야한다
```
deinit {
    NotificationCenter.default.removeObserver(self)
}
```
- closure로 등록
```
var token: NSObjectProtocol?

override func viewDidLoad() {
        super.viewDidLoad()
        
    token = NotificationCenter.default.addObserver(forName: NSNotification.Name.NewValueDidInput, object: nil, queue: OperationQueue.main) { [weak self] ( notification ) in
        guard let value = notification.userInfo?["NewValue"] as? String else {
            return
        }
        self?.valueLabel.text = value
    }
}
```
- 등록 해제
```
deinit {
    if let token = token {
        NotificationCenter.default.removeObserver(token)
    }
}
```