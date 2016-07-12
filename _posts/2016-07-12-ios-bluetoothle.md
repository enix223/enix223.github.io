---
layout: blog
categories: blog
published: true
title: iOS BluetoothLE 开发资料汇总
---
iOS蓝牙开发搜集到的资料，以备后续查阅使用。

## 通过RSSI计算距离:

	powe(10, (abs(rssi) - 59) / (10 * 2.0));


## BLE 问题与解答

[AudioSessionManager](https://github.com/Jawbone/AudioSessionManager/blob/master/AudioSessionManager.m)


[AVAudioSession bluetooth support](http://devmonologue.com/ios/tutorials/avaudiosession-bluetooth-support/)


[AudioSession input from bluetooth output to line out or speaker](http://stackoverflow.com/questions/8305986/audiosession-input-from-bluetooth-output-to-line-out-or-speaker)


[Bluetooth mic recorder app](http://lifehacker.com/5879232/the-best-voice-recording-app-for-iphone)



[iOS: Using Bluetooth audio output (kAudioSessionProperty_OverrideCategoryEnableBluetoothInput) AudioSession](http://stackoverflow.com/questions/14601517/ios-using-bluetooth-audio-output-kaudiosessionproperty-overridecategoryenableb)


[iPhone蓝牙编程之实现语音聊天](http://tech.chinaunix.net/a2010/0129/845/000000845690.shtml)
