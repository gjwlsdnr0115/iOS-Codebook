# Auto Layout

## Constant
```
class ConstantViewController: UIViewController {
    @IBOutlet weak var redView: UIView!
    @IBOutlet weak var heightConstraint: NSLayoutConstraint!
    @IBOutlet weak var widthConstraint: NSLayoutConstraint!
    
    @IBAction func updateFrame(_ sender: Any) {
        heightConstraint.constant = 100
        widthConstraint.constant = 100
    }
}
```

## Priority
- 동적으로 바꿀때는 storyboard에서 큰 priority를 999로 한다
```
class PriorityViewController: UIViewController {
    
    @IBOutlet weak var widthOne: NSLayoutConstraint!
    @IBOutlet weak var widthTwo: NSLayoutConstraint!
    
    @IBAction func togglePriority(_ sender: Any) {
        widthOne.priority = widthOne.priority.rawValue < 999 ? UILayoutPriority(rawValue: 999) : UILayoutPriority(rawValue: 800)
        widthTwo.priority = widthOne.priority.rawValue < 999 ? UILayoutPriority(rawValue: 999) : UILayoutPriority(rawValue: 800)
    }
}

```

## Margin
Root view의 최소 마진 무시하기
```
if #available(iOS 11.0, *) {
    viewRespectsSystemMinimumLayoutMargins = false
} else {
    // Fallback on earlier versions
}
```

## Layout Guide
- iOS 11 이후
    - Safe Area Guide 사용
- iOS 11 이전
    - top, bottom layout guides 사용
```
override func viewDidLoad() {
    super.viewDidLoad()
        
    redView.translatesAutoresizingMaskIntoConstraints = false
        
    if #available(iOS 11.0, *)  {
        redView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor).isActive = true
        redView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor).isActive = true
        redView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor).isActive = true
        redView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor).isActive = true
    } else {
        redView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
        redView.topAnchor.constraint(equalTo: topLayoutGuide.bottomAnchor).isActive = true
        redView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
        redView.bottomAnchor.constraint(equalTo: bottomLayoutGuide.topAnchor).isActive = true
    }        
}
```