# Content View
## Activity Indicator View
```
// start
if !loader.isAnimating {
    loader.startAnimating()
}

// stop
if loader.isAnimating {
    loader.stopAnimating()
}
```

## Progress View
- **Style:**
    - bar - tool bar / navigation bar
    - default

progress view 초기화
```
progress.progress = 0.0  // 애니메이션 없음 
```
progress view 값 설정
```
progress.setProgress(0.8, animated: true)
```

## Alert View
- Action Style
    - default
    - cancel - 두꺼운 폰트, 왼쪽 or 맨 밑
    - destructive - 빨간색, canel 바로 위
    - 나머지는 순서 대로 위/오른쪽 부터
- preferred action은 present하기 직전에 하는것이 좋다
- TextField는 .alert에서만 가능하다

action 추가
```
let controller = UIAlertController(title: "Hello", message: "Have a nice day", preferredStyle: .alert)
let okAction = UIAlertAction(title: "Ok", style: .default) { (action) in
    print(action.title)
}
controller.addAction(okAction)
        
let cancelAction = UIAlertAction(title: "Cancel", style: .cancel) { (action) in
    print(action.title)
}
controller.addAction(cancelAction)
        
let destructiveAction = UIAlertAction(title: "Destructive", style: .destructive) { (action) in
    print(action.title)
}
controller.addAction(destructiveAction)
        
controller.preferredAction = okAction
present(controller, animated: true, completion: nil)
```

TextField 추가
- textField에 입력한 텍스트에 대한 처리는 AlertAction의 handler에서 처리
```
let controller = UIAlertController(title: "Sign In", message: nil, preferredStyle: .alert)
        
controller.addTextField { (idField) in
    idField.placeholder = "AppleID"
}
controller.addTextField { (passwordField) in
    passwordField.placeholder = "Input password"
    passwordField.isSecureTextEntry = true
}
```
text field 접근
```
let fieldList = controller.textFields // 추가한 순서대로 배열
```

action disable - 회색으로 나오고 선택 안됨
```
action.isEnabled = false
```

## 추가

closure 내부에서 self를 사용하게 된다면 `[weak self]`를 명시하자 - strong reference cycle 방지
```
let newAction = UIAlertAction(title: "New Action", style: .default) { [weak self] (action) in
    self?.label.text = action.title
}
```