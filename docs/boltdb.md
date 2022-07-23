# BoltDB

## 数据结构

### DB

### Bucket

### Cursor

### Page
page 的结构

### node
node 是 page 加载到内存以后的结构。

#### Page Header

```go
type page struct {
  	id       pgid
	flags    uint16
	count    uint16
	overflow uint32
}
```


#### Page Body

#### Page Item

```go
// leafPageElement represents a node on a leaf page.
type leafPageElement struct {
	flags uint32
	pos   uint32
	ksize uint32
	vsize uint32
}
```

## Meta

前四个 Page
db 在初始化时（函数`func (db *DB) init() error`），会初始化两个 meta page，而 meta 在 page 中位于 page header 之后。
|page header|meta|

### Page 1

### Page 2

### Page 3

### Page 4

