# Text
## Label
Set alignment
- left, center, right, justified, natural
```
guard let alignment = NSTextAlignment(rawValue: Int) else {
    return
}
        
label.textAlignment = alignment
```
Change font size
- label.font는 읽기 전용
- font 새로 만들고 다시 할당해야 한다
```
let newFont = valueLabel.font.withSize(newSize)
label.font = newFont
```
Change numer of lines
```
label.numberOfLines = Int(sender.value)
```
Change linebreak mode
```
if let mode = NSLineBreakMode.init(rawValue: Int) {
    valueLabel.lineBreakMode = mode
}
```
Autoshrink
```
valueLabel.minimumScaleFactor = sender.isOn ? 0.5 : 0.0
valueLabel.adjustsFontSizeToFitWidth = sender.isOn
```

## TextField
- Clear Button
- Clear when editing begins
- 왼쪽/오른쪽에 Overlay View 추가 가능 - 보통 오른쪽에 Clear Button

TextField에서 입력 된 값 가져올 때는 guard 사용 - text가 없을 수도 있기 때문
```
guard let text = inputField.text, text.count > 0 else {
    return
}
valueLabel.text = text
```
Border Style
- None, Line, Bezel, RoundedRect
- RoundedRect는 높이 지정 없음
```
let index = sender.selectedSegmentIndex
let style = UITextField.BorderStyle(rawValue: index) ?? .roundedRect  // 기본은 roundedRect
        
inputField.borderStyle = style
```

Add button to TextField
```
let btn = UIButton(type: .custom)
btn.frame = CGRect(x: 0, y: 0, width: 20, height: 20)
btn.setImage(UIImage(named: "menu"), for: .normal)
btn.addTarget(self, action: #selector(showPredefinedValue), for: .touchUpInside)
        
inputField.leftView = btn
inputField.leftViewMode = .always
```

## 추가
- enum (열거형)인 객체들은 .으로 할당한다
    - ex)
    ```
    inputField.leftViewMode = .always
    ```
