# Gesture Recognizer
- Discrete gesture
    - Possible
    - Ended
    - Recognized
    - or Failed
- Continuous gesture
    - Possible
    - Began
    - Changed
    - Ended
    - or Cancelled

## Tap Gesture
- tap recognizer는 연결된 뷰에 일어난 제스쳐만 인식한다
- @IBAction으로 연결
- Label과 같은 뷰에 연결하면 `User interaction enabled`를 설정해야 한다
- 꼭 `state`를 확인하고 코드를 구현해야 에러를 막을 수 있다
```
@IBAction func handleTap(_ sender: UITapGestureRecognizer) {
        
    if sender.state == .ended {
        count += 1
        countLabel.text = "\(count)"
    }        
}
```

Code로 구현
```
var count = 0
    
@IBOutlet weak var countLabel: UILabel!
    
var tapGesture: UITapGestureRecognizer?
    
@objc func handleTap(_ tap: UITapGestureRecognizer) {
    if tap.state == .ended {
        count += 1
        countLabel.text = "\(count)"
    }
}
    
override func viewDidLoad() {
    super.viewDidLoad()
        
    countLabel.text = "\(count)"
    tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap(_:)))
        
    tapGesture?.numberOfTapsRequired = 1
    tapGesture?.numberOfTouchesRequired = 1
        
    // countLabel에 tap gesture 추가
    countLabel.addGestureRecognizer(tapGesture!)
    countLabel.isUserInteractionEnabled = true
        
}
```

## Pinch Gesture
- scale은 누적이 되기 때문에 매번 1로 초기화 해야한다
```
@IBAction func handlePinch(_ sender: UIPinchGestureRecognizer) {
        
    guard let targetView = sender.view else {
        return
    }
        
    targetView.transform = targetView.transform.scaledBy(x: sender.scale, y: sender.scale)
    sender.scale = 1
        
    }
```
Reset transform
```
@IBAction func reset(_ sender: Any) {
    UIView.animate(withDuration: 0.3) {
        self.imageView.transform = CGAffineTransform.identity
    }
}
```

## Rotation Gesture
```
@IBAction func handleRotation(_ sender: UIRotationGestureRecognizer) {
    guard let targetView = sender.view else {
        return
    }
    targetView.transform = targetView.transform.rotated(by: sender.rotation)
    sender.rotation = 0
}
    
@IBAction func reset(_ sender: Any) {
    UIView.animate(withDuration: 0.3) {
        self.imageView.transform = CGAffineTransform.identity
    }
}
```

## Swipe Gesture
```
@IBAction func handleSwipe(_ sender: UISwipeGestureRecognizer) {
        
    if sender.state == .ended {
        dismiss(animated: true, completion: nil)
    }
}
```

## Pan Gesture
- recognizer가 속한 뷰 밖까지 이동해도 인식 된다
- 이동시키려는 뷰에다가 추가
```
@IBAction func handlePan(_ sender: UIPanGestureRecognizer) {
    guard let targetView = sender.view else {
        return
    }
        
    // view 속성을 주면 rootview에서 얼마나 이동했는지 알려준다
    let translation = sender.translation(in: view)
        
    targetView.center.x += translation.x
    targetView.center.y += translation.y
        
    // 거리 초기화
    sender.setTranslation(.zero, in: view)
        
}
```

## Long Press Gesture
- Continuous gesture이다
- 중간에 조금이라도 움직이면 changed state가 된다
- else가 아니라 아예 ended일때를 구현하는게 좋다
```
@IBAction func handleLongPress(_ sender: UILongPressGestureRecognizer) {
        
    if sender.state == .began {
        blurView.isHidden = true
    } else if sender.state == .ended {
        blurView.isHidden = false
    }
}
```

## Screed Edge Gesture
```
@IBAction func handleScreenEdge(_ sender: UIScreenEdgePanGestureRecognizer) {
    if sender.state == .ended {
        UIView.transition(with: containerView, duration: 1, options: [.transitionFlipFromRight], animations: {
            self.redView.isHidden = !self.redView.isHidden
            self.blueView.isHidden = !self.blueView.isHidden
        }, completion: nil)
    }
}
```
- 기본적으로 아이폰 자체가 edge gesture를 사용한다
- 우선 순위를 바꿀 수 있다
```
// navigation control 뒤로가기 비활성화
override func viewDidLoad() {
    super.viewDidLoad()
    
    navigationController?.interactivePopGestureRecognizer?.isEnabled = false
}

// 모든 엣지에서 우선순위 높이기
override var preferredScreenEdgesDeferringSystemGestures: UIRectEdge {
    return UIRectEdge.all
}
```

## Multiple Gestures
- 인식 순서 정하기
```
pinchGesture.require(toFail: rotationGesture)
```
- delegate로 하기
```
override func viewDidLoad() {
    super.viewDidLoad()
        
    pinchGesture.delegate = self
    rotationGesture.delegate = self
}
```
```
extension MultipleGesturesViewController: UIGestureRecognizerDelegate {
    func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRequireFailureOf otherGestureRecognizer: UIGestureRecognizer) -> Bool {
        if gestureRecognizer == pinchGesture && otherGestureRecognizer == rotationGesture {
            return true
        }
        
        return false
    }
}
```