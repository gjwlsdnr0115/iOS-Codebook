# Database


## CoreData
- Context를 이용해서 데이터를 다룬다
- Context에서 수정한다고 원본 데이터에 바로 반영되지 않는다
- 따로 업데이트 해야한다
- Read 할때도 Contex에 있는 데이터는 복사본이다. 수정해도 원본데이터는 그대로

## Basic Pattern
- 미리 .xcdatamodeld 파일에 entity 추가한다
Create
```
@IBAction func createEntity(_ sender: Any) {
        guard let name = nameField.text else {
        return
    }
    guard let val = ageField.text, let age = Int(val) else {
        return
    }
        
    let newEntity = NSEntityDescription.insertNewObject(forEntityName: "Person", into: context)
    newEntity.setValue(name, forKey: "name")
    newEntity.setValue(age, forKey: "age")
        
    if context.hasChanges {
        do {
            try context.save()
            print("saved")
        } catch {
            print(error)
        }
    }
        
    nameField.text = nil
    ageField.text = nil
}
```
Read
```
@IBAction func readEntity(_ sender: Any) {
    let request = NSFetchRequest<NSManagedObject>(entityName: "Person")
        
    do {
        let list = try context.fetch(request)
        if let first = list.first {
            nameField.text = first.value(forKey: "name") as? String
            if let age = first.value(forKey: "age") as? Int {
                ageField.text = "\(age)"
            }
                
            editTarget = first
                
        } else {
            print("not found")
        }
    } catch {
        print(error)
    }    
}
```
Update
```
var editTarget: NSManagedObject?
    
@IBAction func updateEntity(_ sender: Any) {
    guard let name = nameField.text else {
        return
    }
    guard let val = ageField.text, let age = Int(val) else {
        return
    }
        
    if let target = editTarget {
        target.setValue(name, forKey: "name")
        target.setValue(age, forKey: "age")
    }
        
    if context.hasChanges {
        do {
            try context.save()
            print("saved")
        } catch {
            print(error)
        }
    }
        
    nameField.text = nil
    ageField.text = nil    
}
```
Delete
```
@IBAction func deleteEntity(_ sender: Any) {
    if let target = editTarget {
        context.delete(target)
    }
        
    if context.hasChanges {
        do {
            try context.save()
            print("saved")
        } catch {
            print(error)
        }
    }
        
    nameField.text = nil
    ageField.text = nil
}
```

## Managed Object & Managed Object Context
- App delegate가 아니라 따로 Data manager 싱글톤 객체를 구현하는게 좋다
- 이전에 데이터모델 파일 생성하고 entity 설정 해야한다
- Editor -> create NSManagedObject subclass
```
import UIKit
import CoreData

class DataManager {
    static let shared = DataManager()
    
    private init() { }
    
    var container: NSPersistentContainer?
    
    var mainContext: NSManagedObjectContext {
        guard let context = container?.viewContext else {
            fatalError()
        }
        
        return context
    }
    
    
    func setup(modelName: String) {
        container = NSPersistentContainer(name: modelName)
        container?.loadPersistentStores(completionHandler: { desc, error in
            if let error = error {
                fatalError(error.localizedDescription)
            }
        })
    }
    
    
    func saveMainContext() {
        mainContext.perform {
            if self.mainContext.hasChanges {
                do {
                    try self.mainContext.save()
                } catch {
                    print(error)
                }
            }
        }
    }
}
```