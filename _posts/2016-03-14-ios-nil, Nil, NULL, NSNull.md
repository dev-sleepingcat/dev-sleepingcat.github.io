---
layout: post
title: iOS - nil, Nil, NULL, NSNull
---
{{ page.title }}
==================
<p class="meta">2016年3月14日 - 杭州</p>

之前一直不曾注意nil,Nil,NULL,NSNull这几个的区别，一般都是直接用nil。这次趁着排查一个相关的Crash问题顺便总结一下，这里面的坑还是蛮深的，后面需要好好的注意一下。

#### nil

nil 是 Objective-C 对象的字面空值，对应 id 类型的对象，或者使用 @interface 声明的 Objective-C 对象。比如：

	NSObject *someObj = nil;
	if(nil == someObj) {
		// do something
	}

也许nil最显著的行为是，它虽然为零，仍然可以有消息发送给它。在其他的语言中，比如C++，这样做会使你的程序崩溃，但在Objective-C中，在nil上调用方法返回一个零值。这大大的简化了表达，因为它避免了在使用nil之前对它的检查：

#### Nil

Nil 是 Objective-C 类类型的字面空值，对应 Class 类型对象。一般不常用。比如：

	Class someCls = Nil;
	Class stringCls = [NSString class];

#### NULL

NULL 是任意的 C 指针的空值。比如：

	int *intPointer = NULL;
	float *floatPointer = NULL;
	struct Node *node = NULL;

#### NSNull

NSNull是一个Objective-C对象，代表一个空值的类。实际上它只有一个单例方法：+[NSNull null]；注意，它和nil是有很大区别的，nil表示一个空值，而NSNull则是空值对象。

NSNull经常用于Foundation的集合当中（NSArray,NSDictionary等）,因为这些集合是无法存储nil值的，其实我们经常可以在代码中看到，他们在初始化的时候会以nil结尾符号。举例来说，在Dictionary中，[dict objectForKey:key] 返回nil表示dictionary中没有当前key值对应的对象，也就是说当前key值就不在dictionary中；如果你想要在dictionary中显示的增加一个空值，则需要用到[NSNull null]，如此 [dict objectForKey:key] 才有可能返回NSNull空值对象。代码说明：

	NSMutableDictionary *dict = [NSMutableDictionary dictionary];
	[dict setObject:nil forKey:@"someKey"];
以上代码会抛出异常，因为nil不能被插入到dict中。

	NSMutableDictionary *dict = [NSMutableDictionary dictionary];
	[dict setObject:[NSNull null] forKey:@"someKey"];
上面这段代码则是正确的，因为[NSNull null]不为nil。

#### 额外的问题：-[NSNull length]: unrecognized selector sent to JSON objects

解决方案：创建一个NSNull的Category，在Category中实现length方法，代码讲解：

	@interface NSNull (JSON)
	@end

	@implementation NSNull (JSON)

	- (NSUInteger)length { return 0; }

	- (NSInteger)integerValue { return 0; };

	- (float)floatValue { return 0; };

	- (NSString *)description { return @"0(NSNull)"; }

	- (NSArray *)componentsSeparatedByString:(NSString *)separator { return @[]; }

	- (id)objectForKey:(id)key { return nil; }

	- (BOOL)boolValue { return NO; }

	@end

通过NSNull+JSON.m完美解决，只需要在用的时候import次Category即可。

引用：

[http://stackoverflow.com/questions/5908936/difference-between-nil-nil-and-null-in-objective-c](http://stackoverflow.com/questions/5908936/difference-between-nil-nil-and-null-in-objective-c)
[http://stackoverflow.com/questions/16607960/nsnull-length-unrecognized-selector-sent-to-json-objects](http://stackoverflow.com/questions/16607960/nsnull-length-unrecognized-selector-sent-to-json-objects)
