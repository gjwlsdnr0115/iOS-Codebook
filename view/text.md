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