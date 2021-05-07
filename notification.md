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

## Local Notifications
- 앱 내에서 알림 기능을 사용
- badges, alert, etc

권한 요청
- `AppDelegate` 내에서 요청한다
- 사용자가 설정에서 권한을 바꿀 수 있으니 항상 현재의 권한을 확인하고 이를 바탕으로 구현해야 한다
- options: array 형태로 여려개 전달 가능
```
UNUserNotificationCenter.current().requestAuthorization(options: [UNAuthorizationOptions.badge, .sound, .alert]) { (granted, error) in
    if granted {
        UNUserNotificationCenter.current().delegate = self
    }
}
```

Notification 등록
- content 만들기
- trigger 설정
- request 보내기
```
@IBAction func schedule(_ sender: Any) {
    let content = UNMutableNotificationContent()
    content.title = "Hello"
    content.body = inputField.text ?? "Empty body"
    content.badge = 123
    content.sound = UNNotificationSound.default()
        
    let trigger = UNTimeIntervalNotificationTrigger(timeInterval: interval, repeats: false)
        
    // identifier는 알아보기 쉬운 문자열로 하는게 좋다
    let request = UNNotificationRequest(identifier: "test", content: content, trigger: trigger)
        
    UNUserNotificationCenter.current().add(request) { (error) in
        if let error = error {
            print(error)
        } else {
            print("done")
        }
    }
    inputField.text = nil
}
```

App이 Foreground에 있을때는 delegate를 실행한다
```
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
//        let content = notification.request.content
//        let trigger = notification.request.trigger
        
        // notification 종류 리턴
        completionHandler([UNNotificationPresentationOptions.alert])
    }
}
```
사용자가 notification badge를 터치했을 때
```
extension AppDelegate: UNUserNotificationCenterDelegate {

    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        let content = response.notification.request.content
        let trigger = response.notification.request.trigger
        
        completionHandler()
    }
}
```
badge 숫자 지우기
```
override func viewDidLoad() {
    super.viewDidLoad()
        
    UIApplication.shared.applicationIconBadgeNumber = 0
        
}
```

## Custom Notification Sound
- 무조건 로컬에 있어야한다
- Sound 폴더 안에 있어야 한다
```
let content = UNMutableNotificationContent()
content.title = "Hello"
content.body = "Custom Sound"
content.sound = UNNotificationSound(named: "bell.aif")
        
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
        
let request = UNNotificationRequest(identifier: "CustomSound", content: content, trigger: trigger)
        
UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
```

## Attachments
- image
- sound
- video

Image
```
let content = UNMutableNotificationContent()
content.title = "Hello"
content.body = "Image Attachment"
content.sound = UNNotificationSound.default()
        
guard let url = Bundle.main.url(forResource: "hello", withExtension: "png") else {
    return
}
        
// 이 옵션을 주면 미리보기에서 안보인다
var options = [UNNotificationAttachmentOptionsThumbnailHiddenKey: true]
        
guard let imageAttachment = try? UNNotificationAttachment(identifier: "hello-image", url: url, options: options) else {
    return
}
                
content.attachments = [imageAttachment]
        
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
let request = UNNotificationRequest(identifier: "Image Attachment", content: content, trigger: trigger)
UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
```
Sound
```
let content = UNMutableNotificationContent()
content.title = "Hello"
content.body = "Audio Attachment"
content.sound = UNNotificationSound.default()
        
guard let url = Bundle.main.url(forResource: "bell", withExtension: "aif") else {
    return
}
        
        
guard let audioAttachment = try? UNNotificationAttachment(identifier: "audio-attachment", url: url, options: nil) else {
    return
}
        
content.attachments = [audioAttachment]
        
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
let request = UNNotificationRequest(identifier: "Audio Attachment", content: content, trigger: trigger)
UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
```
Video
```
let content = UNMutableNotificationContent()
content.title = "Hello"
content.body = "Video Attachment"
content.sound = UNNotificationSound.default()
        
guard let url = Bundle.main.url(forResource: "video", withExtension: "mp4") else {
    return
}
        
guard let videoAttachment = try? UNNotificationAttachment(identifier: "video", url: url, options: nil) else {
    return
}
        
content.attachments = [videoAttachment]
        
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
let request = UNNotificationRequest(identifier: "Video Attachment", content: content, trigger: trigger)
UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
```

