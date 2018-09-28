---
layout: post
title: Yosemites dark mode
tag: MacOS
date: 2018-08-21 09:00:00.000000000 +09:00
---

### 检查Dark模式的代码如下：

```
NSDictionary *dict = [[NSUserDefaults standardUserDefaults] persistentDomainForName:NSGlobalDomain];
id style = [dict objectForKey:@"AppleInterfaceStyle"];
BOOL darkModeOn = ( style && [style isKindOfClass:[NSString class]] && NSOrderedSame == [style caseInsensitiveCompare:@"dark"]);

```

### 设置监听 darkmode 改变

```
[[NSDistributedNotificationCenter defaultCenter] addObserver:self selector:@selector(darkModeChanged:) name:@"AppleInterfaceThemeChangedNotification" object:nil];

-(void)darkModeChanged:(NSNotification *)notif {
    NSLog(@"Dark mode changed");
}
```

