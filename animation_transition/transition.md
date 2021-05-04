# Transition

## View Tranistion

- isHidden 속성을 이용하여 transition하는 방식
```
@IBAction func startTransition(_ sender: Any) {
        
    // isHidden 속성을 이용하여 transition하는 방식
    UIView.transition(with: containerView, duration: 1, options: [.transitionFlipFromLeft], animations: {
        self.redView.isHidden = !self.redView.isHidden
        self.blueView.isHidden = !self.blueView.isHidden
    }, completion: nil)
}
```
or
- 뷰 계층에 추가/삭제 방식
    - 뷰 계층에서 삭제하기 때문에 weak 참조인 redView를 handler 내부에서 바로 쓸 수 없음 - 메모리에서 삭제되기 때문
```
@IBAction func startTransition(_ sender: Any) {
    // 뷰 계층에 추가/삭제 방식
    UIView.transition(from: redView, to: blueView, duration: 1, options: [.transitionFlipFromLeft], completion: nil)
}
```
## Presentation

기본
```
@IBAction func present(_ sender: Any) {
    guard let modalVC = storyboard?.instantiateViewController(identifier: "ModalViewController") else {
        return
    }  
    present(modalVC, animated: true, completion: nil)
}
```

Transition Stytle 바꾸기
- segue로 구현
```
@IBAction func styleChanged(_ sender: UISegmentedControl) {
    selectedTransitionStyle = UIModalTransitionStyle(rawValue: sender.selectedSegmentIndex) ?? .coverVertical
}
    
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {    
    let presentVC = segue.destination
        
    presentVC.modalTransitionStyle = selectedTransitionStyle
}
```
- 코드로 구현
```
@IBAction func styleChanged(_ sender: UISegmentedControl) {
    selectedTransitionStyle = UIModalTransitionStyle(rawValue: sender.selectedSegmentIndex) ?? .coverVertical
}

@IBAction func present(_ sender: Any) {
    let sb = UIStoryboard(name: "Presentation", bundle: nil)
    let modalVC = sb.instantiateViewController(withIdentifier: "ModalViewController")
        
    modalVC.modalTransitionStyle = selectedTransitionStyle
    present(modalVC, animated: true, completion: nil)
}
```
Present Style
```
 @IBAction func present(_ sender: UIButton) {
    let sb = UIStoryboard(name: "Presentation", bundle: nil)
    let modalVC = sb.instantiateViewController(withIdentifier: "ModalViewController")
        
    let style = UIModalPresentationStyle(rawValue: sender.tag) ?? .fullScreen
    modalVC.modalPresentationStyle = style
        
        
    // ipad 일때
    if let pc = modalVC.popoverPresentationController {
        pc.sourceView = sender
            modalVC.preferredContentSize = CGSize(width: 500, height: 300)
    }
        
    printPresentationStyle(for: modalVC)
    present(modalVC, animated: true, completion: nil)
}
    
```

Presentation Context
- context에 따라 present 할때의 방식이 달라진다
- Naviation controller냐 아니면 지금 보고 있는 VC냐 등.
```
@IBAction func switchChanged(_ sender: UISwitch) {
    definesPresentationContext = sender.isOn
}
```

## Custom Presentation

