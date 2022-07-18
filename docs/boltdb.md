# BoltDB

## 数据结构

### DB

### Bucket

### Cursor

### node

### Page
page 的结构

#### Page Header

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

