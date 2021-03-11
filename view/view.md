# View & Window
## Attributes Inspector
- **User Interaction Enabled:** 사용자가 터치했을 때 인식
- **Multiple Touch:** 두 손가락 이상 터치
- **Alpha:** 투명도
- **Hidden:** 숨기기
- **Clips to Bounds:** 프레임 벗어난 subview 자르기
- **Opaque:** Alpha값이 1이면 설정, 1미만이면 해제
- **Clears Graphics Context:** view 그리기 전에 그래픽 초기화

## 추가
switch 초기화 할 때,
```
@IBAction func toggleClipToBounds(_ sender: UISwitch) {
        redView.clipsToBounds = sender.isOn
    }

override func viewDidLoad() {
        super.viewDidLoad()
        
        clippingSwitch.isOn = redView.clipsToBounds
    }
```