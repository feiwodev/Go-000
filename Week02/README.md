### Week02 学习笔记

> 作业：我们在数据库操作的时候，比如 dao 层中当遇到一个 `sql.ErrNoRows` 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

### 分析

首先弄清楚`sql.ErrNoRows`是一个什么样的`error`, 在包`database/sql`包中的`sql.go`源码中，我们可以找到该`error`的定义`var ErrNoRows = errors.New("sql: no rows in result set")` 在定义中我们可以看出，该`error`是在查询数据库时，没有查询结果集，则会返回这个`error`，类似`gorm`库中的`gorm.ErrRecordNotFound`的`error`。

`sql.ErrNoRows`文档

> ErrNoRows is returned by Scan when QueryRow doesn't return a row. In such a case, QueryRow returns a placeholder *Row value that defers this error until a Scan.

### 解

那么，当查询出现了`sql.ErrNoRows`这个`error`， 我们应不应该返回呢？个人觉得应该分情况判定， 我一般的做法是，查询列表数据时， 在这种确定的情况下，出现`sql.ErrNoRows`返回空的`slice`， 查询单个数据时候，返回`sql.ErrNoRows`，方便上传服务做`error`处理，往上层抛自定义`error`。

### 伪代码

```go
// dao.go

type User struct {
  Name string
  Age int
  ...
}

// 查询列表是， 查询结果集为empty返回空的slice
func FindUsers(args ...string) ([]*User, error){
  var users []*User
  err := db.find(args, &users).Error
  if (err != nil) {
    if (erros.is(err, sql.ErrNoRows)) {
      return users, nil
    }
    return nil ,err
  }
  return users
}

// 查询单个数据时，查询结果集为empty是，直接返回 error
func FindUserById(id uint) (*User, error) {
  u := new(User)
  err := db.Find(User{}).Where("id = ?", id).FindFirst(u).Error
  if (err != nil) {
    return nil, err
  }
  return u
}
```



