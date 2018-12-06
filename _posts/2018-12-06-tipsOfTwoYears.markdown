---
layout: post
title: "上个工作的Tips小结"
date: 2018-12-06 22:16:00.000000000 +09:00
tags: 回梦游仙
---

来日本两年多了，一直是一个人开发，新的工作环境也一样，完全没人切磋，感觉进步龟速。。

但是路还是要往前走的，所以现在总结下之前记录的知识点。

### 代码写的 view 如何放入 xib
```swift
required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
    setupView()
    // fatalError("init(coder:) has not been implemented")
}
```
用代码初始化的 view 会自动提示让我们加这个方法，在这个方法里把抛出异常删了，写上所需代码即可。  


### UITableView原点是（0，0）的时候，透明的导航栏会自动调整位置
如果不是这个点就不会调整位置，个人认为透明的时候很帅，像全屏穿透。
（其实具体记不清了，如果最上面不是tableView的话是肯定不会自动调整的）


### 为啥 xib 设置 borderUIColor 无效
borderUIColor 随便搜都能搜到
在 extension 写 borderUIColor 的实现方法前 + `@objc` 关键字


### xib tableHeaderView  
需要代码 view.add xib View ，不然高度不对


### IGListKit
adapter 必须得用懒加载，不然 sectionController 里的方法数量和配置 cell 不走


### tableViewCell 上的view 添加手势是没用的
（除非重写吧）

### tableView 顶部空了30
开始以为
```swift
automaticallyAdjustsScrollViewInsets = false

orderTableView.contentInset = UIEdgeInsetsMake(0, 0, 0, 0)
```
都不是 是 `group` 问题 改成 `plain` 解决


### tableView 键盘不遮挡cell输入框问题
代码的话，重写getter setter
```swift
get {
    let tvc = UITableViewController(style: .plain)
    addChildViewController(tvc)
    tvc.view.frame = self.view.frame
    let _tableView = tvc.tableView!
    _tableView.delegate = self
    _tableView.dataSource = self
    _tableView.separatorStyle = .none
    _tableView.register(UINib(nibName: "TFTableViewCell", bundle: nil), forCellReuseIdentifier: "TFTableViewCell")
    return _tableView
}
set {

}
```
xib 的话，用containerView包个TableViewController，自带



### collectionView reloadData() 没动画
```swift
UIView.transition(with: self.collectionView, duration: 0.35, options: UIViewAnimationOptions.transitionCrossDissolve, animations: {

}, completion: nil)
```
里面加 `reloadData()` 即可。


### ATS
https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW68

Allow Arbitrary Loads he  Allow Arbitrary Loads in web


### Struct and Class
https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes

Choosing Between Structures and Classes


# 回顾
虽然知识点很少，但是感觉都是网上很少列举的（自己搜肯定能搜到），不是很水吧。

换了个新现场，俩月过去了，新人时间早已结束，虽然有点适应了节奏，但是日语还是个坎，继续加油吧！
