---
ID: 161
post_title: >
  HID emulation on an iPhone is not
  allowed
author: Emanuel Carnevale
post_date: 2015-02-03 09:16:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/02/03/hid-emulation-on-an-iphone-is-not-allowed/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/109964410059/hid-emulation-on-an-iphone-is-not-allowed
tumblr_mikamayhem_id:
  - "109964410059"
---
<p>Last week in a project of ours we were exploring the BLE capabilities of <code>CoreBluetooth</code> framework in iOS: the vast majority of apps have the iPhone working as a central Bluetooth hub and interacting with various BLE peripherals that advertise their services.
We wanted instead to have the iPhone acting as a peripheral and advertise its services to another Bluetooth enabled device.</p>

<p>For the sake of simplicity we decided to implement an HID device: the iPhone would&rsquo;ve acted as a bluetooth keyboard.</p>

<p>We set up our service and characteristics constants as:</p>

<h1>define humanInterfaceDeviceService @&ldquo;0X180D&rdquo;</h1>

<h1>define bootKeyboardInputReportService @&ldquo;0X2A22&rdquo;</h1>

<h1>define bootKeyboardOutputReportService @&ldquo;0X2A32&rdquo;</h1>

<p>and created a <a href="https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBPeripheralManager_Class/index.html#//apple_ref/occ/cl/CBPeripheralManager">CBPeripheralManager</a> and implemented the <a href="https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBPeripheralManagerDelegate_Protocol/index.html#//apple_ref/doc/uid/TP40013016">CBPeripheralManagerDelegate</a> into our <code>ViewController</code>. More info <a href="https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW1">here</a>.</p>

<p>Unfortunately the service was not being correctly discovered and after some tedious debugging, we found out Apple has intentionally reserved HID support for iOS since it filters out all related services during the discovery process, although there&rsquo;s no clear documentation about that.</p>

<p>Changing constants to non standard ones solved the discovery problem, although in this way the iPhone cannot work anymore as a standard HID device.</p>