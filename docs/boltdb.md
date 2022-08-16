# BoltDB

## 数据结构

### DB

### Bucket

Bucket 类似于关系型数据库（比如 MySQL）中的一张表，但是它是 Key-Value 形式的，而不是二维表，在一个 Bucket 中的 key 是唯一的。在 BoltDB 中，Bucket 之间可以是从属关系，也就是一个 Bucket 实例可以有子 Bucket，比如整个 BoltDB 实例可以看成是一个根 Bucket，每创建一个新的 Bucket，就为根 Bucket 创建一个子 Bucket。在 Bucket 中有属性`buckets  map[string]*Bucket` 来存储子 Bucket。每当打开（`func (tx *Tx) Bucket(name []byte) *Bucket`）或者创建（`func (tx *Tx) CreateBucket(name []byte) (*Bucket, error)`）一个新的 Bucket，就会创建一个子 Bucket 对象。

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
这个页是第一个 B+ 树的数据页，
比如在 Cursor 中会有根据 key 查找的操作，那么查找操作一定会遍历 B+ 树，在 `func (c *Cursor) seek(seek []byte) (key []byte, value []byte, flags uint32)` 这个方法中，会使用 c.bucket.root 这个 page 开始查找。这个 page 实际上是在 tx.init 函数中赋值的 `*tx.root.bucket = tx.meta.root`，而 tx.meta.root 实际上是 db.meta().root，在 db.init 的初始化时，会将 m.root = bucket{root: 3} 赋值给 root。


## 函数介绍
`func (db *DB) init() error`
当打开一个 boltdb 数据库文件的时候，如果文件大小为 0，那么就初始化这个数据库。
* 创建一个大小为 4 个页（page）大小的缓冲区（字节数组）。
* 前 2 个页置为 meta page，每个第 0 个 meta 的 txid 为 0，第一个 meta 的 txid 为 1。
* 第 3 个页置为 freelist page。
* 第 4 个页置位 B+ 树的叶子 page。
* 将这 4 个 page flush 到数据库文件中。

`func (tx *Tx) init(db *DB)`
事务初始化，由于 txid 会取较新的 meta 中的 txid 再加 1，所以第一个写事务的 txid 是 2，读事务的 txid 为事务开始时的较新的 meta 的事务 id

`func (tx *Tx) rollback()`
事务回滚，对于写事务，要么从 db.meta() 中的 freelist page 中恢复，要么从 db 文件中恢复。

`func (n *node) spill()` 
node 节点写入到物理 page 中，这个过程中如果 n 的中数据大于某个值（由填充因子`FillPercent`决定）就会分裂成多个 node。自顶向下进行 spill，也就是会将父节点也重新写到新的 page 中。

`func (tx *Tx) allocate(count int) (*page, error)`
事务中新的页的分配，会先调用 `db.allocate` 函数创建 page，然后将该 page 记录到 `tx.pages` 中，`tx.pages[p.id] = p`。所以调用该方法而不是直接调用 db.allocate 是为了记录事务中分配的新的 page。

`func (c *Cursor) seek(seek []byte) (key []byte, value []byte, flags uint32)`
在 Cursor 所引用的 bucket 中查找 seek（key）对应的 key，value 和这对 key-value 对应的标识，标识是标记 key， value 的类型，是代表 bucket 的信息，或是数据。标识的几个常量分别是

```go
const (
	branchPageFlag   = 0x01
	leafPageFlag     = 0x02
	metaPageFlag     = 0x04
	freelistPageFlag = 0x10
)

const (
	bucketLeafFlag = 0x01
)
```

seek 函数会调用 `func (c *Cursor) search(key []byte, pgid pgid)` 从一颗 B+ 树的根节点开始查找对应的 key 是否存在。整个查找 B+ 树的路径中经过的 page（或 node）都会记录在 Cursor.stack 中。

`func (c *Cursor) node() *node`
读取 Cursor.stack 中的 page 到 node 。

