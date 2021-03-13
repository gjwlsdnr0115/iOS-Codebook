# System Controls
## Page Control

viewDidLoad에서 초기화
```
override func viewDidLoad() {
    super.viewDidLoad()
        
    pager.numberOfPages = list.count
    pager.currentPage = 0
        
    pager.pageIndicatorTintColor = UIColor.systemGray3
    pager.currentPageIndicatorTintColor = UIColor.systemRed
        
}


```
스크롤 되었을 때 반영
```
extension PageControlViewController: UIScrollViewDelegate {
    // 스크롤 되었을 때 실행
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        let width = scrollView.bounds.size.width
        let x = scrollView.contentOffset.x + (width / 2.0)
        
        let newPage = Int(x / width)
        if pager.currentPage != newPage {
            pager.currentPage = newPage
        }
    }
}
```

터치 했을 때 페이지 넘김
```
@IBAction func pageChanged(_ sender: UIPageControl) {
    let indexPath = IndexPath(item: sender.currentPage, section: 0)
    listCollectionView.scrollToItem(at: indexPath, at: .centeredHorizontally, animated: true)
}
```

## Slider
- value 가져올때는 IBOutlet에서
- 하나의 IBAction에 여러개 연결 가능
- **Continuous:** 움직이는 순간마다 valueChange action
- **Non-Continuous:** 터치 종료시 action

slider 값 가져오기
```
let value = redSlider.value
```
slider 값 바꾸기
```
redSlider.value = 1.0
redSlider.setValue(value: Float, animated: Bool) // 애니메이션 넣을 때
redSlider.minimumValue = 0.0
redSlider.maximumValue = 1.0
```

Custom Slider
```
let img = UIImage(systemName: "lightbulb")
        
slider.setThumbImage(img, for: .normal) // 모든 상태 설정해야함
slider.minimumTrackTintColor = UIColor.systemRed
slider.maximumTrackTintColor = UIColor.black
        
slider.thumbTintColor
slider.setMinimumTrackImage(image: UIImage?, for: UIControl.State)
slider.setMaximumTrackImage(image: UIImage?, for: UIControl.State)
```

## Segment Control
- **Momentary:** 눌러도 선택된 상태 유지 안됨

선택된 값 접근
```
sender.selectedSegmentIndex
```
title 바꾸기
```
alignmentControl.setTitle("title", forSegmentAt: 0)
```
segment 추가, 제거
```
segmentedControl.insertSegment(withTitle: title, at: segmentedControl.numberOfSegments, animated: true)

segmentedControl.removeSegment(at: index, animated: true)
```
## Switch
코드로 toggle 할 경우
```
@IBAction func toggle(_ sender: Any) {
    testSwitch.isOn.toggle() // action 메도스 호츨 안됨, 애니메이션 없음

    testSwitch.setOn(!testSwitch.isOn, animated: true) 
    stateChanged(testSwitch) //IBAction 호출
}
```
## Stepper
- **Autorepeat:** 길게 눌렀을 때 계속 반응
- **Continuous:** 손을 떼지 않아도 IBAction 계속 호출
- **Wrap:** 최대 혹은 최소 넘어갔을 때 순환

속성 바꾸기
```
valueStepper.autorepeat = sender.isOn
valueStepper.isContinuous = sender.isOn
valueStepper.wraps = sender.isOn
```
stepper 값 가져오기
```
let value = stepper.value
```

## 추가
- UIColor 만들 때는 CGFloat로 만든다
- `label.textAlignment.rawValue`
    - left = 0, center = 1, right = 2