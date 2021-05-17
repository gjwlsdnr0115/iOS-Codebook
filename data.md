# Data Persistence

## File System
App Sandbox
- Bundle container
- Data container
    - 앱에서 데이터를 저장하면 data container에 저장
- iCloud container

## Data Container
- Documents: 사용자가 직접 생성한 데이터 저장
- Library: 사용자가 직접 생성하지 않은 데이터 중에서 영구적으로 저장되어야 하는 데이터
    - Caches
    - Application Support: 설정 파일과 같은 앱실행에 필요한 파일 저장
- tmp: 특정 작업을 수행할 때 사용하는 임시 파일 저장

- Documents, Caches를 제외한 Library의 파일들은 자동으로 백업 된다
- 이때, 사용자가 직접 생성하지 않은 파일, 다운로드한 파일, 다시 생성 가능한 파일, 캐시 파일은 반드시 백업에서 제외 되어야 한다
- 안그러면 Reject!

특정 directory에 있는 파일 혹은 하위 디렉토리 가져오기
```
let properties: [URLResourceKey] = [.localizedNameKey, .isDirectoryKey, .fileSizeKey, .isExcludedFromBackupKey]
            
// default는 공유 인스턴스
let currentContentUrls = try FileManager.default.contentsOfDirectory(at: url, includingPropertiesForKeys: properties, options: .skipsHiddenFiles)

```
하나의 url에서 속성/정보 가져오기
```
guard let url = currentDirectoryUrl  else {
    navigationItem.title = "???"
    return
}
do {
    let values = try url.resourceValues(forKeys: [.localizedNameKey])
    navigationItem.title = values.localizedName
} catch {
    print(error)
}
```
URL 접근 가능한지 확인하기
```
do  {
    let reachable = try url.checkResourceIsReachable()
    if !reachable {
        return
    }
} catch {
    print(error)
    return
}
```

홈 디렉토리 가져오기
```
currentDirectoryUrl = URL(fileURLWithPath: NSHomeDirectory())
```

Directory 추가
```
guard let url = currentDirectoryUrl?.appendingPathComponent(named, isDirectory: true) else { return }
        
do {
    try FileManager.default.createDirectory(at: url, withIntermediateDirectories: true, attributes: nil)
} catch {
    print(error)
}
```
Text file 추가
- Bundle에 위치한 파일 추가
```
guard let sourceUrl = Bundle.main.url(forResource: "lorem", withExtension: "txt") else {
    return
}
guard let targetUrl = currentDirectoryUrl?.appendingPathComponent("lorem").appendingPathExtension("txt") else {
    return
}
        
do {
    let data = try Data(contentsOf: sourceUrl)
    try data.write(to: targetUrl)
            
} catch {
    print(error)
}
```
사진 파일 추가
- Bundle에 위치한 파일
```
guard let sourceUrl = Bundle.main.url(forResource: "fireworks", withExtension: "png") else {
    return
}
        
guard let targetUrl = currentDirectoryUrl?.appendingPathComponent("fireworks").appendingPathExtension("png") else {
    return
}
        
do {
    let data = try Data(contentsOf: sourceUrl)
    try data.write(to: targetUrl)
} catch {
    print(error)
}
```
파일 확장자 가져오기
- 파일 이름과 확장자 리턴
- NSString은 파일 경로 관련해서 다양한 api 제공
```
let ext = (content.url.lastPathComponent as NSString).pathExtension.lowercased()
```
Url로부터 텍스트 파일의 데이터 가져오기
- 데이터 객체가 아닌 바로 String으로 가져온다
```
if let url = url {
    textView.text = try? String(contentsOf: url)
}
```
Url로부터 이미지 파일의 데이터 가져오기
- 데이터 객체로 만들어서 이미지로 변환
```
if let url = url, let data = try? Data(contentsOf: url) {
    imageView.image = UIImage(data: data)
}
```
Delete directory or file
```
do {
    try FileManager.default.removeItem(at: url)
} catch {
    print(error)
}
```
Rename
```
func renameItem(at url: URL) {
    let name = "newname"
    let ext = (url.lastPathComponent as NSString).pathExtension
        
    let newUrl = url.deletingLastPathComponent().appendingPathComponent(name).appendingPathExtension(ext)
        
    do {
        try FileManager.default.moveItem(at: url, to: newUrl)
    } catch {
        print(error)
    }
}
```

