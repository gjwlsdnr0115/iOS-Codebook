# Color
- CGColor: 섬세한 컬러 처리 필요할 때 사용, color space, bit 직접 설정
- DCI-P3: srgb보다 25% 더 넓은 컬러 범위
- Assets에서 Color Set을 직접 설정할 수 있음

컬러 생성
```
 let color = UIColor(red: 29.0/255.0, green: 161.0/255.0, blue: 242.0/255.0, alpha: 1.0)

 // 주로 이거 사용
 let dciColor = UIColor(displayP3Red: 29.0/255.0, green: 161.0/255.0, blue: 242.0/255.0, alpha: 1.0)
```

RGB 컬러 추출
```
var r = CGFloat(0)
var g = CGFloat(0)
var b = CGFloat(0)
var a = CGFloat(0)
        
targetView.backgroundColor?.getRed(&r, green: &g, blue: &b, alpha: &a) // 포인터를 전달
```

pattern color
```
if let img = UIImage(named: "pattern") {
    let patternColor = UIColor(patternImage: img)
    view.backgroundColor = patternColor
            
}
```

Get Color Set
```
view.backgroundColor = UIColor(named: "PrimaryColor") ?? UIColor.white
```