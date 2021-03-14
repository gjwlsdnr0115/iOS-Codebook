# Image
## ImageView
- Intrinsic Content Size 사용 - 높이/넓이 제약 없으면 사진 크기로 지정
- highlighted 사진 따로 지정 가능
animate image view
```
imageView.animationImages = images
imageView.animationDuration = 1.0
imageView.animationRepeatCount = 3  // 기본값 0

imageView.startAnimating()
if imageView.isAnimating {
    imageView.stopAnimating()
}
```

## Image
- asset에서 불러올때는 `UIImage(named: String)`을 사용
- image literal은 충돌이 날 가능성이 있기 때문에 사용하지 않는게 좋다
- vector image 사용하면 깨지지 않고 깔끔하게 이미지 사용 가능
    - assets에서 `Preserve Vector Data`, `Scale - Single Scale` 설정
    - PDF vector - 범용적, svg - iOS 13 이후만 가능

get image size
```
// point size
if let ptSize = img1?.size {
    print("Image size: \(ptSize)")
}

// pixel size
if let ptSize = img1?.size, let scale = img1?.scale {
    let px = CGSize(width: ptSize.width * scale, height: ptSize.height * scale)
    print("Image size(px): \(px)")
}
```

image 바이너리 데이터로 만들기
```
let pngData = img1?.pngData()
let jpgData = img1?.jpegData(compressionQuality: 1.0)
```

Rendering mode
- assets에서 직접 바꿀 수도 있다
```
if let img = UIImage(named: "clover")?.withRenderingMode(.alwaysTemplate) {
    imageView.image = img
}
```

## 추가
- super view의 tint color 속성은 모든 subview에 상속된다

## Image Resizing

```
func resizingWithImageContext(image: UIImage, to size: CGSize) -> UIImage? {
    UIGraphicsBeginImageContextWithOptions(size, true, 0.0)
        
    let frame = CGRect(origin: CGPoint.zero, size: size)
    image.draw(in: frame)
        
    let resultImage = UIGraphicsGetImageFromCurrentImageContext()
        
    UIGraphicsEndImageContext()
    return resultImage
    }
```

bitmap속성 함께 설정하는 경우
```
func resizingWithBitmapContext(image: UIImage, to size: CGSize) -> UIImage? {
    guard let cgImage = image.cgImage else {
        return nil
    }
        
    let bpc = cgImage.bitsPerComponent
    let bpr = cgImage.bytesPerRow
    let colorSpace = cgImage.colorSpace!
    let bitmapInfo = cgImage.bitmapInfo
        
    guard let context = CGContext(data: nil, width: Int(size.width), height: Int(size.height), bitsPerComponent: bpc, bytesPerRow: bpr, space: colorSpace, bitmapInfo: bitmapInfo.rawValue) else {
        return nil
    }
        
    context.interpolationQuality = .high
        
    let frame = CGRect(origin: .zero, size: size)
    context.draw(cgImage, in: frame)
        
    guard let resultImage = context.makeImage() else {
        return nil
    }
        
    return UIImage(cgImage: resultImage)
}
```