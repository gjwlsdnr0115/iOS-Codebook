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

## GCD

Concurrent Queue
- 주어진 task를 동시에 실행
- 앱에서 기본적으로 제공하는 concurrent queue는 global queue
- 직접 생성도 가능

Serial queue
- 들어온 순선대로 하나씩 실행
- Main queue

Quality of Service
- `userInteractive`
- `userInitiated`
- `utility`
- `background`

Basic Pattern
- autoreleasepool 따로 신경 쓸 필요 없다
- global안에 main 수행
```
DispatchQueue.global().async {
    var total = 0
    for num in 1...100 {
        total += num
        Thread.sleep(forTimeInterval: 0.1)
    }
            
    DispatchQueue.main.async {
        self.valueLabel.text = "\(total)"
    }
}
```
Sync
```
concurrentWorkQueue.sync {
    for _ in 0..<3 {
        print("Hello")
    }
    print("Point1")
}
print("Point2")
```
```
>>> Hello
>>> Hello
>>> Hello
>>> Point1
>>> Point2
```
Async
```
concurrentWorkQueue.async {
    for _ in 0..<3 {
        print("Hello")
    }
    print("Point1")
}
print("Point2")
```
```
>>> Point2
>>> Hello
>>> Hello
>>> Hello
>>> Point1
```
Delay
```
// 현재 시간에 3초 더하기
let delay = DispatchTime.now() + 3
        
concurrentWorkQueue.asyncAfter(deadline: delay) {
    print("Point1")
}
print("Point2")
```
Concurrent Iteration
```
var start = DispatchTime.now()
for index in 0..<20 {
    print(index, separator: " ", terminator: " ")
    Thread.sleep(forTimeInterval: 0.1)
}
var end = DispatchTime.now()
        
print("\nfor-in: ", Double(end.uptimeNanoseconds - start.uptimeNanoseconds) / 1000000000)
        
// iteration의 순서가 중요하지 않을때
// 더 빠르게 처리하고 싶을 때
        
start = DispatchTime.now()
DispatchQueue.concurrentPerform(iterations: 20) { (index) in
    print(index, separator: " ", terminator: " ")
    Thread.sleep(forTimeInterval: 0.1)
}
end = DispatchTime.now()
print("\nconcurrentPerform: ", Double(end.uptimeNanoseconds - start.uptimeNanoseconds) / 1000000000)
```

DispatchWorkItem
- Submit
```
currentWorkItem = DispatchWorkItem(block: {
    for num in 0..<100 {
        guard !self.currentWorkItem.isCancelled else { return }
        print(num, separator: " ", terminator: " ")
        Thread.sleep(forTimeInterval: 0.1)
    }
})
        
serialWorkQueue.async(execute: currentWorkItem)
        
currentWorkItem.notify(queue: serialWorkQueue, execute: {
    print("Done")
})
```
- Wait
    - dispatch item에서 dead lock이 발생하면 notify  실행 안된다
    - 이런 문제 해결할 때 사용
    - 전달된 시간만큼 기다리고 완료 안되면 time out
```
let result = currentWorkItem.wait(timeout: .now() + 3)
switch result {
case .timedOut:
    print("timedOut")
case .success:
    print("Success")
}
```
- Cancel
    - dispatchQueue는 취소 기능 따로 없다
    - workItem에 대한 참조 저장했다가 취소 메소드 직접 호출 해야한다
```
currentWorkItem.cancel()
```
DispatchSourceTimer
```
var timer: DispatchSourceTimer?
    
@IBAction func start(_ sender: Any) {
    // 중복 생성 방지
    if timer == nil {
        // label update하는거니까 메인
        timer = DispatchSource.makeTimerSource(flags: [], queue: DispatchQueue.main)
        timer?.schedule(deadline: .now(), repeating: 1)
        timer?.setEventHandler(handler: {
            self.timeLabel.text = self.formatter.string(from: Date())
        })
            
    }
    timer?.resume()
}
    
@IBAction func suspend(_ sender: Any) {
    timer?.suspend()
}
    
@IBAction func stop(_ sender: Any) {
    timer?.cancel()
    timer = nil  // 메모리에서 제거
}
    
    
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    timer?.resume()
}
    
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    stop(self)
}
```

Semaphore
- race condition 방지
```
value = 0
valueLabel.text = "\(value)"
        
let sem = DispatchSemaphore(value: 1)
        
workQueue.async(group: group) {
    for _ in 1...1000 {
        sem.wait()
        self.value += 1
        sem.signal()
    }
}
        
workQueue.async(group: group) {
    for _ in 1...1000 {
        sem.wait()
        self.value += 1
        sem.signal()
    }
}
        
workQueue.async(group: group) {
    for _ in 1...1000 {
        sem.wait()
        self.value += 1
        sem.signal()
    }
}
        
group.notify(queue: DispatchQueue.main) {
    self.valueLabel.text = "\(self.value)"
}
```
- Execution order을 설정할 수도 있다
```
value = 0
valueLabel.text = "\(value)"
        
let sem = DispatchSemaphore(value: 0)
        
workQueue.async {
    for _ in 1...100 {
        self.value += 1
        Thread.sleep(forTimeInterval: 0.1)
    }
            
    sem.signal()
}
        
DispatchQueue.main.async {
            
    // main에서 wait은 실제 상황에서는 안쓰는게 좋다
    sem.wait()
    self.valueLabel.text = "\(self.value)"
}
```

## Implementation Example
```
class ImageFilterViewController: UIViewController {
    
    @IBOutlet weak var listCollectionView: UICollectionView!
    
    let downloadQueue = DispatchQueue(label: "DownloadQueue", attributes: .concurrent)
    let downloadGroup = DispatchGroup()
    
    let filterQueue = DispatchQueue(label: "FilterQueue", attributes: .concurrent)
    
    var isCancelled = false
    
    @IBAction func start(_ sender: Any) {
        PhotoDataSource.shared.reset()
        listCollectionView.reloadData()
        
        isCancelled = false
        
        PhotoDataSource.shared.list.forEach { data in
            self.downloadQueue.async(group: self.downloadGroup, execute: {
                self.downloadAndResize(target: data)
            })
        }
        
        self.downloadGroup.notify(queue: DispatchQueue.main) {
            self.reloadCollectionView()
        }
        
        self.downloadGroup.notify(queue: self.filterQueue) {
            DispatchQueue.concurrentPerform(iterations: PhotoDataSource.shared.list.count) { (index) in
                let data = PhotoDataSource.shared.list[index]
                self.applyFilter(target: data)
                
                let targetIndexPath = IndexPath(item: index, section: 0)
                DispatchQueue.main.async {
                    self.reloadCollectionView(at: targetIndexPath)
                }
            }
        }
        
    }
    
    @IBAction func cancel(_ sender: Any) {
        isCancelled = true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        PhotoDataSource.shared.reset()
    }
}
.
.
.
```