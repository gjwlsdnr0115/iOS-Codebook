# Animation

기본
```
@IBAction func animate(_ sender: Any) {

    // 새로운 frame 설정
    var frame = redView.frame
    frame.origin = view.center
    frame.size = CGSize(width: 100, height: 100)
        
    UIView.animate(withDuration: 0.3) {
        
        // frame 적용 및 변경사항
        self.redView.frame = frame
        self.redView.alpha = 0.5
        self.redView.backgroundColor = UIColor.blue
    }
}
```

애니메이션 끝나고 바로 실행할 코드 있을 때
- completion handler
- 애니메이션 성공하면 true 리턴
- handler애 새로운 animation 실행 가능

```
@IBAction func animate(_ sender: Any) {
    var frame = redView.frame
    frame.origin = view.center
    frame.size = CGSize(width: 100, height: 100)

    // 애니메이션 실행 후 세번째 파라미터에 전달된 블록 실행
    // bool은 애니메이션 잘 실행됬으면 true
    UIView.animate(withDuration: 0.3, animations: {
        self.redView.frame = frame
        self.redView.alpha = 0.5
        self.redView.backgroundColor = UIColor.blue
    }) { (finished) in
        UIView.animate(withDuration: 0.3) {
            // 원래 상태로 돌아가기 - with animation
            self.reset(nil)
        }
    }
}
```

애니메이션 미리 정의 후 전달 가능
- 별도 option 전달 가능
```
@IBAction func animate(_ sender: Any) {
    let animations: () -> () = {
        var frame = self.redView.frame
        frame.origin = self.view.center
        frame.size = CGSize(width: 100, height: 100)
        self.redView.frame = frame
        self.redView.alpha = 0.5
        self.redView.backgroundColor = UIColor.blue
    }
        
    UIView.animate(withDuration: 1.0, delay: 0.0, options: [.curveLinear, .repeat, .autoreverse], animations: animations, completion: nil)
}
```

애니메이션 중단하기
```
@IBAction func stop(_ sender: Any) {
    redView.layer.removeAllAnimations()
    reset(nil)
}
```

Sprint Animation
- 튕겨지는 효과
- damping 0~1 - 진폭
- velocity 0~5 - 속도
```
@IBAction func animate(_ sender: Any) {
    let targetFrame = CGRect(x: view.center.x - 100, y: view.center.y - 100, width: 200, height: 200)
        
    // damping 0~1
    // velocity 0~5
    UIView.animate(withDuration: 3, delay: 0.0, usingSpringWithDamping: CGFloat(dampingSlider.value), initialSpringVelocity: CGFloat(velocitySlider.value), options: [], animations: {
        self.redView.frame = targetFrame
    }, completion: nil)
}
```

Keyframe Animation
- 연속으로 여러 animation 구현
- relative duration 0~1까지
- 시작점이랑 duration 합이 1이 되게 구현
```
@IBAction func animate(_ sender: Any) {
    UIView.animateKeyframes(withDuration: 1.0, delay: 0, options: [], animations: {
        UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.3, animations: {
            self.phase1()
        })
        UIView.addKeyframe(withRelativeStartTime: 0.3, relativeDuration: 0.6, animations: {
            self.phase2()
        })
        UIView.addKeyframe(withRelativeStartTime: 0.6, relativeDuration: 0.4, animations: {
            self.phase3()
        })
    }, completion: nil)
}
```

Autolayout Animation
- 애니메이션 적용후 유지시키고 싶으면 제약도 업데이트 해야한다
- 제약을 먼저 설정
- 그 후에 animate

```
@IBAction func animate(_ sender: Any) {
        
    // 애니메이션 시작 전에 업데이트
    self.widthConstraint.constant = 200
    self.heightConstraint.constant = 200
    redView.setNeedsUpdateConstraints()

    UIView.animate(withDuration: 0.3, animations: {
        self.view.layoutIfNeeded()
    })
}
```

## Property Animator
- iOS 10 이상부터는 이거 권장
- 중간에 add도 가능
- inactive, active, stop 세가지 state

Start
```
@IBAction func animate(_ sender: Any) {
        
    // handler에는 에니메이션 종료 위치 나타내는 열거형 전달
    animator = UIViewPropertyAnimator.runningPropertyAnimator(withDuration: 7, delay: 0, options: [], animations: {
        self.moveAndResize()  // 애니메이션 코드
    }, completion: { position in
        switch position {
        case .start:
            print("Start")
        case .end:
            print("End")
        case .current:
            print("Current")
        }
    })
}
```
or
```
@IBAction func animate(_ sender: Any) {
 
    animator = UIViewPropertyAnimator(duration: 7, curve: .linear, animations: {
        self.moveAndResize()
    })
        
    // handler는 따로 추가해야 한다
    animator?.addCompletion({ (position) in
        print("Done \(position)")
    })
}
```

Pause
```
@IBAction func pause(_ sender: Any) {
    animator?.pauseAnimation()
    print(animator?.fractionComplete)  // 애니메이션 남은 비율
}
```

Resume
```
@IBAction func resume(_ sender: Any) {
    animator?.startAnimation()
}
```

Stop
```
@IBAction func stop(_ sender: Any) {
    animator?.stopAnimation(false)  // true - inactive, false - stoped
    animator?.finishAnimation(at: .current) // inactive로 전환
}
```
Add
```
@IBAction func add(_ sender: Any) {
        
    // inactive, active 상태에서만 호출
    // stopped에서 호출하면 crash
    animator?.addAnimations({
        self.redView.backgroundColor = UIColor.blue
    }, delayFactor: 0)
}
```

Interactive Animation
- 이렇게 직접 애니메이션에 관여 가능 (slider)
```
@IBAction func sliderChanged(_ sender: UISlider) {
    // 실행된 애니메이션 비율
    animator?.fractionComplete = CGFloat(sender.value)
}
```

## Motion Effect
- 기울였을때 움직이는 효과

```
override func viewDidLoad() {
    super.viewDidLoad()
        
    // x축에 motion effect 추가
    // keypath: motion 적용할 대상의 keypath
    let x = UIInterpolatingMotionEffect(keyPath: "center.x", type: .tiltAlongHorizontalAxis)
    x.minimumRelativeValue = -100
    x.maximumRelativeValue = 100
        
    let y = UIInterpolatingMotionEffect(keyPath: "center.y", type: .tiltAlongVerticalAxis)
    y.minimumRelativeValue = -100
    y.maximumRelativeValue = 100
        
    let group = UIMotionEffectGroup()
    group.motionEffects = [x, y]
        
    targetImageView.addMotionEffect(group)
    }
```