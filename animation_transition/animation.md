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