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
## TextView
Add insets
```
textView.textContainerInset = UIEdgeInsets(top: 30, left: 0, bottom: 30, right: 0)
textView.scrollIndicatorInsets = textView.textContainerInset
```
Text select 했을 때 실행되는 delegate
```
func textViewDidChangeSelection(_ textView: UITextView) {
    let range = textView.selectedRange  //NSRange
    selectedRangeLabel.text = "\(range)"
}
```
코드로 select Text
```
let lastWord = "pariatur?"
        
if let text = textView.text as NSString? {
    let range = text.range(of: lastWord)
    textView.selectedRange = range
    textView.scrollRangeToVisible(range)  // 선택된 곳으로 스크롤 이동
}
```
TextView는 내용에서 Data detect가 가능하다
- storyboard에서도 설정 가능
```
textView.dataDetectorTypes = [.link, .address, .phoneNumber]

```

## Text Input Traits
- 기본적으로 아이폰의 설정에 따라 맞춰짐
- storyboard에서도 설정 가능
Capitalization
```
inputField.resignFirstResponder()  // 키보드 입력을 종료했다가 다시 시작해야 적용됨
let type = UITextAutocapitalizationType(rawValue: sender.selectedSegmentIndex) ?? .none
inputField.autocapitalizationType = type
inputField.becomeFirstResponder()
```

Autocorrection
```
inputField.resignFirstResponder()
let type = UITextAutocorrectionType(rawValue: sender.selectedSegmentIndex) ?? .default
inputField.autocorrectionType = type
inputField.becomeFirstResponder()
```
Spell Checking
```
let type = UITextSpellCheckingType(rawValue: Int) ?? .default
textView.spellCheckingType = type
```
Secure Text Entry - 붙여넣기만 가능
```
textField.isSecureTextEntry = true
```
Smart Punctuation
- iOS11 이상만 가능

## Keyboard
First Responder
- viewWillAppear에서 설정을 해준다
- viewWillDisappear에서 해제
```
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    inputField.becomeFirstResponder()
}
    
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
        
    if inputField.isFirstResponder {
        inputField.resignFirstResponder()
    }
        
    //view.endEditing(true)  // textField가 여러개인 경우 한번에 처리
}
```
Keyboard Type
```
let types: [UIKeyboardType] = [.default, .asciiCapable, .numbersAndPunctuation, .URL, .numberPad, .phonePad, .namePhonePad, .emailAddress, .decimalPad, .twitter, .webSearch, .asciiCapableNumberPad]
```
Appearance
```
let type = UIKeyboardAppearance(rawValue: sender.selectedSegmentIndex) ?? .default
inputField.keyboardAppearance = type
```
Return Key
- storyboard에서도 설정 가능
- return key 눌렀을 때 반응하려면 delegate 구현해야 함
```
// 첫번째 리턴 누르면 두번째 textField로 감
// 두번째 리턴 누르면 검색

extension ReturnKeyViewController: UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        switch textField {
        case firstInputField:
            secondInputField.becomeFirstResponder()
        case secondInputField:
            guard let keyword = secondInputField.text, keyword.count > 0 else {
                return true
            }
            let encodedKeyword = keyword.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? keyword
            let urlStr = "http://www.google.com/m/search?q=\(encodedKeyword)"
            guard let url = URL(string: urlStr) else {
                return true
            }
            
            if UIApplication.shared.canOpenURL(url) {
                UIApplication.shared.open(url, options: [:], completionHandler: nil)
            }
        default:
            break
        }
        return true
    }
}

```
Keyboard Notification
- viewDidLoad에서 notificationn 추가
- textView랑 scrollInset 모두 설정
```
override func viewDidLoad() {
    super.viewDidLoad()

    // noti에는 키보드(notification)에 대한 정보 저장
    NotificationCenter.default.addObserver(forName: UIResponder.keyboardWillShowNotification, object: nil, queue: .main) { (noti) in
            
        guard let userInfo = noti.userInfo else { return }
        // Any이기 때문에 casting 필수
        guard let frame = userInfo[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect else { return }
            
        var inset = self.textView.contentInset
        inset.bottom = frame.height
        self.textView.contentInset = inset
            
        self.textView.scrollIndicatorInsets = inset
    }
        
    NotificationCenter.default.addObserver(forName: UIResponder.keyboardWillHideNotification, object: nil, queue: .main) { (noti) in
            
        var inset = self.textView.contentInset
        inset.bottom = 8
        self.textView.contentInset = inset
        self.textView.scrollIndicatorInsets = inset
    }
        
}
```
## Delegate
### TextField
- Tap
- `textFieldShouldBeginEditing(_:)`
- Become First Responder
- Start Editing Session
- `textFieldDidBeginEditing(_:)`
- 사용자가 텍스트 입력/삭제
- `textField(_:shouldChangeCharactersIn:replacementString:)`
    - 2: 텍스트 대체할 범위, 3: 대체할 텍스트, 삭제일시 빈 문자열
    - bool return
