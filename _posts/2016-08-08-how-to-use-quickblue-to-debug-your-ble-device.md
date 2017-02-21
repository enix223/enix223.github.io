---
layout: post
categories: iOS
published: true
title: How to use QuickBlue to debug your BLE device
---
This post is about to describe how to use my iOS app, named `QuickBlue` to help you debugging your BLE device. There are many app in apple store to work with BLE device, and the most favorite one, should be `lightBlue`. It is really powerful, and you can even use it to creaet many virtual devices. But from my point of view, it dose have some weakness.

If you have been working with BLE development, you may have encountered the following paintful issues:

1. If you have sent data to the device with lightblue, you may not get it back when you close your app, and reopen it again. This is paintful if you have done some development to your BLE peripheral, and need to test with the app again. You have no choice but type in the data again in the app, or copy the data to `Note` and save it for future usage.

2. You can not download all the data from the app all during you test. Or you can only use the stupid way, `copy and paste` with your `Note` app to keep your record.

3. You can't customize a workflow and 'replay' it to commit repeat testing task. For many cases, we need to test a function with 2-3 frames (or more) exchanging between the app and BLE device. And if you have fixed some issue in the BLE device, and you have to repeat typing the testing frames in the app, and send and wait for the response from the device, after that, you have to type another frame data in the app, and send. Or you may need to repeat this workflow again and again.

If you have got such awful experiences, then `QuickBlue` is right here waiting for you.

`QuickBlue` is one of my apple app work, and aim to reduce the workload when developing with BLE device. With `QuickBlue`, you can earn the following functions:

1. Keep your send history data, and get it back when you need it.

2. Keep your testing data, and send it out to your mailbox.

3. Record a repeating workflow, and customize it as you wish. You can also replay the action, without doing repeating task.

Please do not hestiate to download `QuickBlue` from apple store. And you may love it and enjoy developing with Bluetooth 4.0.

Happy coding. :)