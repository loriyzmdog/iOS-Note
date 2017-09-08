# KKBox 基本開發教材筆記2-Category

## Category - 不用繼承物件也能新增或替換method

在Swift中的Extension特性與Objective-C的Category差不多

### 何時應該使用Category

若想要擴充一個class的成員變數或方法，但沒有class程式碼時，正規做法就是繼承．

**但若很難繼承該class又想要擴充方法時，則使用Category**，大致為幾種狀況：

1. Foundation物件
2. 用Factory Method Pattern實作的物件
3. Singleton物件
4. 在專案中出現次數頻繁的物件

> Category可以擴充新增method，但卻**不能新增成員變數或屬性**！



- Factory Method Pattern：

> Factory Method Pattern是一套用來解決不用特別指定哪個class就可以建立物件的方法，比如某個class底下有一堆subclass，我們只要對上層class輸入指定條件，就可以從中挑選符合指定條件的subclass來建立instance回傳．

在UIKit中，UIButton是個好例子，因為UIButton在初始化時並非用alloc init，而是輸入指定type類型來建立button物件：`[UIButton buttonWithType: UIButtonTypeCustom];`、`[UIButton buttonWithType: UIButtonTypeRoundedRect];`

若想擴充UIButton但難以繼承UIButton，這時我們就會使用category來擴充．

也就是說：

**假使今天我們的需求是想要改動某個上層class，讓下層所有subclass都新增一個method，而我們又無法更動上層class時，就是採用category**．例如：我們希望所有UIViewController都有一個新method，讓整個應用程式中的每個UIViewController的subclass都可以呼叫這個method，但我們無法改動UIViewController，就會採用Category



- Singleton物件：

> 指某個class只有、也只該有一個instance，每次都只對這個instance操作，而非建立一個新的instance，例如：UIApplication、NSUserDefault、NSNotificationCenter，都採用singleton設計，它難以繼承

如何實作singleton：

宣告部分

```
@inteface MyClass: NSObject
+ (MyClass *) sharedInstance;
@end
```

實作部分

```
static MyClass *sharedInstance = nil;

@implemetntation MyClass
+ (MyClass *) sharedInstance{
  return sharedInstance ? sharedInstance : (sharedInstance = [[MyClass alloc] init]);
}
@end
```

**現在singleton大多會使用GCD的`dispatch_once`實作**，避免多重thread環境下時，可能會產生重複建立instance的問題，確保只呼叫一次alloc init，只建立一次instance：

```
@interface MyClass: NSObject
+ (instancetype) sharedInstance;
@end

@implementation MyClass
+ (instancetype) sharedInstance {
  
  static MyClass *instance = nil;
  static dispatch_once_t onceToken;
  
  dispatch_once(&onceToken, ^{
    instance = [[MyClass alloc] init];
  });
  return instance;
}
@end
```

若我們subclass了MyClass，卻沒有override掉sharedInstance，那麼sharedInstance回傳的依舊是MyClass的singleton instance，可是如果我們沒有MyClass程式碼，要override sharedInstance也不容易，或是說直接override掉sharedInstance有可能會發生其他情形產生不符合預期結果時，可以使用category



- 在專案中若有某些class已頻繁使用地到處都是，此時有需求改變必須新增新method，可我們卻也無法通通變更成新的subclass時，可以使用category來解決問題

### 實作Category

語法：一樣使用@interface宣告header，在@implementation/@end中實作，在原本的class名稱後面用()表示新增的category名稱

例如1：我們不想用NSLog印出某物件資料，而是每個物件都把自己印出來的method，所以我對NSObject建立了一個叫SmallTalkish的category

```
@interface NSObject (SmallTalkish)
- (void) printNl;
@end

@implementation NSObject (SmallTalkish)
- (void) printNl{
  NSLog(@"%@",self);
}
@end
```

如此一來每個NSObject都有一個新method，可以這麼呼叫：`[aObject printNl];`

例如2:假如我們希望字串陣列做排序時，能依照中文筆畫順序來排序，我們可以自己寫一個method

```
@interface NSString (CustomCompare)
- (NSComparisonResult)strokeCompare: (NSString *)anotherString;
@end

@implementation NSString (CustomCompare)
- (NSComparisonResult)strokeCompare: (NSString *)anotherString {

	NSLocale *strokeSortingLocale = [[[NSLocale alloc] initWithLocaleIdentifier: @"zh@collation=stroke"] autorelease];
	
	return [self compare: anotherString
				 options: 0
				 range: NSMakeRange(0, [self length])
				 locale: strokeSortingLocale];
}
@end
```

其生成的檔名為：NSString+CustomCompare.h, NSString+CustomCompare.m

### Category其他用途

除了幫原有的class新增method外，其他幾種情況也會使用category:

1. 將一個大class切成幾個部分 - 跨專案、跨平台
2. 替換原本的實作

#### 將一個大class切成幾個部分

若一個class裡有數十個method，實作上千行，我們可以考慮將method拆成數個category，讓整個class的實作分開在不同檔案中，以利方便知道某一群method屬於什麼用途，方便維護．

###### 跨專案

切開class的好處是，若之前寫的程式碼可以重複使用，造成可能每個專案都會共用一個class，但並非會用到這個class內的全部實作，故我們可以將只屬於某專案的實作部分，拆分成一個category

###### 跨平台

若我們打算在Mac OS X與iOS共用同一個class的話，就需要考慮將跨平台的部分與平台相依的部份拆開，將只屬於某個平台的部分拆成另一個category．

例如：Mac OS X 與 iOS都有`NSString`，但兩個平台在繪圖方面的實作有所不同，所以在繪製字串的部分，就會被拆分到`NSStringDrawing`與`UIStringDrawing`這些category中．

#### 替換原本的實作

在Objective-C中，因為一個class有哪些method是於run time時加入的，故我們若在category中加入一個已經存在且名稱相同的method時，則category被執行時，原本class的method實作就會被替換成category中的method實作．

但是實務上這需要避免，有可能日後維護時忽略category內的method部分，造成怎麼改動class的method都無效．或是，多個category都有這個名稱相同的method實作時，因為我們無法確保category被執行的順序，所以無法得知哪個categroy的method被實作，我們須避免寫出這種程式．



















