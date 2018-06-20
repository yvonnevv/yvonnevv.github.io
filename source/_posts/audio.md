---
title: H5音频踩坑与填坑
date: 2017-09-08
categories: 前端
comments: true
articleId: audio
---

最近用createjs完成了个H5需求，体验二维码如下。在音效接入方面踩了一点坑，但...庆幸的是，坑还是能被填上的。

![clipboard.png](https://segmentfault.com/img/bVURYI?w=370&h=370)



本文能为你解决：

- 微信音频自动播放问题
- audio预加载问题（解决网络环境差下，音频由于未缓冲完造成的音效动画不同步问题

## 微信环境下音频自动播放
### 问题
IOS设备系统是不允许视频音频自动播放的，需要用户明确指定播放（通过一定的交互动作），相关的音频或视频才能被加载。

### 解决（推荐使用方法一

**方法一**：使用weixin提供的sdk

```
// 引入sdk
<script src="https://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>

var audio = (function(){
  var _audio = new Audio();
  _audio.src = 'xxx';
  _audio.load();
  return _audio;
})()

// 微信配置信息（即使不正确也没问题
wx.config({
  debug: false,
  appId: '',
  timestamp: 1,
  nonceStr: '',
  signature: '',
  jsApiList: []
});
// 在ready时触发相关事件
wx.ready(function() {
  // 触发一下play事件
  audio.play();
});
```

**方法二**：野生方法

```
var audio = (function(){
  var _audio = new Audio();
  _audio.src = 'xxx';
  _audio.load();
  return _audio;
})()

document.addEventListener("WeixinJSBridgeReady", function () {
  WeixinJSBridge.invoke('getNetworkType', {}, function (e) {
    // 触发一下play事件
    audio.play();
  });
}, false);
```
### 说明
1、为什么推荐使用方法一？经过验证，方法二的会在微信浏览器准备完成后马上执行回调函数，假如我们使用了第三方库预加载音频，在页面监听到`WeixinJSBridgeReady`事件时，音频由于未加载完成，所以是获取不到的。

2、只要我们在`WeixinJSBridgeReady` OR `ready`时触发了`audio.play()`事件，浏览器会识别到audio对象已被启用，这样我们可以在H5动画的任意位置使用`audio.play()`和`audio.pause()`，而不需要再监听用户动作以触发音频播放。

## 音频预加载
上文提到过，为什么需要音频预加载，在动画音效丰富的H5中，可能不止bgm一个音频，同时，我们还要根据场景的变换播放不同的音效。假如在我们动画播放过程中，音频还没有缓冲完毕，或者只缓冲了一部分，音频是会不播放或者停止播放的。这样，就会造成动效音效不同步的问题了。（在叙事性H5动画中尤为重要

### 使用audio 标签属性 preload（不推荐
很简单，直接在标签里使用preload，**燃鹅**该方法兼容性不太好，不同机型差异大，并且有些机型下会有preload与无preload表现一致，并不会对音频缓冲带来多大改善。

```
<audio src='xxx.mp3' loop=true preload='auto' />

```
### preloadjs
推荐使用`createjs`里的`preloadjs`，但还需要配合`soundjs`使用。

官方文档给出的预加载原因，和上文提到的大同小异。
> Without CreateJS, most modern browsers do a great job of loading enough audio data to play back a sound continuously until the end using HTML tags. The canplaythrough event will fire when the buffer is full, and the sound will start. This is sufficient for a click-to-play sound, or for a background sound that can start whenever its ready - but in order to play on-demand audio for games, applications, and web sites, sounds must be preloaded first.

**使用**

**1、preloadjs + soundjs**
```
var queue = new createjs.LoadQueue();

// 添加声音支持
queue.installPlugin(createjs.Sound)

queue.addEventListener("fileload", handleFileLoad);
queue.addEventListener("complete", handleComplete);
 
queue.loadFile({id:"bgm", src:"assets/bgm.mp3"});
// OR
queue.loadManifest([
  {id:"bgm", src:"assets/bgm.mp3"},
  {id:"myImage2", src:"assets/image2.jpg"}
]);

// 资源全部加载完成时触发
var handleComplete = function () {
  // 在次调用weixinReady事件，让浏览器获得音频对象
  /*
     注意：音频实例为 var bgmInstance = createjs.Sound.play("bgm")
     然后可以通过控制 bgmInstance.play() OR bgmInstance.stop() 播放暂停音效
  */
  ...
}
```

**2、仅使用soundjs**

    // createjs.Sound.alternateExtensions = ["mp3"]; 控制音频类型
    createjs.Sound.registerSound({id:"bgm", src:"assets/bgm.mp3"});
    createjs.Sound.play("bgm");
 
使用`preloadjs`会在预加载时调用`createjs.Sound.registerSound`事件注册声音，所以无需显示调用。

** *音频实例类型选择**
`preloadjs`加载的默认音频实例是[AudioBuffer][1]，也就是已解码的音频流，感兴趣的同学可以看看api，本人对它不太了解，就不误导大家了。

即：

```
console.log(createjs.Sound.play('bgm')) // 返回AudioBuffer实例
```

所以我们不能通过`audio`的`api`来对它进行操作，`soundjs`为我们提供了控制它：[AbstractSoundInstance][2] 的`api` 。那么我们还能不能通过`audio dom`的方式控制呢？答案是可以的，但是官方不推荐，并且本人在**移动端**也遇到获取不到实例对象的问题，还是稍微提一下：

```
// 注册HTMLAudioPlugin，那么将声音注册为原生的audio dom
createjs.Sound.registerPlugins([createjs.HTMLAudioPlugin]);

// 预加载
var queue = new createjs.LoadQueue();

// 添加声音支持
queue.installPlugin(createjs.Sound);

queue.addEventListener("fileload", handleFileLoad);
queue.addEventListener("complete", handleComplete);
 
queue.loadFile({id:"bgm", src:"assets/bgm.mp3"});

// 获取
var handleComplete = function () {
  var bgmAudioDom = queue.getResult('bgm');
  // console.log(bgmAudioDom) 返回 <audio src='assets/bgm.mp3' loop></audio>
}

```

## 项目代码截取

```
// 定义loader
var loader = new createjs.LoadQueue();

// 添加声音支持
loader.installPlugin(createjs.Sound)

loader.addEventListener('progress', __handleLoading);
loader.addEventListener('complete', __handleComplete);
loader.loadManifest(manifest);

var manifest = [
{src:"plugin/bgm.mp3", id:"bgm"},
  {src:"plugin/s1_female1.mp3", id:"s1_female1"},
  {src:"plugin/s1_female2.mp3", id:"s1_female2"},
  {src:"plugin/s1_danmu.mp3", id:"s1_danmu"},
  {src:"plugin/s1_effect.mp3", id:"s1_effect"},
  {src:"plugin/s3_switch.mp3", id:"s3_switch"},
]

// 加载完成
var __handleComplete = function (evt) {
  wx.config({
    debug: false,
    appId: '',
    timestamp: 1,
    nonceStr: '',
    signature: '',
    jsApiList: []
  });

  wx.ready(function() {
    musicControlData['bgm'].instance = createjs.Sound.play('bgm');
    musicControlData['bgm'].instance.stop();
  });

}
```
该H5使用了preload预加载第一屏所需音效，分屏加载其他音效的方式，所以是方法1和2结合使用的，这里不展开叙述了，有兴趣的同学私下交流（拒绝伸手党，礼貌微笑～

![clipboard.png](https://segmentfault.com/img/bVUEKk?w=68&h=86)

【完】


  [1]: https://developer.mozilla.org/en-US/docs/Web/API/AudioBuffer
  [2]: http://createjs.com/docs/soundjs/classes/AbstractSoundInstance.html