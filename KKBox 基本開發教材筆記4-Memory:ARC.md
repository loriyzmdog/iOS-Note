# KKBox 基本開發教材筆記4-Memory/ARC

[TOC]



###**Memory Leak**(記憶體漏水)

**記憶體用完之後沒有釋放**，造成佔用了一推沒有用到的記憶體，當記憶體用量越來越大，造成記憶體用盡時，iOS系統會強制終止我們的應用程式，這種狀況就是memory leak

反之，若記憶體已被釋放，但我們卻以為這塊記憶體還存在可以呼叫，當嘗試呼叫時，應用程式會crash，這種狀況叫做**over-release**或是**invalid memory reference**，crash log的錯誤類型是`EXC_BAD_ACCESS`



###Reference count

只要一個物件被某個地方用到一次，這個地方就對這個物件加一，反之就減一，若數字減到變成零就是放這塊記憶體；一個物件如果使用了alloc, init，初始的reference count = 1

可以使用`retain` `release`加減reference count，例如：

```
[anObject retain]; //+1
[anObject release]; //-1

NSLog(@"Retain ocunt: %d", [anObject retainCount]); //用retainCount檢查某個物件被retain了幾次

//錯誤例子1
id a = [[NSObject alloc] init];
[a release];
[a release]; 
//因為第二行已經將a釋放，第三行想釋放會造成錯誤

//錯誤例子2
id a = [[NSObject alloc] init];
id b = [[NSObject alloc] init];
b = a;
[a release];
[b release];
// 由於第三行中，b指向了a原本指向的記憶體，但b原本的記憶體卻沒有釋放，同時也沒有任何物件會指向b原本的記憶體，因此這塊記憶體就會發生memory leak．
// 而因為第四行已將a原本的記憶體釋放，所以在a與b指向同塊記憶體時，執行第五行[b release]時則找不到該記憶體，故發生over-release，也就是EXC_BAD_ACCESS錯誤．

```

- 呼叫retain、release時機包括：
  - 在一般程式碼中用了某個物件，用完就要release或是auto-release
  - 如果要將某個物件變成另一個物件的成員變數時，就要將物件retain起來，但須注意delegate物件不能被retain
  - 在釋放一個物件時，要同時釋放該物件自己的成員變數，也就是在實作dealloc時釋放自己的成員變數



#### Getter/Setter與Property語法

**某個物件要設為另一個物件的成員變數時，需寫一組getter/setter**

```
@interface MyClass: NSObject 
{
  int number;
}
- (int)number;  // getter, in Objective-c, we do not write 'getNumber'
- (void)setNumber: (int)inNumber;  // setter
@end
```

```
- (int)number {
  retrun number;
}
- (void)setNumber: (int)inNumber {
  number = inNumber;
}
```

使用Property語法，變成：

```
@interface MyClass: NSObject 
@property (retain, nonatomic) id myVar;
@property (assign, nonatomic) int number;
@end

@implementation MyClass 
@synthesize myVar;  
@synthesize number;

@end
```

`@synthesize`：在compile的時候，會編譯成上面所寫的getter/setter

- 注意：
  - `myVar = nil`：並非釋放原本所指向的記憶體位置，而只是單純的將myVar指標指向nil而已，會造成memory leak
  - `self.myVar = nil`：等同呼叫`[self setMyVar: nil]`，會先釋放myVar原本指向的記憶體，再將myVar設成nil



## ARC

有了ARC，則原本程式碼加入retain, release, [super dealloc] 的地方都要拿掉

#### 要了解哪些地方是weak reference

我們可以將target/action與必要的參數結合成另一種物件-**NSInvocation**，在ARC下**從NSInvocation拿出參數時需注意記憶體管理問題**

例如：把對UIApplication要求開啟指定URL這件事變成一個Invocation

```
// 設好Invocation參數
NSURL *url = [NSURL URLWithString:@"https://kkbox.com"];
    NSMethodSignature *sig = [UIApplication instanceMethodSignatureForSelector: @selector(openURL:)];
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:sig];
    [invocation setTarget:[UIApplication sharedApplication]];
    [invocation setSelector:@selector(openURL:)];
    [invocation setArgument:&url atIndex:2];

// 取出Invocation參數
    __weak NSURL *arg = nil; 
    //加上__weak則此物件不會被ARC管理，要求ARC不要幫此物件作任何自動retain/release
    
    [invocation getArgument:&arg atIndex:2];
    NSLog(@"arg:%@",arg);
    //不加上__weak，則會在印出NSLog後Crash, EXC_BAD_ACCESS
```

crash原因：在透過`getArgument:atIndex:`拿出參數時，ARC並不會把arg多retain一次，到了NSLog印出後，ARC認為我們不會用到arg了，故對arg多做一次release，造成retain與release不成對

解決方式：將arg設為weak reference，讓arg這個物件不會被ARC管理，要求ARC不要幫這個物件作任何自動的retian/release，使用\__weak or __wusafe_unretained

### Retain Cycle

大意為**A物件 retain B物件，B物件 retain A物件, 我們在要釋放Ａ的時候才會釋放Ｂ，但Ｂ又得再被釋放時才會釋放Ａ，最終造成Ａ和Ｂ都無法被釋放**．ARC即使遇到了Retain Cycle也不會排除， 還是會造成memory leak

可能出現情形：

1. **將delegate設為strong reference**
2. **某個物件的某個property是一個block，但是在這個block裡把物件自己給retain了**
3. **使用timer**時，到了dealloc時才停止timer

關於3.timer：

使用`+scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:`建立timer物件，指定定時執行某個selector，注意在建立timer時，我們指定給timer的**target會被timer retain一份**，因此若我們想要在view controller做dealloc的時候來停止timer就會有問題，因為<u>view controller已經被timer retain起來</u>，所以<u>只要timer還在執行，view controller就不可能走到dealloc</u>!

解決方法：將`[self.timer invalidate];`改在**viewDidDisappear:**來停止timer















