<div style="text-align: center; width: 100%">
<img src="https://github.com/1amageek/Pring/blob/master/Pring.png", width="100%">

 [![Version](http://img.shields.io/cocoapods/v/Pring.svg)](http://cocoapods.org/?q=Pring)
 [![Platform](http://img.shields.io/cocoapods/p/Pring.svg)](http://cocoapods.org/?q=Pring)
 [![Downloads](https://img.shields.io/cocoapods/dt/Pring.svg?label=Total%20Downloads&colorB=28B9FE)](https://cocoapods.org/pods/Pring)

</div>

[Please donate to continue development.](https://gum.co/lNNIn)

# Pring <β>
Firestore model framework.
The concept of Document and Collection has been added to Firestore. Pring defines the Scheme of the Document and enables type - safe programming. SubCollection can also be defined in Scheme.

<b> ⚠️ This code still contains bugs.</b><br>
Please see TODO for details.

Please report issues [here](https://github.com/1amageek/Pring/issues/new)

## Requirements ❗️
- iOS 10 or later
- Swift 4.0 or later
- [Firebase firestore](https://firebase.google.com/docs/firestore/quickstart)
- [Firebase storage](https://firebase.google.com/docs/storage/ios/start)
- [Cocoapods](https://github.com/CocoaPods/CocoaPods/milestone/32) 1.4 ❗️  ` gem install cocoapods --pre `

## Installation ⚙
#### [CocoaPods](https://github.com/cocoapods/cocoapods)

- Insert `pod 'Pring' ` to your Podfile.
- Run `pod install`.


## Feature 🎊

☑️ You can define Firestore's Document scheme.<br>
☑️ Of course type safety.<br>
☑️ It seamlessly works with Firestore and Storage.<br>
☑️ You can easily associate subcollections.<br>
☑️ Support GeoPoint.<br>

## TODO ✅

### 

### Implementation
- [x] Implement DataType that Firestore can handle
- [x] Implement data management
- [ ] Implement custom DataType (Specification under consideration)
- [x] Implement linkage with Firestorage
- [x] Implement the NestedCollection feature
- [x] Implement the ReferenceCollection feature
- [ ] Implement the NestedCollection feature of saved
- [ ] Implement the ReferenceCollection feature of saved
- [x] Implement DataSource
- [ ] Implement Query-enabled DataSource (Specification under consideration)

### Verification (Running Unit test)
- [x] Verify the implementation of DataType that Firestore can handle
- [x] Verify the implementation of data management
- [ ] Verify the implementation of custom DataType
- [x] Verify cooperation with Firestorage
- [x] Verify the implementation of the NestedCollection feature
- [ ] Verify the implementation of the ReferenceCollection feature
- [ ] Verify the implementation of Query-enabled DataSource

If you have a Feature Request, please post an [issue](https://github.com/1amageek/Pring/issues/new).

## Usage

### Model 

Pring inherits Object class and defines the Model. Pring supports many data types.

``` swift
@objcMembers
class MyObject: Object {
    dynamic var array: [String]                     = ["array"]
    dynamic var set: Set<String>                    = ["set"]
    dynamic var bool: Bool                          = true
    dynamic var binary: Data                        = "data".data(using: .utf8)!
    dynamic var file: File                          = File(data: UIImageJPEGRepresentation(UIImage(named: "")!, 1))
    dynamic var url: URL                            = URL(string: "https://firebase.google.com/")!
    dynamic var int: Int                            = Int.max
    dynamic var float: Double                       = Double.infinity
    dynamic var date: Date                          = Date(timeIntervalSince1970: 100)
    dynamic var geoPoint: GeoPoint                  = GeoPoint(latitude: 0, longitude: 0)
    dynamic var dictionary: [AnyHashable: Any]      = ["key": "value"]    
    dynamic var string: String                      = "string"
    
    let nestedCollection: NestedCollection<TestDocument>   　　　　　　　　　　= []
    let referenceCollection: ReferenceCollection<TestDocument>  = []
}
```

| DataType | Description |
|---|---|
|Array|It is Array type.|
|Set|It is Set type.In Firestore it is expressed as `{"value": true}`.|
|Bool|It is a boolean value.|
|File|It is File type. You can save large data files.|
|URL|It is URL type. It is saved as string in Firestore.|
|Int|It is Int type.|
|Float|It is Float type. In iOS, it will be a 64 bit Double type.|
|Date|It is Date type.|
|GeoPoint|It is GeoPoint type.|
|Dictionary|It is a Dictionary type. Save the structural data.|
|Relation|It is Relation type. Relation type. Holds the count stored in SubCollection.|
|String|It is String type.|
|Null|It is Null type.|

⚠️ `Bool` `Int` `Float` `Double` are not supported optional type. 


### ⚙️ Manage data

#### Save
Document can be saved only once.

``` swift
let object: MyObject = MyObject()
object.save { (ref, error) in
   // completion
}
```

#### Retrieve
Retrieve document with ID.

``` swift
MyObject.get(document!.id, block: { (document, error) in
    // do something
})
```

#### Update
Document has an update method.
Be careful as it is different from [Salada](https://github.com/1amageek/Salada).

``` swift
MyObject.get(document!.id, block: { (document, error) in
    document.string = "newString"
    document.update { error in
       // update
    }
})
```

#### Delete
Delete document with ID.

``` swift
MyObject.delete(id: document!.id)
```

### 📄 File
**Pring** has a File class because it seamlessly works with Firebase Storage.

#### Save
File is saved with Document Save at the same time.

``` swift
let object: MyObject = MyObject()
object.thumbnailImage = File(data: PNG_DATA, mimeType: .png)
let tasks: [String: StorageUploadTask] = object.save { (ref, error) in

}
```

`save` method returns the StorageUploadTask that is set with the key.
For details on how to use StorageUploadTask, refer to [Firebase docs](https://firebase.google.com/docs/storage/ios/upload-files?authuser=0).

``` swift
let task: StorageUploadTask = tasks["thumbnailImage"]
```

#### Get data
Get data with size.

``` swift
let task: StorageDownloadTask = object.thumbnail.getData(100000, block: { (data, error) in
    // do something
})
```

#### Update
If the Document is already saved, please use update method.
`update` method also returns StorageUploadTask.
Running update method automatically deletes old files.

``` swift
let newFile: File = File(data: PNG_DATA, mimeType: .png)
object.thumbnailImage = newFile
let task: StorageUploadTask = object.thumbnailImage.update { (metadata, error) in

}
```

#### Delete
Delete it with `delete` method.

``` swift
object.thumbnailImage.delete { (error) in

}
```

### Nested Collection & Reference Collection

`NestedCollection` and `ReferenceCollection` are classes that define SubCollection.

When holding `File` in SubCollection, saving of `File` will be executed first. When many `File`s are stored in SubCollection at once, the performance deteriorates.

#### Nested Collection
- NestedCollection nests data and saves it under the document.
- The destination path of File is nested path.

#### Reference Collection
- ReferenceCollection saves the documentID under the document.
- Data is saved separately.

``` swift
@objcMembers
class User: Object {
    dynamic var name: String?
    dynamic var referenceCollection: ReferenceCollection<User> = []
    dynamic var nestedCollection: NestedCollection<Item> = []
}

@objcMembers
class Item: Object {
    dynamic var thumbnail: File?
}

let userA: User = User()
userA.name = "userA"

let userB: User = User()
userB.name = "userB"

let item: Item = Item()
item.thumbnail = File(data: JPEG_DATA, mimeType: .jpeg)

userA.referenceCollection.insert(userB)
userA.nestedCollection.insert(item)
userA.save()
```

### DataSource

DataSource is a class for easy handling of data retrieval from Collection.
``` swift
override func viewDidLoad() {
    super.viewDidLoad()

    self.dataSource = DataSource(reference: User.reference) { [weak self] (changes) in
        guard let tableView: UITableView = self?.tableView else { return }

        switch changes {
        case .initial:
            tableView.reloadData()
        case .update(let deletions, let insertions, let modifications):
            tableView.beginUpdates()
            tableView.insertRows(at: insertions.map { IndexPath(row: $0, section: 0) }, with: .automatic)
            tableView.deleteRows(at: deletions.map { IndexPath(row: $0, section: 0) }, with: .automatic)
            tableView.reloadRows(at: modifications.map { IndexPath(row: $0, section: 0) }, with: .automatic)
            tableView.endUpdates()
        case .error(let error):
            print(error)
        }
    }
}

func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return self.dataSource?.count ?? 0
}
```
