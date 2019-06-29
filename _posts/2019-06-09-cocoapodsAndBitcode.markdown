---
layout: post
title: "使cocoapods打开bitcode"
date: 2019-06-09 16:16:00.000000000 +09:00
tags: 回梦游仙
---

### 问题

做了俩 Framework A 和 B，bitcode 均设置 yes；

A 就自己写了一些方法，做了个 APP 并且导入，APP 可以 archive ；

B 依赖于 cocoapods 也写了一些方法，也导入到刚刚的 APP，此时 APP 可以编译或者运行成功，但是不能 archive 了，错误如下。

![](/assets/images/2019/cocoapodsAndBitcode1.jpeg)

### 尝试

①百度 大部分都是让关闭 bitcode ，没参考价值。

②谷歌 有的说设置 bitcode 的一系列配置，AB 均相同配置，具体如下。

![](/assets/images/2019/cocoapodsAndBitcode2.jpeg)

还有说可以用终端 otool 检查是否包含 bitcode，AB 均测试，结果数量都一样，报错的 B 文件和 A 库里其他文件测试结果也相同，具体如下。

![](/assets/images/2019/cocoapodsAndBitcode3.jpeg)

### 解决

因为 B 库是依赖于 cocoapods 的，所以得给 APP pod install，编译能成功也能运行，但是不能打包。

从这一点思考出可能是因为 cocoapods 编译引起的，赶紧继续搜索，找到以下代码，加入 podfile 里，终于可以打包了，不容易啊。

```
#bitcode enable
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'YES'

      if config.name == 'Release'
          config.build_settings['BITCODE_GENERATION_MODE'] = 'bitcode'
      else
          config.build_settings['BITCODE_GENERATION_MODE'] = 'marker'
      end

      cflags = config.build_settings['OTHER_CFLAGS'] || ['$(inherited)']

      if config.name == 'Release'
          cflags << '-fembed-bitcode'
      else
          cflags << '-fembed-bitcode-marker'
      end

      config.build_settings['OTHER_CFLAGS'] = cflags
    end
  end
end
```

果然给 cocoapods 打开 bitcode 的思路是对的。

## 拓展

cocoapods 禁用 bitcode

```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'NO'
    end
  end
end
```