## Actionable Notification
action종류 및 category 등록
```
func setupCategory() {
    let likeAction = UNNotificationAction(identifier: ActionIdentifier.like, title: "Like", options: [])
    let dislikeAction = UNNotificationAction(identifier: ActionIdentifier.dislike, title: "Dislike", options: [])
        
    let imagePostingCategory = UNNotificationCategory(identifier: CategoryIdentifier.imagePosting, actions: [likeAction, dislikeAction], intentIdentifiers: [], options: [])
        
    UNUserNotificationCenter.current().setNotificationCategories([imagePostingCategory])
}

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        
    UNUserNotificationCenter.current().requestAuthorization(options: [UNAuthorizationOptions.badge, .sound, .alert]) { (granted, error) in
        if granted {
            self.setupCategory()
            UNUserNotificationCenter.current().delegate = self
        }
    }
        
    return true
}

```
Action 눌렀을 때 동작 구현
- background에 있을 때 UI 업데이트 하면 안된다
- 미리 저장
```
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        
        switch response.actionIdentifier {
        case ActionIdentifier.like:
            print("like")
        case ActionIdentifier.dislike:
            print("dislike")
        default:
            print("default")
        }
        
        // background에 있을 때 UI 업데이트 하면 안된다
        // 미리 저장
        UserDefaults.standard.set(response.actionIdentifier, forKey: "usersel")
            
        completionHandler()
    }
}
```
이후 저장된 내용에 대한 처리
```
@objc func updateSelection() {
    switch UserDefaults.standard.string(forKey: "usersel") {
    case .some(ActionIdentifier.like):
        imageView.image = UIImage(named: "thumb-up")
    case .some(ActionIdentifier.dislike):
        imageView.image = UIImage(named: "thumb-down")
    default:
        imageView.image = UIImage(named: "question")
    }
}

override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    updateSelection()
}
```
앱이 열리자마자 반영되기 위해 미리 notification 등록해서 `updateSelection()` 처리
```
override func viewDidLoad() {
    super.viewDidLoad()
        
    NotificationCenter.default.addObserver(self, selector: #selector(updateSelection), name: NSNotification.Name.UIApplicationDidBecomeActive, object: nil)
        
}
```
Catetory option
- `.hiddenPreviewsShowTitle` 옵션 주면 무조건 미리보기 보인다
```
if #available(iOS 11.0, *) {
    let imagePostingCategory = UNNotificationCategory(identifier: CategoryIdentifier.imagePosting, actions: [likeAction, dislikeAction], intentIdentifiers: [], options: [.hiddenPreviewsShowTitle])
    UNUserNotificationCenter.current().setNotificationCategories([imagePostingCategory])
} else {
    let imagePostingCategory = UNNotificationCategory(identifier: CategoryIdentifier.imagePosting, actions: [likeAction, dislikeAction], intentIdentifiers: [], options: [])
    UNUserNotificationCenter.current().setNotificationCategories([imagePostingCategory])
}
```

Content에 Category 추가
```
content.categoryIdentifier = CategoryIdentifier.imagePosting
```

Custom dismiss option
```
var options = UNNotificationCategoryOptions.customDismissAction
        
let imagePostingCategory = UNNotificationCategory(identifier: CategoryIdentifier.imagePosting, actions: [likeAction, dislikeAction], intentIdentifiers: [], options: options)
UNUserNotificationCenter.current().setNotificationCategories([imagePostingCategory])
```

- extension에서
```
extension AppDelegate: UNUserNotificationCenterDelegate {
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        
        switch response.actionIdentifier {
        case ActionIdentifier.like:
            print("like")
        case ActionIdentifier.dislike:
            print("dislike")
        case UNNotificationDismissActionIdentifier:
            print("Custom Dismiss")
        
        // 배너를 눌러 앱을 실행하는것도 하나의 identifier다
        // default는 따로 action을 추가하지 않아도 쓸 수 있다
        case UNNotificationDefaultActionIdentifier:
            print("launch from noti")
        default:
            print("default")
        }
        
        UserDefaults.standard.set(response.actionIdentifier, forKey: "usersel")
            
        completionHandler()
    }
}
```
Action options
- `.authenticationRequired`
    - 잠금화면이 풀린 상태에서만 action 선택 가능
- `.destructive`
- `.foreground`
    - 앱이 실행된다

## 추가
- app 전체에 대한 싱글톤에 접근
```
UIApplication.shared
```
- Struct로 상수 미리 구현하면 가독성이 높다
```
struct ActionIdentifier {
    static let like = "ACTION_LIKE"
    static let dislike = "ACTION_DISLIKE"
    private init() {}
}

struct CategoryIdentifier {
    static let imagePosting = "CATEGORY_IMAGE_POSTING"
    private init() {}
}
```