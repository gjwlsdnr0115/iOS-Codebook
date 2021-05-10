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