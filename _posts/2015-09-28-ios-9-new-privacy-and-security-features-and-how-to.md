---
ID: 80
post_title: >
  iOS 9 new privacy and security features
  and how to comply to (or work around)
  them
author: Lorenzo Bulfone
post_date: 2015-09-28 14:08:37
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/09/28/ios-9-new-privacy-and-security-features-and-how-to/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/130061135359/ios-9-new-privacy-and-security-features-and-how-to
tumblr_mikamayhem_id:
  - "130061135359"
---
<p>With the release of iOS 9 last week, Apple has introduced new under-the-hood features meant to protect the end-user and their privacy. While this is a welcome addition, said features may break your app if not addressed properly.</p>

<h2>App Transport Security</h2>

<p>According to Apple, <strong>App Transport Security</strong> is a feature that improves the security of connections between an app and web services. The feature consists of default connection requirements that conform to best practices for secure connections.</p>

<p>These are the App Transport Security requirements:</p>

<ul><li>The server must support at least Transport Layer Security (TLS) protocol version 1.2.</li>
<li>Connection ciphers are limited to those that provide forward secrecy (see the list of ciphers below.)</li>
<li>Certificates must be signed using a SHA256 or greater signature hash algorithm, with either a 2048-bit or greater RSA key or a 256-bit or greater Elliptic-Curve (ECC) key.
Invalid certificates result in a hard failure and no connection.</li>
</ul><p>The <strong>default behaviour</strong> causes all connections using NSURLConnection, CFURL, or NSURLSession to fail if they don&rsquo;t follow these requirements.
Luckily, this layer of security can be disabled pretty easily by adding an exception into your Info.plist file. Just add a <strong>NSAppTransportSecurity</strong> dictionary to the .plist and inside of that a <strong>NSExceptionDomains</strong> dictionary with the desired exceptions. You can also disable the transport security altogether by adding a <strong>NSAllowsArbitraryLoads</strong> boolean to the first dictionary. For more options just check the relative <a href="https://developer.apple.com/library/prerelease/ios/technotes/App-Transport-Security-Technote/" title="App Transport Security Technote">Apple technote</a>.</p>

<h2>Privacy and URL Scheme Changes</h2>

<p>URL schemes are used by many apps for a variety of purposes. A common use is to deep link to a particular piece of content within an app – seen as an “Open in app” link from a web page.</p>

<p>Before trying to open a URL handled by another app (using NSApplication&rsquo;s openURL: method) it&rsquo;s good practice to check if that app is actually installed on the device. NSApplication&rsquo;s canOpenURL: method used to return a YES or NO answer after checking if there were any apps installed on the device that know how to handle the given URL.</p>

<p>With iOS 9, canOpenURL: can&rsquo;t be used on arbitrary URLs anymore. Apple noticed that some apps were using this method to collect information about what apps were or were not installed on the user&rsquo;s device for marketing and other purposes the method was not intended for.</p>

<p>From now on canOpenURL: will return NO even if there is an app able to handle the given URL, unless said URL has been whitelisted in the app Info.plist before submission to the App Store. These limits will be retroactively enforced on apps built and submitted to the store prior to iOS 9 by the system, which will build it’s own whitelist of up to 50 URL schemes allowed for an app based on the actual calls made by the app.</p>

<p>To whitelist the URL schemes your app needs to function properly, just a <strong>LSApplicationQueriesSchemes</strong> array into your Info.plist and individual allowed schemes inside of that array.</p>

<p>This new limitations should not take long to adjust to for apps that use canOpenURL: sparingly, but it can have a very big impact on launcher-style apps that heavily rely on them. For more information about this, check out Apple <a href="https://developer.apple.com/videos/wwdc/2015/?id=703" title="Privacy and Your App">Privacy and Your App video</a> and Awkard Hare&rsquo;s <a href="http://awkwardhare.com/post/121196006730/quick-take-on-ios-9-url-scheme-changes" title="Quick Take on iOS 9 URL Scheme Changes">Quick Take on iOS 9 URL Scheme Changes</a>.</p>