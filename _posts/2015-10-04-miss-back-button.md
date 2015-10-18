---
layout: post
title: Missing the back button in iOS 9?
categories: [ios, swift]
tags: [ios, swift, segue]
fullview: true
comments: true
---

## Back Button?

iOS 9 and Swift 2 are awesome, but your app may not work as you expect when running on iOS 9 device. Here is a problem I found. 

When user click the edit button, it will bring up a new `ViewController`, so use can edit data then return to previous screen. It is a simple case to use show (push) `segue`. Here is the `storyboard` for the relationship between `ViewController`. 

![](/image/2015-10-04/ios-8.png)

Here is the screenshot for the app [LogICU](http://itunes.apple.com/app/log4as/id1004352445?ls=1&mt=8&uo=4) when running on iOS 8 and 9.

![](/image/2015-10-04/miss-button.png)

There is no back button in iOS 9, so user can't go back to the previous screen. When I went to google the problem, someone already report it the here [Openradar iOS 9.0 beta 3](http://www.openradar.me/21839771)


## Solution
It turns out to fix the problem is very simple: just remove the `NavigationViewController` from the data edit `ViewController`. In the iOS 9, you don't need to integrate your `ViewController` into a `NavigationViewController` if your previous `ViewController` is already integrated with one. Here is the updated `storyboard`.

![](/image/2015-10-04/ios-9.png)

### For future

1. Always test the app before the new iOS final version release
2. Use the new xcode 7 UI test in the future to detect this problem, some nice tuturial from Joe Masilotti [[1]](http://masilotti.com/ui-testing-xcode-7/)
[[2]](https://github.com/joemasilotti/UI-Testing-Cheat-Sheet) 
3. Aware the issues reported by other developers, such as [openradar](http://www.openradar.me), [developer forums](https://forums.developer.apple.com)