Update Backup Property
- `url`은 상수이기 때문에 `targetUrl`에 따로 저장해서 변경
```
func updateBackupProperty(of url: URL, exclude: Bool) {
    do {
        var targetUrl = url
        var values = try targetUrl.resourceValues(forKeys: [.isExcludedFromBackupKey])
        values.isExcludedFromBackup = exclude
            
        try targetUrl.setResourceValues(values)
    } catch {
        print(error)
    }
        
}
```
파일작업 Background queue에서 작업하기
```
func deleteFile(at url: URL) {
        
    DispatchQueue.global().async { [weak self] in
        do {
            let manager = FileManager()
            try manager.removeItem(at: url)
                
            DispatchQueue.main.async {
                // UI update
                self?.refreshContents()
            }
        } catch {
            print(error)
        }
    }
        
}
```

## User Defaults
- 사용자 설정과 같은 간단한 데이터 저장
- 한번 설정되면 영구적으로 유지 된다

Save data
```
let key = "sampleKey"
UserDefaults.standard.set("Hello", forKey: key)
```
Load data
- 원하는 형식에 따라 메소드 써서 가져온다
```
valueLabel.text = "\(UserDefaults.standard.integer(forKey: thresholdKey))"
valueLabel.text = UserDefaults.standard.string(forKey: key) ?? "Not Set"
```
초기 설정값 구현
```
let thresholdKey = "thresholdKey"
let initialLaunchKey = "initialLaunchKey"

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    var window: UIWindow?
    
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        if !UserDefaults.standard.bool(forKey: initialLaunchKey) {
            
            let defaultSettings = [thresholdKey: 123] as [String: Any]
            UserDefaults.standard.register(defaults: defaultSettings)
            
            UserDefaults.standard.set(true, forKey: initialLaunchKey)
            print("initial laucnh")
        }
        
        return true
    }
}
```
Remove data
- 둘중에 하나 사용
```
UserDefaults.standard.set(nil, forKey: key)
UserDefaults.standard.removeObject(forKey: key)
```

Check user defaults
```
print(UserDefaults.standard.dictionaryRepresentation())  // 전체 데이터
print(UserDefaults.standard.dictionaryWithValues(forKeys: [key]))  // 특정 key에 대한 데이터
```

데이터 변경되었을 때 notification
```
var token: NSObjectProtocol?
    
deinit {
    if let token = token {
        NotificationCenter.default.removeObserver(token)
    }
}


override func viewDidLoad() {
    super.viewDidLoad()
        
    token = NotificationCenter.default.addObserver(forName: UserDefaults.didChangeNotification, object: nil, queue: OperationQueue.main, using: { [weak self] (noti) in
        self?.updateDateLabel()
    })      
}
```

## Property List

Load from Bundle
```
guard let url = Bundle.main.url(forResource: "data", withExtension: "plist") else {
    fatalError("not found")
}
        
if let dict = try? NSDictionary(contentsOf: url, error: ()) {
    print(dict)
}
```
Save to documents
```
@IBAction func saveToDocuments(_ sender: Any) {
    do {
        let dict = ["language": "Swift", "os": "iOS"]
                        
        let encoder = PropertyListEncoder()
        let data = try encoder.encode(dict)
            
        try data.write(to: fileUrl)
        print("Done")
            
    } catch {
        print(error)
    }
}
```

Load from documents
```
let fileUrl: URL = {
    let documentsDirectory = FileManager().urls(for: .documentDirectory, in: .userDomainMask).first!
    return documentsDirectory.appendingPathComponent("data").appendingPathExtension("plist")
}()
    
@IBAction func loadFromDocuments(_ sender: Any) {
    do {
        let data = try Data(contentsOf: fileUrl)
        let decoder = PropertyListDecoder()
        let dict = try decoder.decode([String: String].self, from: data)

        print(dict)
    } catch {
        print(error)
    }
}
```
Save using custom structure
```
struct Development: Codable {
    let language: String
    let os: String
}

```
```
@IBAction func loadFromDocuments(_ sender: Any) {
        do {
            let data = try Data(contentsOf: fileUrl)
            let decoder = PropertyListDecoder()
            let dict = try decoder.decode(Development.self, from: data)

            print(dict)
        } catch {
            print(error)
        }
    }
    
    @IBAction func saveToDocuments(_ sender: Any) {
        do {
            
            let dev = Development(language: "Swift", os: "iOS")
            
            let encoder = PropertyListEncoder()
            let data = try encoder.encode(dev)
            
            try data.write(to: fileUrl)
            print("Done")
            
        } catch {
            print(error)
        }
    }
```

