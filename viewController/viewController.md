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

## Navigation Controller
- Navigation bar에 버튼은 `UIBarButtonItem` - swich 같은 것도 항상 래핑하여 추가
- title과 버튼은 Navigation Item에 저장 - 모든 VC마다 하나씩 가지고 있다
- iOS 11.0 부터 Large title과 검색바 구현 가능
- back 버튼과 이전화면으로 돌아가는건 따로 구현할 필요 없다
- Navigation controller를 모달로 띄워서 별도로 관리할 수도 있다
    - 이렇게 할 경오 NC를 segue로 연결하고 present modally로 설정

코드로 구현
```
guard let rootVC = storyboard?.instantiateViewController(identifier: "FirstViewController") else {
    return
}
        
let nav = UINavigationController(rootViewController: rootVC)  // navigation controller 생성
present(nav, animated: true, completion: nil)  // present
```
NC에 새로운 vc push
- 모든 view controller는 `navigationController` 속성을 가지고 있음 - 만약 NC에 속한게 아니면 nil 리턴
```
guard let secondVC = storyboard?.instantiateViewController(identifier: "SecondViewController") else { return }
navigationController?.pushViewController(secondVC, animated: true)
```
pop하여 뒤로가기
```
navigationController?.popViewController(animated: true)
```
특정 vc로 pop 하기
- unwind segue
```
// 뒤로 가려는 view controller에서
// 그 뒤에 출발지 view controller애서 exit에 segue 연결
@IBAction func unwindToFirst(_ unwindSegue: UIStoryboardSegue) {
}
```
- 코드로 구현
```
navigationController?.popToRootViewController(animated: true)

guard let thirtVC = navigationController?.viewControllers.first(where: {$0 is ThirdViewController}) else { return }
navigationController?.popToViewController(thirtVC, animated: true)
```
바에 버튼 추가/제거
```
navigationItem.leftBarButtonItem = UIBarButtonItem(barButtonSystemItem: .add, target: nil, action: nil)

navigationItem.leftBarButtonItem = nil
```
이미 버튼이 있는경우 옆에 더 추가/제거
```
if var list = navigationItem.rightBarButtonItems {
    let btn = UIBarButtonItem(title: "Item", style: .plain, target: nil, action: nil)
    list.append(btn)
    navigationItem.rightBarButtonItems = list
}

            
let list = navigationItem.rightBarButtonItems?.dropLast()
navigationItem.rightBarButtonItems = Array(list!)
```
한번에 여려개 추가
```
let btn1 = UIBarButtonItem(barButtonSystemItem: .action, target: nil, action: nil)
let btn2 = UIBarButtonItem(title: "Two", style: .plain, target: nil, action: nil)
        
// 스위치 같은 것은 래핑을 해야한다
let sw = UISwitch()
let switchItem = UIBarButtonItem(customView: sw)
        

navigationItem.setRightBarButtonItems([switchItem, btn1, btn2], animated: true)
```
## Toolbar
- 툴바에 버튼 넣기
```
let flexibleSpaceItem = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: nil, action: nil)
let addItem = UIBarButtonItem(barButtonSystemItem: .add, target: nil, action: nil)
let trashItem = UIBarButtonItem(barButtonSystemItem: .trash, target: nil, action: nil)
        
setToolbarItems([flexibleSpaceItem, addItem, flexibleSpaceItem, trashItem, flexibleSpaceItem], animated: true)
```
hide toolbar
```
@IBAction func toggleToolbar(_ sender: Any) {
    let hidden = navigationController?.isToolbarHidden ?? false
    navigationController?.setToolbarHidden(!hidden, animated: true)
}
```

## Tab Bar Controller
- Navigation controller와 함께 사용하는 경우 nc를 tc에 embedd

tabBarItem 새로 할당
- 꼭 awakeFromNib에서 해야한다
```
override func awakeFromNib() {
    super.awakeFromNib()
    tabBarItem = UITabBarItem(tabBarSystemItem: .downloads, tag: 0)
}
```
badgeValue
```
tabBarItem.badgeValue = "1"
```
특정 tab 편집되지 않게 하기
- custom UITabBarController 만들고 스토리보드에서 할당
```
override func viewDidLoad() {
    super.viewDidLoad()

    customizableViewControllers = Array(customizableViewControllers!.dropFirst(2))
}
```

delegate
```
extension CustomTabBarController: UITabBarControllerDelegate {
    
    // 두번째 파라미터로 선택된 vc 전달
    // 코드로 선택할땐 적용 안된다
    func tabBarController(_ tabBarController: UITabBarController, shouldSelect viewController: UIViewController) -> Bool {
        // 두번째 vc인 경우 선택 제한
        if let secondVC = tabBarController.viewControllers?[1] {
            return viewController != secondVC
        }
        return true
    }
    
    // 선택 된 후 호출
    func tabBarController(_ tabBarController: UITabBarController, didSelect viewController: UIViewController) {
        
    }
    
}

```
코드로 tab bar controller 만들기
- tab bar item이 없다면 여기서 만들어줘야 한다
```
let firstVC = storyboard!.instantiateViewController(identifier: "FirstTabViewController")
let secondVC = storyboard!.instantiateViewController(identifier: "SecondTabViewController")
let thirdVC = storyboard!.instantiateViewController(identifier: "ThirdTabViewController")
        
let tbc = UITabBarController()
tbc.viewControllers = [firstVC, secondVC, thirdVC]
present(tbc, animated: true, completion: nil)
```
코드로 custom tab bar item 설정
```
override func awakeFromNib() {
        super.awakeFromNib()
        let regularImage = #imageLiteral(resourceName: "calendar_regular")
        let compactImage = #imageLiteral(resourceName: "calendar_compact")
        
        let item = UITabBarItem(title: "Calendar", image: regularImage, selectedImage: compactImage)
        item.badgeColor = .black
        item.badgeValue = "7"
        tabBarItem = item
        
    }
```