- `textFieldShouldClear(_:)`
- `textFieldShouldReturn(_:)`
- 사용자 입력 종료
- `textFieldShouldEndEditing(_:)`
- Resign First Responder
- End Editing Session
- `textFieldDidEndEditing(_:)`

### TextView
- Tap
- `textViewShouldBeginEditing(_:)`
- Become First Responder
- Start Editing Session
- `textViewDidBeginEditing(_:)`
- 사용자가 텍스트 입력/삭제
- `textView(_:shouldChangeTextIn:replacementText:)`
    - 2: 텍스트 대체할 범위, 3: 대체할 텍스트, 삭제일시 빈 문자열
    - true를 리턴할 경우
    - `textViewDidChange(_:)`
- 사용자가 입력 포커스 이동, 특정 범위 선택
- `textViewDidChangeSelection(_:)`
- 사용자가 attachment랑 인터랙션
- `textView(_:shouldInteractWith:in:interaction:)`
- 사용자 입력 종료
- `textViewShouldEndEditing(_:)`
- Resign First Responder
- End Editing Session
- `textViewDidEndEditing(_:)`

리턴키 눌렀을 때 반응 delegate
```
func textFieldShouldReturn(_ textField: UITextField) -> Bool {
    switch textField {
    case nameField:
        ageField.becomeFirstResponder()
    case ageField:
        genderField.becomeFirstResponder()
    case genderField:
        emailField.becomeFirstResponder()
    case emailField:
        emailField.resignFirstResponder()
    default:
        break
    } 
    return true
}
```
문자 길이 제한하기
-  `textField(_:shouldChangeCharactersIn:replacementString:)` 안에서는 아직 결과 반영되지 않음
- 키보드에 눌렀을 때 바로 호출
- `return true` 해야 textField에 반영됨
- 붙여넣기 해도 호출 됨
```
func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
    
    // 아직 반영되기 전이기 때문에 직접 finalText 만든다
    let currentText = NSString(string: textField.text ?? "")
    let finalText = currentText.replacingCharacters(in: range, with: string)
        
    switch textField {
    case nameField:
        if finalText.count > 10 {
            return false
        }
    case ageField:
        if let _ = string.rangeOfCharacter(from: charSet) {
            return false
        }
        if let age = Int(finalText), !(1...100).contains(age) {
            return false
        }
    case genderField:
        if let _ = string.rangeOfCharacter(from: invalidGenderCharSet){
                return false
        }
            
        if finalText.count > 1 {
            return false
        }
    default:
        break
    }
        
    return true
}
```
입력 완료 했을 때 확인 후 입력 상태 유지
```
func textFieldShouldEndEditing(_ textField: UITextField) -> Bool {
        
    if textField == emailField {
        guard let email = textField.text, let _ = email.range(of: regex, options: .regularExpression) else {
            alert(message: "Invalid Email")
            return false  // 입력 유지
        }
    }
    return true  // 입력 종료
}
```

Character Set
```
lazy var charSet = CharacterSet(charactersIn: "0123456789").inverted
lazy var invalidGenderCharSet = CharacterSet(charactersIn: "MF").inverted
```
Regex 정의
```
let regex = "^([a-z0-9_\\.-]+)@([\\da-z\\.-]+)\\.([a-z\\.]{2,6})$"
```

## 추가
- enum (열거형)인 객체들은 .으로 할당한다
    - ex)
    ```
    inputField.leftViewMode = .always
    ```
- String, NSString
    - 가능하면 String 사용 - Swift native
    - 필요한 경우에만 NSSTring으로 cast - Objective-C code
- control, command 키 동시에 누르고 클릭하면 선언 코드로 넘어감
    - 거기서 사용하는 정보들의 type 항상 확인 필요