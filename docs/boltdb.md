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

db 在初始化时（函数`func (db *DB) init() error`），会初始化两个 meta page，而 meta 在 page 中位于 page header 之后。
|page header|meta|

### Page 1 & Page 2
meta page

### Page 3
freelist Page

### Page 4
first leaf page

## 函数介绍
`func (db *DB) init() error`
当打开一个 boltdb 数据库文件的时候，如果文件大小为 0，那么就初始化这个数据库。
* 创建一个大小为 4 个页（page）大小的缓冲区（字节数组）。
* 前 2 个页置为 meta page。
* 第 3 个页置为 freelist page。
* 第 4 个页置位 B+ 树的叶子 page。
* 将这 4 个 page flush 到数据库文件中。

