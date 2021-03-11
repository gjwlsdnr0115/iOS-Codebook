# System View & Control
## Target-Action
- Button, Switch, Slider, Page Control, Date Picker, Segmented Control, Stepper
- UIControl을 상속 받는다
- **Action:** Event가 발생하면 호출되는 메소드
- **Target:** Action이 구현되어 있는 객체
- 코드에서 Target-Action 구현할 때
    ```
    func addTarget(Any?, action: Selector, for: UIControl.Event)
    ```

## 코드

```
class ViewController {
    @IBOutlet weak var btn: UIButton!

    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Selector로 변환
        let sel = #selector(action(_:))

        // Target이 ViewController이기 때문에 self로 전달
        btn.addTarget(self, action: sel, for: .touchUpInside)
    }

    // Objective-C 형태의 메소드로 작성
    @objc func action(_ sender: Any) {
        print(#function)
    }
}
```