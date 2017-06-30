---
layout: post
title: "推开HTML的门：搭界面"
date: 2017-06-30 19:30:38.000000000 +09:00
tags: 回梦游仙
---

哇，不得不感叹，一个月过得太快，虽然没什么活，但是自己也没什么产出，很尴尬。只能用最近研究的东西水一发了……惭愧惭愧……

这段时间逛GitHub的时候，偶然间发现[freecodecamp(免费学coding的夏令营（自译）)](https://www.freecodecamp.com)，毕竟自己想学网页，就参加了这个夏令营。

这里把每一步作为一个关卡，加上一些水关，我现在进行到了137关，现在在JS入门阶段，没法总结，所以在这展示下之前的HTML阶段的成果吧。

```HTML
<link href="https://fonts.googleapis.com/css?family=Lobster" rel="stylesheet" type="text/css">
<style>
  h2 {
    font-family: Lobster, Monospace;
  }

  .thick-green-border {
    border-color: green;
    border-width: 10px;
    border-style: solid;
    border-radius: 50%;
  }

</style>

<div class="container-fluid">
  <div class="row">
    <div class="col-xs-8">
      <h2 class="text-primary text-center">CatPhotoApp</h2>
    </div>
    <div class="col-xs-4">
      <a href="#"><img class="img-responsive thick-green-border" src="https://bit.ly/fcc-relaxing-cat" alt="A cute orange cat lying on its back. "></a>
    </div>
  </div>
  <img src="https://bit.ly/fcc-running-cats" class="img-responsive" alt="Three kittens running towards the camera. ">
  <div class="row">
    <div class="col-xs-4">
      <button class="btn btn-block btn-primary"><i class="fa fa-thumbs-up"></i> Like</button>
    </div>
    <div class="col-xs-4">
      <button class="btn btn-block btn-info"><i class="fa fa-info-circle"></i> Info</button>
    </div>
    <div class="col-xs-4">
      <button class="btn btn-block btn-danger"><i class="fa fa-trash"></i> Delete</button>
    </div>
  </div>
  <p>Things cats <span class="text-danger">love:</span></p>
  <ul>
    <li>cat nip</li>
    <li>laser pointers</li>
    <li>lasagna</li>
  </ul>
  <p>Top 3 things cats hate:</p>
  <ol>
    <li>flea treatment</li>
    <li>thunder</li>
    <li>other cats</li>
  </ol>
  <form action="/submit-cat-photo">
    <div class="row">
      <div class="col-xs-6">
        <label><input type="radio" name="indoor-outdoor"> Indoor</label>
      </div>
      <div class="col-xs-6">
        <label><input type="radio" name="indoor-outdoor"> Outdoor</label>
      </div>
    </div>
    <div class="row">
      <div class="col-xs-4">
        <label><input type="checkbox" name="personality"> Loving</label>
      </div>
      <div class="col-xs-4">
        <label><input type="checkbox" name="personality"> Lazy</label>
      </div>
      <div class="col-xs-4">
        <label><input type="checkbox" name="personality"> Crazy</label>
      </div>
    </div>
    <div class="row">
    <div class="col-xs-7">
    <input type="text" class="form-control" placeholder="cat photo URL" required>
      </div>
    <div class="col-xs-5">
    <button type="submit" class="btn btn-primary"><i class="fa fa-paper-plane"></i> Submit</button>
      </div>
      </div>
  </form>
</div>
```

基于`Bootstrap`框架产出的，效果如下图——

![](/assets/images/2017/toHTML1.png)

![](/assets/images/2017/toHTML2.png)

###反思

这次有点偷懒，但是还是要思考下以后要做的事情

> - 探索`Bootstrap`框架
> - 自己不看提示自己做出这个界面
> - 要有紧迫感，不能偷懒
