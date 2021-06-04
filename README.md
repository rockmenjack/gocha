# gocha 避坑手册
## golang
### unreleased resource with blank identifier 你忽略的变量可能会引起资源泄漏
Sometimes you might ignore some of the return values from a function call, because you don't need them:
```go
val, err := f()
// if val is never used, then we might change to:
_, err := f()
```
however if for example `val.Close()` must be called to release the underlying resources, then you are leaking with blank identifier

