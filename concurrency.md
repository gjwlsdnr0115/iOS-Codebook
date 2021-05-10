# Concurrency

## Run Loop Timer
생성
```
@IBAction func startTimer(_ sender: Any) {
    guard timer == nil else {
        return
    }
        
    timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true, block: { (timer) in
        guard timer.isValid else { return }
        self.updateTimer(timer)
    })  
}
```
```
@IBAction func startTimer(_ sender: Any) {
    guard timer == nil else {
        return
    }

    timer = Timer(timeInterval: 1, target: self, selector: #selector(timerFired(_:)), userInfo: nil, repeats: true)
    timer?.tolerance = 0.2
    RunLoop.current.add(timer!, forMode: .defaultRunLoopMode)
}
```
적절할 때 invalidate 해야 하다
```
@IBAction func stopTimer(_ sender: Any) {
    // 이걸 호출해야 run loop에서 제거 된다
    timer?.invalidate()
}
    
override func viewDidLoad() {
    super.viewDidLoad()    
    resetTimer()
}
    
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    startTimer(self)
}
    
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
        
    timer?.invalidate()
    timer = nil
        
}
```

## Operation Queue

Operation
- 하나의 작업을 나타내는 객체
- 실행 완료된 작업은 다시 실행 불가
- Ready - Executing - Finished or Cancelled
- 직접 실행보다는 Operation Queue에 추가하여 사용

Operation queue 생성
- Main queue
```
let queue = OperationQueue.main
```
- Background queue
```
let queue = OperationQueue()
```

Operation 생성
- 클래서 없이는 3가지 방법
```
autoreleasepool {
    queue.addOperation {
        for _ in 1..<100 {
            guard !self.isCancelled  else { return }
            print("Hi", separator: " ", terminator: " ")
            Thread.sleep(forTimeInterval: 0.3)
        }
    }
}
        
let op = BlockOperation {
    autoreleasepool {
        for _ in 1..<100 {
            guard !self.isCancelled  else { return }
            print("Bye", separator: " ", terminator: " ")
            Thread.sleep(forTimeInterval: 0.6)
        }
    }
}
        
queue.addOperation(op)
        
op.addExecutionBlock {
    autoreleasepool {
        for _ in 1..<100 {
            guard !self.isCancelled  else { return }
            print("Wow", separator: " ", terminator: " ")
            Thread.sleep(forTimeInterval: 0.5)
        }
    }
}
        
let op2 = CustomOperation(type: "Good")
        
queue.addOperation(op2)
```
- operation에 있는 내용이 실행 된 후 호출
```
op.completionBlock = {
    print("done")
}
```
Custom Operation Class
```
class CustomOperation: Operation {
    let type: String
    
    init(type: String) {
        self.type = type
    }
    
    override func main() {
        autoreleasepool {
            for _ in 1..<100 {
                guard !isCancelled else { return }
                print(type, separator: " ", terminator: " ")
                Thread.sleep(forTimeInterval: 0.9)
            }
        }
    }
}
```

Operation Cancel
- `isCancelled`만 바꿀 뿐 실제로 멈추는건 직접 구현 해야한다
```
@IBAction func cancelOperation(_ sender: Any) {
    isCancelled = true
    queue.cancelAllOperations()
}
```

## Dependencies
- operation 생성 -> 의존성 추가 -> 배열에 추가 -> queue에 추가

Queue와 배열 생성
```
let backgroundQueue = OperationQueue()
let mainQueue = OperationQueue.main
    
var uiOperations = [Operation]()
var backgroundOperations = [Operation]()
```

Operation 생성 및 배열에 추가
```
DispatchQueue.global().async {
    let reloadOp = ReloadOperation(collectionView: self.listCollectionView)
    self.uiOperations.append(reloadOp)
            
    for (index, data) in PhotoDataSource.shared.list.enumerated() {
        let downloadOP = DownloadOperation(target: data)
        reloadOp.addDependency(downloadOP)
        self.backgroundOperations.append(downloadOP)
            
        let filterOp = FilterOperation(target: data)
        filterOp.addDependency(reloadOp)
        self.backgroundOperations.append(filterOp)
                
        let reloadItemOp = ReloadOperation(collectionView: self.listCollectionView, indexPath: IndexPath(item: index, section: 0))
        reloadItemOp.addDependency(filterOp)
        self.uiOperations.append(reloadItemOp)
    }
            
    // queue에 operation 배열 추가
    self.backgroundQueue.addOperations(self.backgroundOperations, waitUntilFinished: false)
    self.mainQueue.addOperations(self.uiOperations, waitUntilFinished: false)
            
}
```