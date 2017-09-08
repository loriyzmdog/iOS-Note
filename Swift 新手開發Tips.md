# Swift 新手開發Tips

### 1. 將字串定義成常數型別，使用struct

使用struct儲存segue ID, cell ID, storyBoard ID, dictionary key, …etc.

```swift
class FirstViewController: UIViewController {
  struct PropertyKeys {
    static let customCell = "customCell"
    static let addFirstSegue = "addFirst"
    static let addSecondSegue = "addSecond" 
  }
  
  struct SegueID {
    static let main = "main"
    static let showDetail = "showDetail"
    static let mainAddNew = "addNew"
  }
  
  struct StoryboardID {
    static let mainVC = "mainVC"
    static let note = "note"
    static let detailVC = "detailVC"
  }
}
```

### 2. 可搭配guard let 建立自定義型別的cell

- 一般可確定自定義型別不怕出錯，可以這麼寫：

  ```swift
  override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(
    				withIdentifier: PropertyKeys.customCell, 
    				for: indexPath) as! BookTableVeiwCell
    let book = books[indexPath.row]
    cell.update(with: book)
    return cell
  }
  ```

  `let cell = tableView.dequeueReuseableCell(withIdentifier: PropertyKey.customCell, for: indexPath) as! BookTableVeiwCell`中的as!為強制轉型成BookTableViewCell

- 安全寫法可使用guard let：

  ```swift
  override func tableView(_ tableView: UITableView, CellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(
    					withIndentifier: PropertyKeys.customCell, 
    					for: indexPath) as? BookTableViewCell 
    					else { 
  					  fatalError("Counld not dequeue a cell") 
    					}
    let book = books[indexPath.row]
    cell.update(with: book)
    return cell
  }
  ```

### 3. 將Cell顯示內容定義成function

```swift
class BookTableVeiwCell: UITableViewCell {
  func update(with book: Book) {
    titleLabel.text = book.title
    authorLabel.text = book.author
    genreLabel.text = book.genre
    lengthLabel.text = book.length
  }
}

class BookTableViewController: UITableViewController {
  override func tableView(_ tableView: UITableView, CellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(
  					withIndentifier: PropertyKeys.customCell, 
  					for: indexPath) as? BookTableViewCell 
  					else { 
					  fatalError("Counld not dequeue a cell") 
  					}
  	let book = books[indexPath.row]
  	cell.update(with: book) //cell顯示內容皆在update()
  	return cell
  }
}
```

