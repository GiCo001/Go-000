# 作业
> 1. 我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？


# 总结
### Error
- 不同于其他语言的exception是一个interface 对象
- errors.New()返回一个内部errorString对象指针（避免等值对比数据串了）
- 不同于panic ，panic会影响程序运行，而error类似于其他语言的运行时异常 

### Error类型

- Sentinel Error  预定义的特定错误 --  不灵活，缺少上下文信息，容易被破坏相等性检查

  ```go
  //通过特定值来判断  预先预料到 
  if err==io.EOF
  ```

  

- Error types  自定义错误类型struct  -- 获取更多上下文但强耦合 需要引入类型 ，通过switch 方式 判断  更多上下文，尽量避免使用

  ```go
  // 定义相关上下文内容
  type MyError{
    context string
    msg string
  }
  func （m *MyError） Error() string{
    return &xxx
  }
  ```

  

- Opaque Errors 不透明处理错误 -- 耦合最少，通过内部处理  外部只需要判断err!=nil  提供对应的接口 传入进行处理

  ```go
  // 定义相关上下文内容
  type MyError{
    
  }
  func （m *MyError） Error() *error{
    return &error
  }
  func (m *MyError)  TimeOut(e error){
    //TODO 进行处理
  }
  
  ---
  err:=Error()
  if TimeOut(err){
    //TODO 
  }
  
  ```

  

### Error 处理

- 避免过多缩进

- 野生gorutinue 处理  （ 使用封装的GO方法）

  ```go
  package sync
  func Go(x func()){
    go func(){
      if err:=recover();err!=nil{
        //TODO something
        return
      }
      x()
    }
  }
  ---
  sync.GO(demo())
  ```

  

- 使用api 减少err判断 

- error wrap

### Error Wrap

- pkg/errors 包
- 五个规则  （总结一句话  调用第三方基础包或者最底层的错误 通过wrap err向上层抛 ，顶层出口负责处理，中间需要传递信息可以通过withMessage 处理）
  - 在你的应用代码中，使用 errors.New 或者  errors.Errorf 返回错误（业务层 需要抛出错误 ）
  - 如果调用其他的函数，通常简单的直接返回
  - 如果和其他库进行协作，考虑使用 errors.Wrap 或者 errors.Wrapf 保存堆栈信息。同样适用于和标准库协作的时候。
  - 直接返回错误，而不是每个错误产生的地方到处打日志
  - 在程序的顶部或者是工作的 goroutine 顶部(请求入口)，使用 %+v 把堆栈详情记录。

### GO1.3 Error

- 添加了IS,AS,Unwrap方法 可以结合pkg/errors使用
- %+w 打印 包装错误信息

### GO 2 Error

- https://go.googlesource.com/proposal/+/master/design/29934-error-values.md