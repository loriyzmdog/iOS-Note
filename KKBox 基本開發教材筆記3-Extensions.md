# KKBox 基本開發教材筆記3-Extensions

與category類似，可以用來拆分一個大的class，但是語法相異：

- extensions像是一個沒有名稱的category，在class名稱後直接加上空括弧
- extensions定義的method需寫在原本的class實作中

例如：

```
@interface MyClass: NSObject
@end

@interface MyClass()
- (void) doSomething;
@end

@implementataion MyClass
- (void) doSomething {
}
@end
```

在`@interface MyClass()`的括弧中沒有定義名稱，且doSomething直接在MyClass中實作

### Extensions用途

1. 拆分Header
2. 管理Private Methods

#### 拆分Header

- 若實作一個大class，而其header的method太多時，我們可以將一部分的method搬到extensions的定義裡
- **extensions除了可以放method外，也可以放成員變數**
- 一個class可以有多個extensions

> 比較
>
> Category：須於class名稱後加上括號並放入名稱，可擴充method，無法擴充成員變數
>
> Extensions：只需於class名稱後加上空括號，可擴充method及成員變數

#### 管理Private Methods

在.m/.mm檔案開頭的地方宣告一個extensions，將private methods都放在這地方，如此一來，其他method就可以找到private method的宣告．

> 在Swift中，extensions統一沒有加名字，其除了可以擴充class之外，也可擴充protocol與struct．
>
> ```
> class MyClass {
> }
> extension MyClass {
> }
>
> protocol MyProtocol {
> }
> extension MyProtocol {
> }
>
> struct MyStruct {
> }
> extension MyStruct {
> }
> ```

##### extensions新增成員變數

```
@interface MyClass()
{
  NSString *myVar;
}
@end

@implementataion MyClass
{
	NSString *myVar;  
}
@end
```







