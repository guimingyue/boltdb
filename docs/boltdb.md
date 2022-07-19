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

### Page 1

### Page 2

### Page 3

### Page 4

