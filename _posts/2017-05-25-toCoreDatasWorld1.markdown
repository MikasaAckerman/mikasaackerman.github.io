---
layout: post
title: "推开CoreData的门：增删改查"
date: 2017-05-25 13:01:00.000000000 +09:00
tags: 回梦游仙
---

在忙完一个阶段之后，就可以准备忙下一个阶段了。——毒鸡汤段子手  
然而我最近不太忙，所以准备研究下接下来能用到的知识点。  
感觉下阶段要追加购物车的功能，自己想了很久，UserDefaults，看不到又占内存；FMDB，又要增大app体积。所以最终还是准备用CoreData来存储数据。其实是瞎扯，是自己想学习这个^_^

虽然说之前也用过CoreData，但是当时模仿公司前辈用的KVC取值赋值编码解码，而且自己封装方法把这些值取出来自己初始化模型，我个人感觉CoreData一个实体就是一个模型，可以直接用并不用初始化了。

打开从喵神和其他大神组织的[Objc中国](https://www.objccn.io/)那里买的[CoreData](https://www.objccn.io/products/core-data/)一书，开始入门！p.s.还有明杰老师多年前写的[Core Data入门](http://blog.csdn.net/q199109106q/article/details/8563438/)，可以总览一下~

然后，自己就撸了一个增删改查的demo，查询并在tableView上表现，增加一条和删除第一条数据，改变cell一个view颜色，这个超简单工程的demo也可以下载对着注释看一遍，可能你就可以简单使用咯~

## 话不多说，开始搞起

#### 建立工程

为何要说这个呢，通过看书，得知一些算是技巧的点吧。  
建立工程不要勾上CoreData，建立完毕可以自己手动添加，这样AppDelegate不会出现那些自带代码，可以把CoreData捏在手里随心所欲。

#### 创建Data Model

`Command + N` 选择CoreData里的DataModel文件，起名为Moody，确认后生成Moody.xcdatamodeld文件。

![](/assets/images/2017/CoreDataBase1.png)

然后`Add Entity`添加Mood实体，再添加可转换类型的colors属性（书上写的遵循 NSCoding 协议的数据类型都可以直接声明为可转换的属性）（偷懒把数组改成了单个颜色，但是属性名字没改，见谅）和日期类型date的属性，选择属性date右边再勾上Indexed去掉Optional（变成索引和变成必选吧，暂时我乱勾对既有的已完成的demo工程也没影响，后面可能才会见到，先按照书上的来了）。

![](/assets/images/2017/CoreDataBase2.png)

![](/assets/images/2017/CoreDataBase3.png)

#### 创建Mood实体的托管对象子类

```swift
import UIKit
import CoreData

final class Mood: NSManagedObject {
    @NSManaged var date: Date
    @NSManaged var colors: UIColor
    
}

// 支持这个自定义的协议，给默认排序赋值
extension Mood: Managed {
    static var defaultSortDescriptors: [NSSortDescriptor] {
        return [NSSortDescriptor(key: #keyPath(date), ascending: false)]
    }
}
```

> - @NSManaged 书上说是告诉编译器这些属性将由Core Data来实现。
> - 自己也不是很明白，去搜了下，大约是类似于OC里@dynamic标签（其实这个我也不知道是啥- -），不过重点来了，就是告诉runtime，它自己会创建，并存在吧，更细节还得慢慢掌握。


#### 关联CoreData与Mood类

如文章第一张图，右侧`Class` `Name`写上Mood。但是这个name好像是自动添加的，个人感觉最重要的是`Codegen`要改成人工/无，不然系统也会自动创建Mood类导致一直报错。

#### 设置CoreData栈

iOS10之后可以用`NSPersistentContainer`来设置一个一本的Core Data栈，自己不是很理解这个栈，但是学完之后感觉是可以用这个栈来获取上下文吧。因此书上会创建容器，从中我们可以获取将在整个app里都被使用的托管对象上下文。  
但是之前版本的iOS方法不同，没这个类，只能直接获取上下文。  
下面是代码——

```swift
import CoreData

// 这个类的作用是定义一些方法，可以在全局使用

// iOS10新方法 NSPersistentContainer
// 首先，我们创建并命名了一个持久化容器 (persistent container)。Core Data 使用这个名字来 查找对应的数据模型，所以它应该和你的 .xcdatamodeld bundle 的文件名一致。接下来，我们调用容器的 loadPersistentStores 方法来尝试打开底层的数据库文件。如果数据库文件还不存在，Core Data 会根据你在数据模型里定义的大纲 (schema) 来生成它。
// 因为持久化存储们 (在我们的例子里，以及大多数真实世界情况下，只会有一个存储) 是异步加载的，一旦一个存储被加载完成，我们的回调就会被执行。如果发生了一个错误，我们现在就直接让程序崩溃掉。在生产环境中，你可能需要采取不同的反应，比如迁移已有的存储到新的版本，或者作为最后的手段，删除并重新创建这个存储。
// 最后，我们调度回主队列，并用这个新的持久化容器作为参数，调用 createMoodyContainer 的完成处理函数。
@available(iOS 10.0, *)
func createMoodyContainer(completion: @escaping (NSPersistentContainer) -> ()) {
    let container = NSPersistentContainer.init(name: "Moody")
    container.loadPersistentStores { (_, error) in
        guard error == nil else {
            fatalError("Failed to load store: \(error)")
        }
        DispatchQueue.main.async {
            completion(container)
        }
    }
}

// 下面是对应iOS10之前的
let documentURL = URL.init(fileURLWithPath: NSSearchPathForDirectoriesInDomains(FileManager.SearchPathDirectory.documentDirectory, FileManager.SearchPathDomainMask.allDomainsMask, true).first!)

private let StoreURL = documentURL.appendingPathComponent("Moody.moody")

public func createMoodyMainContext() -> NSManagedObjectContext {
    
    let bundles = [Bundle(for: Mood.self)]
    guard let model = NSManagedObjectModel.mergedModel(from: bundles) else { fatalError("model not found") }
    let psc = NSPersistentStoreCoordinator(managedObjectModel: model)
    try! psc.addPersistentStore(ofType: NSSQLiteStoreType, configurationName: nil,
                                at: StoreURL, options: nil)
    let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
    context.persistentStoreCoordinator = psc
    return context
}
```

#### 获取请求

基础的应该是创建请求，给个排序和获取数量，就能查询到所有的数据了。

``` swift
let request = NSFetchRequest<Mood>(entityName: "Mood")
let sortDescriptor = NSSortDescriptor(key: "date", ascending: false)
request.sortDescriptors = [sortDescriptor]
request.fetchBatchSize = 20
```

但是作者毕竟是大神，自己封装一个协议类，让Mood模型来实现这个协议，通用类简化代码量。

```swift
import CoreData

// 自定义抓取请求协议，包含实体的名字和一个排序
protocol Managed: class, NSFetchRequestResult {
    static var entityName: String { get }
    static var defaultSortDescriptors: [NSSortDescriptor] { get }
}

extension Managed {
    // 给一个默认排序
    static var defaultSortDescriptors: [NSSortDescriptor] {
        return []
    }
    
    // 真正的请求
    static var sortedFetchRequest: NSFetchRequest<Self> {
        let request = NSFetchRequest<Self>(entityName: entityName)
        request.sortDescriptors = defaultSortDescriptors
        return request
    }
}

// 通过约束为NSManagedObject子类型的协议扩展来给静态的entityName添加一个默认实现
extension Managed where Self: NSManagedObject {
    static var entityName: String {
        if #available(iOS 10.0, *) {
            return entity().name!
        } else {
            // Fallback on earlier versions
            return "Mood" // 这样的话不是iOS10的话感觉不能通用了啊，求大神告知
        }
    }
}
```

#### 与Fetched Results Controller配合在ViewController里表现

关键点在于`NSFetchedResultsController`的`.sections[i].objects?[i]`，和tableView的indexPath，类似，这样就可以获取一个Mood。  
删除修改都可以随意操作，最后记得`save()`，不然只是在内存中表现了，并没有让上下文通知数据库改变数据哟。

```swift
import UIKit
import CoreData

class ViewController: UIViewController, UITableViewDataSource, UITableViewDelegate, NSFetchedResultsControllerDelegate {
    
    /// 上下文
    var managedObjectContext: NSManagedObjectContext! // 用来记录操作的，也就是app与数据库的信使吧
    /// 抓取结果控制器
    var frc: NSFetchedResultsController<Mood>! // 用来协调模型和视图的，与tableView or collectionView应该是好搭档

    /// demo的tableView
    @IBOutlet weak var mainTableView: UITableView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        // 给上下文赋值
        if #available(iOS 10.0, *) {
            createMoodyContainer { (container) in
                self.managedObjectContext = container.viewContext
                self.setupTableView()
            }
        } else {
            // Fallback on earlier versions
            managedObjectContext = createMoodyMainContext()
            self.setupTableView()
        }
    }

    // MARK: - 与抓取控制器相结合配置tableView
    fileprivate func setupTableView() {
        mainTableView.dataSource = self
        mainTableView.delegate = self
        mainTableView.register(UINib.init(nibName: "MoodTableViewCell", bundle: nil), forCellReuseIdentifier: "MoodCell")
        
        let request = Mood.sortedFetchRequest
        request.fetchBatchSize = 20
        request.returnsObjectsAsFaults = false
        frc = NSFetchedResultsController(fetchRequest: request,
                                         managedObjectContext: managedObjectContext,
                                         sectionNameKeyPath: nil, cacheName: nil)
        frc.delegate = self
        try! frc.performFetch()
        mainTableView.reloadData()
    }
    
    // MARK: - 增加数据
    @IBAction func plusData(_ sender: UIButton) {
        
        guard let mood = NSEntityDescription.insertNewObject(forEntityName: "Mood", into: managedObjectContext) as? Mood else {
            fatalError("Wrong object type")
        }
        mood.colors = UIColor.brown
        mood.date = Date.init()
        try! self.managedObjectContext.save()
    }
    
    // MARK: - 删除数据
    @IBAction func minusData(_ sender: UIButton) {
        if (frc.sections?[0].objects?.count)! > 0 {
            guard let mood = frc.sections?[0].objects?[0] as? Mood else {
                fatalError("Delete wrong object type")
            }
            mood.managedObjectContext?.delete(mood)
            try! self.managedObjectContext.save()
        }
    }
    
    // MARK: - UITableViewDelegate
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        // 用来交替改变颜色
        guard let mood = frc.sections?[indexPath.section].objects?[indexPath.row] as? Mood else {
            fatalError("modifyData wrong object type")
        }
        if mood.colors == UIColor.brown {
            mood.colors = UIColor.gray
        } else {
            mood.colors = UIColor.brown
        }
        try! self.managedObjectContext.save()
    }
    
    // MARK: - UITableViewDataSource
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        guard let section = frc.sections?[section] else {
            return 0
        }
        return section.numberOfObjects
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let object = frc.object(at: indexPath)
        
        let cell = tableView.dequeueReusableCell(withIdentifier: "MoodCell", for: indexPath) as! MoodTableViewCell
        
        cell.configure(for: object)
        
        return cell
    }
    
    // MARK: - NSFetchedResultsControllerDelegate
    // 在这里开始更新tableView
    func controllerWillChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        mainTableView.beginUpdates()
    }
    
    // 这些是书里封装在一个自定义类里的，用于tableViewController，但是自己学艺不精，最终摘抄出来修改下，放在了ViewController里。
    func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>, didChange anObject: Any, at indexPath: IndexPath?, for type: NSFetchedResultsChangeType, newIndexPath: IndexPath?) {
        switch type {
        case .insert:
            guard let indexPath = newIndexPath else { fatalError("Index path should be not nil") }
            mainTableView.insertRows(at: [indexPath], with: .fade)
        case .update:
            guard let indexPath = indexPath else { fatalError("Index path should be not nil") }
            let object = frc.object(at: indexPath)
            guard let cell = mainTableView.cellForRow(at: indexPath) as? MoodTableViewCell else { break }
            cell.configure(for: object)
            mainTableView.reloadRows(at: [indexPath], with: UITableViewRowAnimation.fade)
        case .move:
            guard let indexPath = indexPath else { fatalError("Index path should be not nil") }
            guard let newIndexPath = newIndexPath else { fatalError("New index path should be not nil") }
            mainTableView.deleteRows(at: [indexPath], with: .fade)
            mainTableView.insertRows(at: [newIndexPath], with: .fade)
        case .delete:
            guard let indexPath = indexPath else { fatalError("Index path should be not nil") }
            mainTableView.deleteRows(at: [indexPath], with: .fade)
        }
    }
    
    // 在这里结束更新tableView
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        mainTableView.endUpdates()
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}
```

## 工程结束，小结

看了大神的代码，感觉有不少收获呢~

#### guard

每次遇到！解包的时候总会心惊胆战怕崩溃，现在可以在各处多写几行`guard let else {fatal error}`。  
自己唯一用到的地方是获取当前tab.nav的第几个ViewController，感觉挺好用的，以后要多多使用。

#### 标签

代码里多出使用`static`、`private`、`fileprivate`、`fileprivate(set)`各种标签，而自己基本没用过，感觉差距有点大，自己感觉是态度，严谨差距。

#### extension

各种延展，就一个tableViewCell的配置，在主class里就写了俩属性，其他方法就在延展里了，虽然不知道为啥，感觉醉醉的，但是还是决定模仿一下，毕竟大神！  
这里没有这个cell的代码，需要的下载demo吧，里面有惊喜哟，对于喜欢dateFormatter的伙伴又是个学习机会，那里我没写注释~

#### 个人猜测

是不是作者是面向协议编程的大神呢，感觉各种协议，自己知道用协议的地方只有代理，以后可以学习作者协议使用的地方，这样以后面试也不会只能说出代理了，嘿嘿。

## 总结

由于非专业出身，基础薄弱，心里总觉得没底，但是内心还是想进步的。可能科班出身一天能进步一千米，而我，虽然不能进步那么多，但是就算只有一厘米，我也要进步！  
虽然下了决心，但是开始的时候真的是很懵，但是静下心来一行一行看，改，本来真有放弃之心的，后来感觉也过来了，给自己点个赞。

这次又了解了很多简称，以前也看过但是忘了现在记下来。  
OOP--面向对象编程(Object Oriented Programming)  
POP--面向过程编程(Process-oriented programming)  
面向协议编程是什么没查到，然后VOP，SOA，AOP……路漫漫啊！

最后，如果有错误或者指导请随意联系，评论邮件微博等等都行。  
最最后，欢迎发邮件或者支持喵大的书以及喵大和他们小伙伴们翻译的书哟！[地址点我](https://www.objccn.io/)  
最最最后，demo[地址点我](https://github.com/MikasaAckerman/CorePracticeDemo)，再附上一张demo丑图。

![](/assets/images/2017/CoreDataBase4.png)




