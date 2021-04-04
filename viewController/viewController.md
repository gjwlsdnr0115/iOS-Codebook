# View Controller
- root view는 view로 접근


## Create View Controller
- Storyboard에서 바로 연결 가능
- 코드로 구현
```
// 따로 nib 파일 만들었을 경우
let vc = CustomNibViewViewController(nibName: "CustomNibViewViewController", bundle: nil)
// navigation controller에 속한 경우
navigationController?.pushViewController(vc, animated: true)
```

## View Management

addSubview
```
let v = generateRandomView()
view.addSubview(v)  // 가장 위에 추가
```
insertSubview
```
let v = generateRandomView()
view.insertSubview(v, at: 0)  // 가장 뒤에 추가
```
remove top view
```
let topMostView = view.subviews.reversed().first
topMostView?.removeFromSuperview()
```
bring view to front
```
view.bringSubview(toFront: redView)
```
send view to back
```
view.sendSubview(toBack: redView)
```
switch views
```
guard let greenViewIndex = view.subviews.index(of: greenView) else { return }
guard let blueViewIndex = view.subviews.index(of: blueView) else { return }
view.exchangeSubview(at: greenViewIndex, withSubviewAt: blueViewIndex)
```
get index of view
```
guard let greenViewIndex = view.subviews.index(of: greenView) else { return }
```
move view under different root view
- 따로 원래 view에서 제가할 필요 없음
```
if let grayView = grayView {
    view.addSubview(grayView)
}
```

## Life Cycle
이 순서대로 실행된다
```
override func viewDidLoad() {
    super.viewDidLoad()
      
    print(className, #function)
}
   
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
      
    print(className, #function)
}
   
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
      
    print(className, #function)
}
   
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
      
    print(className, #function)
}
   
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)
      
    print(className, #function)
}
   
deinit {
      print(className, #function)
}
```

## Orientation & Rotation
특정 화면 상태로 제한하기
```
override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
    return [.landscapeLeft, .landscapeRight]
}
    
override var shouldAutorotate: Bool {
    return true
}
```
특정 화면 상태 권장
```
override var preferredInterfaceOrientationForPresentation: UIInterfaceOrientation {
    return .landscapeLeft
}
```
화면 상태 업데이트 시 동작 구현
- 회전 적용되기 직전에 호출
- 아이폰에만 구별 가능
```
override func willTransition(to newCollection: UITraitCollection, with coordinator: UIViewControllerTransitionCoordinator) {
    super.willTransition(to: newCollection, with: coordinator)
        
    print(newCollection.verticalSizeClass.description)
        
    switch newCollection.verticalSizeClass {
    case .regular:
        closeButton.backgroundColor = UIColor.red.withAlphaComponent(0.5)
    default:
        closeButton.backgroundColor = UIColor.green.withAlphaComponent(0.5)
    }
}
```
- root view 크기 업데이트 될때 호출
- 첫번째 파라미터로 새로운 사이즈 전달 된다
- coordinator로 애니메이션 구현
```    
override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
    super.viewWillTransition(to: size, with: coordinator)
        
    coordinator.animate(alongsideTransition: { (context) in
        if size.height > size.width {
            self.closeButton.backgroundColor = UIColor.red.withAlphaComponent(0.5)
        } else {
            self.closeButton.backgroundColor = UIColor.green.withAlphaComponent(0.5)
        }
    }, completion: nil)

}
```

## Container View
- Storyboard에서 연결 가능
- Storyboard에서 view만 만들고 코드로 넣는것도 가능

코드로 구현
- 컨테이너에 먼처 추가
- 뷰 계층에 추가
```
if let childVC = storyboard?.instantiateViewController(identifier: "BottomViewController") {
    addChildViewController(childVC)  // 컨테이너에는 추가되지만 실제로 화면에 보이지는 않는다
    childVC.view.frame = bottomContainerView.bounds
    bottomContainerView.addSubview(childVC.view)
}
```
제거하기
- 제거 순서는 추가와 반대
- view 계층에서 제거 -> 컨테이너에서 제거
```
view.removeFromSuperview()  // rootView를 뷰 계층에서 제거
removeFromParentViewController() // 컨테이너에서 제거
```
ParentView에서도 제거 가능
```
@objc func removeChild() {
    for vc in childViewControllers {
        vc.view.removeFromSuperview()
        vc.removeFromParentViewController()
    }
}
```
Call 되는 methods
- 추가/삭제 직전에 호출
- 추가할때는 parentVC 전달
- 삭제할때는 nil 전달
- 코드로 삭제할때는 호출되지 않는다 - 따로 직접 호출해야한다
```
override func willMove(toParentViewController parent: UIViewController?) {
    super.willMove(toParentViewController: parent)    
}
```
- 추가/삭제 후 호출
- storyboard에서 embed segue로 구현한게 아니면 호출되지 않는다
- 삭제될때 연속적으로 호출 될 수 있다

```    
override func didMove(toParentViewController parent: UIViewController?) {
    super.didMove(toParentViewController: parent)
}
```
- 그럴때는 직접 호출 코드를 넣어야 한다
```
if let childVC = storyboard?.instantiateViewController(identifier: "BottomViewController") {
    addChildViewController(childVC)
    childVC.didMove(toParentViewController: self)  // 요기
    childVC.view.frame = bottomContainerView.bounds
    bottomContainerView.addSubview(childVC.view)
}
```