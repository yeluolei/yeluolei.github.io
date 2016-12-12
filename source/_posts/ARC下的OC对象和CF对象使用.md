---
title: ARC下的OC对象和CF对象使用
date: 2016-07-11 17:30:57
tags: ios
---

在ARC下，iOS开发终于可以自动管理内存了。然而iOS内的Core Foundation对象仍然不能自动进行管理（如Core Graphics、Core Text等）。如果我们的程序需要访问CF对象是，仍然需要手动对其进行管理。当我们创建一个CF对象后，我们需要使用`CFRelease`对其进行手动释放操作。但是当CF和OC对象进行了转化的时候，又该如何管理呢？在这里OC提供了一下几个关键字对其进行操作。

1. `__bridge`：CF和OC对象进行转化的时候只涉及对象类型转换，而不涉及对象所有权的额转换

 ```objc
 NSURL *url = [[NSURL alloc] initWithString:@"http://www.baidu.com"];
 CFURLRef ref = (CFURLRef)url;
 ```

 上面的代码在ARC下需要改成如下：

 ```objc
 NSURL *url = [[NSURL alloc] initWithString:@"http://www.baidu.com"];
 CFURLRef ref = (__bridge CFURLRef)url;
 ```

 由于`url`是OC创建的对象，并且在转换的过程中并不涉及对象所有权的转换，也就是说这个对象仍然会在OC中被释放，所以不需要调用`CFRelease`。

 同理，如果一个对象在CF中创建，并用`__bridge`在OC中使用，则需要使用`CFRelease`进行释放操作。

2. `__bridge_transfer`: CF对象转换成OC对象时，将CF对象的所有权交给OC对象，此时ARC就能自动管理该内存（作用同`CFBridgingRelease()`)

3. `__bridge_retained`:与`__bridge_transfer`相反，常用在将OC对象转换成CF对象时，将OC对象的所有权交给CF对象来管理(作用同CFBridgingRetain())

 ``` objc
 NSURL *url = [[NSURL alloc] initWithString:@"http://www.baidu.com"];
 CFURLRef ref = (__bridge_retained CFURLRef)url;
 CFRelease(ref);
 ```

 当使用`_bridge_retained`标识符以后，代表OC要将对象所有权交给CF对象自己来管理,所以我们要在ref使用完成以后用CFRelease将其手动释放.

 ``` objc
CFStringRef cfstring = CFURLCreateStringByAddingPercentEscapes(NULL,(__bridge CFStringRef)text, NULL, CFSTR("!*’();:@&=+\$,/?%#[]"), CFStringConvertNSStringEncodingToEncoding(NSUTF8StringEncoding));
NSString *ocString = (__bridge_transfer CFStringRef)cfString;
 ```
 
 此时OC即获得了对象的所有权，ARC负责自动释放该对象，如果我们在结尾加上CFRelease(cfString)纯属画蛇添足，虽不会崩溃，但是控制台会打印出该对象被free了两次。
 
文章来源：[请尊重作者](http://www.cnblogs.com/zzltjnh/p/3885012.html)
 