커스텀 클래스 정의
```
class SimplePresentationController: UIPresentationController {
    let dimmingView = UIVisualEffectView(effect: UIBlurEffect(style: .dark))
    
    let closeButton = UIButton(type: .custom)
    var closeButtonTopConstraint: NSLayoutConstraint?
    
    
    let workaroundView = UIView()
    
    override init(presentedViewController: UIViewController, presenting presentingViewController: UIViewController?) {
        super.init(presentedViewController: presentedViewController, presenting: presentingViewController)
        
        closeButton.setImage(#imageLiteral(resourceName: "close"), for: .normal)
        closeButton.addTarget(self, action: #selector(dismiss), for: .touchUpInside)
        closeButton.translatesAutoresizingMaskIntoConstraints = false  // prototyping 제약 추가되지 않게
    }
    
    @objc func dismiss() {
        presentingViewController.dismiss(animated: true, completion: nil)
    }
    
    override var frameOfPresentedViewInContainerView: CGRect {
        print(String(describing: type(of: self)), #function)
        
        guard var frame = containerView?.frame else {
            return .zero
        }
        
        frame.origin.y = frame.size.height / 2
        frame.size.height = frame.size.height / 2
        
        return frame
    }
    
    // 여기서 애니메이션 구현
    override func presentationTransitionWillBegin() {
        print("\n\n")
        print(String(describing: type(of: self)), #function)
        
        guard let containerView = containerView else {
            fatalError()
        }
        
        dimmingView.alpha = 0.0
        dimmingView.frame = containerView.bounds
        containerView.insertSubview(dimmingView, at: 0)
        
        workaroundView.frame = dimmingView.bounds
        workaroundView.isUserInteractionEnabled = false
        containerView.insertSubview(workaroundView, aboveSubview: dimmingView)
        
        containerView.addSubview(closeButton)
        closeButton.centerXAnchor.constraint(equalTo: containerView.centerXAnchor).isActive = true
        closeButtonTopConstraint = closeButton.topAnchor.constraint(equalTo: containerView.topAnchor, constant: -80)
        closeButtonTopConstraint?.isActive = true
        
        containerView.layoutIfNeeded()  // 버튼이 화면 위쪽의 프레임으로 이동
        closeButtonTopConstraint?.constant = 60
        
        guard let coordinator = presentedViewController.transitionCoordinator else {
            dimmingView.alpha = 1.0
            presentingViewController.view.transform = CGAffineTransform(scaleX: 0.8, y: 0.8)
            containerView.layoutIfNeeded()
            return
        }
        
        coordinator.animate(alongsideTransition: { (context) in
            self.dimmingView.alpha = 1.0
            self.presentingViewController.view.transform = CGAffineTransform(scaleX: 0.8, y: 0.8)
            containerView.layoutIfNeeded()
        }, completion: nil)
    }
    
    override func presentationTransitionDidEnd(_ completed: Bool) {
        print(String(describing: type(of: self)), #function)

    }
    
    override func dismissalTransitionWillBegin() {
        print(String(describing: type(of: self)), #function)

        closeButtonTopConstraint?.constant = -80
        
        guard let coordinator = presentedViewController.transitionCoordinator else {
            dimmingView.alpha = 0.0
            presentingViewController.view.transform = CGAffineTransform.identity
            containerView?.layoutIfNeeded()
            return
        }
        
        coordinator.animate(alongsideTransition: { (context) in
            self.dimmingView.alpha = 0.0
            self.presentingViewController.view.transform = CGAffineTransform.identity
            self.containerView?.layoutIfNeeded()
            return
        }, completion: nil)
    }
    
    override func dismissalTransitionDidEnd(_ completed: Bool) {
        print(String(describing: type(of: self)), #function)
    }
    
    override func containerViewWillLayoutSubviews() {
        print(String(describing: type(of: self)), #function)
        presentedView?.frame = frameOfPresentedViewInContainerView
        dimmingView.frame = containerView!.bounds
    }
    
    override func containerViewDidLayoutSubviews() {
        print(String(describing: type(of: self)), #function)
    }
    
    override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
        super.viewWillTransition(to: size, with: coordinator)
        
        presentingViewController.view.transform = CGAffineTransform.identity
        
        coordinator.animate(alongsideTransition: { (context) in
            self.presentingViewController.view.transform = CGAffineTransform(scaleX: 0.8, y: 0.8)
        }, completion: nil)
    }
}

```

prepare에서 custom으로 설정
```
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    segue.destination.modalPresentationStyle = .custom
    segue.destination.transitioningDelegate = self
}
```
delegate에서 커스텀 클래스 리턴
```
extension CustomPresentationViewController: UIViewControllerTransitioningDelegate {
    func presentationController(forPresented presented: UIViewController, presenting: UIViewController?, source: UIViewController) -> UIPresentationController? {
        return SimplePresentationController(presentedViewController: presented, presenting: presenting)
    }
}

```


## 추가
- weak var은 참조하는 부분이 더이상 없으면 메모리에서 삭제 된다