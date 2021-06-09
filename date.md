# Date
## Date Picker
- iOS13 부터 새로운 모습 지원
- 근데 왠만하면 Wheel 모드로 사용

버전 확인
```
if #available(iOS 13.4, *) {
    datePicker.preferredDatePickerStyle = .wheels
} else {
    // Fallback on earlier versions
}
```

Date Picker 세부 설정
```
datePicker.datePickerMode = .dateAndTime
datePicker.locale = Locale(identifier: "ko_kr")
datePicker.minuteInterval = 1  // 꼭 60의 약수
        
datePicker.date = Date()
datePicker.setDate(Date(), animated: true)  // animation 적용하고 싶을 때

datePicker.minimumDate = Date()
datePicker.maximumDate = Date()

datePicker.calendar = Calendar.current
datePicker.timeZone = TimeZone.current
```

Value 선택 되었을 때
```
@IBAction func dateChanged(_ sender: UIDatePicker) {
    print(sender.date)
}
```
CountDown Timer
```
remainingTime.text = "\(picker.countDownDuration)"
remainingSeconds = Int(picker.countDownDuration)
```
Timer 구현
```
@IBAction func start(_ sender: Any) {
    timeLabel.text = "\(picker.countDownDuration)"
    remainingSeconds = Int(picker.countDownDuration)
        
    Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { (timer) in
        self.remainingSeconds -= 1
        self.timeLabel.text = "\(self.remainingSeconds)"
            
        if self.remainingSeconds == 0 {
            timer.invalidate()  // timer 종료
            AudioServicesPlaySystemSound(1315)  // 시스템 사운드 내기
        }
    }
}
```