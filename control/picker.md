# PickerView
## 개념
- Custom PickerView는 무조건 코드로만 구현
- delegate, datasource 메소드 구현해야 함
- **Component:** 종류 개수

## 코드
Delegate Methods
```
extension TextPickerViewController: UIPickerViewDataSource {
    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        return 2
    }
    
    func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        switch component {
        case 0:
            return comp1.count
        case 1:
            return comp2.count
        default:
            return 0
        }
    }

}

extension TextPickerViewController: UIPickerViewDelegate {
    func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        switch component {
        case 0:
            return comp1[row]
        case 1:
            return comp2[row]
        default:
            return nil
        }
    }
    func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        //선택 직후 바로 구현할 내용
        switch component {
        case 0:
            print(comp1[row])
        case 1:
            print(comp2[row])
        default:
            break
        }
    }
}

```
선택 직후 구현해야하는 내용이 아니라면 따로 메소드 구현
```
let row1 = pickerView.selectedRow(inComponent: 0)
let row1 = pickerView.selectedRow(inComponent: 1)
...
```
직접 picker 내용 선택
```
picker.selectRow(firstIdx, inComponent: 0, animated: true)
```
ImagePickerView 구현할때는 ImageView가 재사용 되도록 구현
```
func pickerView(_ pickerView: UIPickerView, viewForRow row: Int, forComponent component: Int, reusing view: UIView?) -> UIView {
        
    // 재사용 큐에 있는 이미지뷰 사용
    if let imageView = view as? UIImageView {
        imageView.image = images[row % images.count]
        return imageView
    }
        
    // 아직 재사용할 이미지뷰 없을 때
    let imageView = UIImageView()
    imageView.image = images[row % images.count]
    imageView.contentMode = .scaleAspectFit
    return imageView
}
```
pickerView의 데이터가 아직 구성되지 않았을 경우 대비
```
picker.reloadAllComponents()
```


## 추가
- switch문에서
    - break: switch에서만 나오고 이후 코드 수행
    - return: 전체 메소드 종료