# Button
## 개념
- **종류:** System, Detail Disclosure, info, Add Contact, Close(iOS 13+)
- State에 따라 다른 설정을 줄 수 있다 (Default, Selected, Highligted, Disabled)
- 코드에서 button.state는 get만 가능하다

## 코드
예시
```
btn.isEnabled = true
btn.isSelected = false
btn.isHighlighted = false
btn.isHighlighted.toggle()
btn.isSelected.toggle()
```
Button의 text, textColor는 바로 할당 못한다. 대신 `.setTitle()`을 이용
```
btn.setTitle("Hello", for: .normal)
btn.setTitle("Haha", for: .highlighted)
btn.setTitleColor(.systemRed, for: .normal)
```