## NSCoding

데이터 Object 구조체 만들기
```
class Language: NSObject, NSCoding, NSSecureCoding {
    static var supportsSecureCoding: Bool {
        return true
    }
    
    func encode(with coder: NSCoder) {
        coder.encode(name as NSString, forKey: "name")
        coder.encode(version as NSNumber, forKey: "version")
        coder.encode(logo, forKey: "logo")
    }
    
    required init?(coder: NSCoder) {
        guard let name = coder.decodeObject(of: NSString.self, forKey: "name") else {
            return nil
        }
        self.name = name as String
        
        guard let version = coder.decodeObject(of: NSNumber.self, forKey: "version") else {
            return nil
        }
        
        
        self.version = version.doubleValue
        
        guard let img = coder.decodeObject(of: UIImage.self, forKey: "logo") else {
            return nil
        }
        self.logo = img

    }
    
    let name: String
    let version: Double
    let logo: UIImage
    
    init(name: String, version: Double, logo: UIImage) {
        self.name = name
        self.version = version
        self.logo = logo
    }
}
```

Encoding
```
do {
    guard let url = Bundle.main.url(forResource: "swiftlogo", withExtension: "png") else {
        return
    }
    let data = try Data(contentsOf: url)
            
    guard let img = UIImage(data: data) else {
        return
    }
            
    let obj = Language(name: "Swift", version: 5.0, logo: img)
            
            
    if #available(iOS 11.0, *) {
        let archivedData = try NSKeyedArchiver.archivedData(withRootObject: obj, requiringSecureCoding: true)
        try archivedData.write(to: fileUrl)
    } else {
        NSKeyedArchiver.archiveRootObject(obj, toFile: fileUrl.path)

    }
            
    print("done")
            
} catch {
    print(error)
}
```

Decoding
```
do {
            
    let data = try Data(contentsOf: fileUrl)
    var language: Language?
            
    if #available(iOS 11.0, *) {
        language = try NSKeyedUnarchiver.unarchivedObject(ofClass: Language.self, from: data)
    } else {
        language = NSKeyedUnarchiver.unarchiveObject(with: data) as? Language
    }
            
    if let language = language {
        self.imageView.image = language.logo
        self.nameLabel.text = language.name
        self.versionLabel.text = "\(language.version)"
    }
            
} catch {
    print(error)
}
```

## Codable

Encoding
```
do {
    guard let url = Bundle.main.url(forResource: "ioslogo", withExtension: "png") else {
        return
    }
            
    let data = try Data(contentsOf: url)
            
    let obj = CodableLanguage(name: "iOS", version: 12.0, logo: data)
            
    let archiver: NSKeyedArchiver
            
    if #available(iOS 11.0, *) {
        archiver = NSKeyedArchiver(requiringSecureCoding: false)
    } else {
        archiver = NSKeyedArchiver()
        archiver.requiresSecureCoding = false
    }
            
    try archiver.encodeEncodable(obj, forKey: NSKeyedArchiveRootObjectKey)
    try archiver.encodedData.write(to: fileUrl)
            
    archiver.finishEncoding()
    print("done")
            
            
} catch {
    print(error)
}
```

Decoding
```
do {
    let data = try Data(contentsOf: fileUrl)
            
    var language: CodableLanguage?
            
    let unarchiver: NSKeyedUnarchiver
            
    if #available(iOS 11.0, *) {
        unarchiver = try NSKeyedUnarchiver(forReadingFrom: data)
    } else {
        unarchiver = NSKeyedUnarchiver(forReadingWith: data)
    }
            
    unarchiver.requiresSecureCoding = false
            
    language = unarchiver.decodeDecodable(CodableLanguage.self, forKey: NSKeyedArchiveRootObjectKey)
            
    unarchiver.finishDecoding()
            
    if let language = language {
        self.imageView.image = UIImage(data: language.logo)
        self.nameLabel.text = language.name
        self.versionLabel.text = "\(language.version)"
    }
} catch {
    print(error)
}
```