# MRC简单用法


---


手动管理内存。涉及关键字：`retain`,`release`, `AutoreleasePool`

- 示例1.set方法中的使用

```objc

  @interface Model:NSObject
{
	NSString *_testSetter;
}
- (void)setTestSetter:(NSString *)str;
@end

@implementation Model

- (void)setTestSetter:(NSString *)str
{
    //设置的不是同一个值
	if(_testSetter != str)
	{
        //先把原先持有的值释放掉。release
		[_testSetter release];
        //然后重新持有新的值。让新的值 retain一次
		_testSetter = [str retain];
	}
}

- (void)dealloc
{
	NSLog(@"%s",__func__);
    //结束的时候 释放持有的对象
	[_testSetter release];
	_testSetter=nil;
	[super dealloc];
}

@end
```
- 示例2 在@property中使用
 
```objc

@interface Student : NSObject
{
	Book *_b; //在低5.0前需要这里要写。@property不会自动实现
}
//在头文件中声明getter,setter。像下面这样
//- (void)setBook:(Book *)book;
@property(retain,nonatomic) Book *book;

@end

//实现
@implementation Student
//自动生成实现getter与setter。在5.0前需要这样写。
//表示生成的getter与setter方法中操作的成员变量为_b
//book 是头文件中对应的@property声明的变量
@synthesize book=_b;

-(void)dealloc
{
    //在结束的时候，释放成员变量
	[_b release];
	_b=nil;
	NSLog(@"%s",__func__);
	[super dealloc];
}
@end

```
- 示例3.自动释放池

```objc
//5.0前的写法。
NSAutoreleasePool *pool=[[NSAutoreleasePool alloc] init];
	
		Book *b=[[Book alloc] init];
		[b autorelease]; //把对象加入池子
		Student *stu=[[Student alloc] init];
		[stu autorelease];
		
		stu.book = b;
		NSLog(@"%i",[b retainCount]);
		
        //在这里不要手动的调用对象的retain，不然，下面的池子也不会自动释放此对象。
        //手动调一次retain就要对应的手动调用一次release
        
	[pool release]; //只有调用此方法，对象才会释放
```