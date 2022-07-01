# type 定义与 server 抽象  

<center> 日期: 2022/6/30 </center>

## http库 —— Request 概览
- **body 和 GetBody**  
  - Response 只有三个方法，分别是`Header()` `Write([]byte)` `WriteHeader(StatusCode int)`
  - Body: 只能读取一次，意味着你读了别人就不能读了；别人读了你就不能读了`body, err := io.ReadAll(r.Body)`  
  这种流的数据结构一般设计成只能读一次，如果需要多次读取可以使用`reset()`重置流
  - GetBody: 原则上是可以多次读取，但是在原生的 http.Request 里面， 这个是 nil
  - 在读取到 body 之后，我们就可以用于反序列化，比如说将 json格式的字符串转化为一个对象等  
  
- **Query**
   - 除了 Body，我们还可能传递参数的地方是 Query
   - 所有的值都被解释成字符串，所以需要自己解析为数字等别的数据类型  
  
- **URL**
   - URL 里面 Host 不一定有值
   - r.Host 一般都有值，是 Host 这个 header的值
   - RawPath 也是不一定有
   - Path肯定有值  

- **Header**
  - header 大体上是两类，一类是 http 预定义的; 一类是自己定义的
  - Go 会自动将 header 的名字转为标准名字 —— 其实就是大小写调整
  - 一般用 `X` 开头来表明是自己定义的， 不如说 `X-Auth-Token = This is a token`  
  
- **Form**
  - Form 和 ParseForm 
  - 要先调用ParseForm
  - 建议加上 `Content-Type:application/x-www-form-urlencoded`  
   
    
   ```
   func form(w http.ResponseWriter, r *http.Request) {

    err := r.ParseForm()

    if err != nil {
        fmt.Fprintf(w, "parse form error %v\n", r.Form)
    }

   fmt.Fprintf(w, "before parse form %v\n", r.Form)
  }  

- ...

**要点总结: http 库使用**  
- Body 和 GetBody: 重点在于Body是一次性的， 而 GetBody 默认情况下是没有，一般中间件会考虑帮你注入这个方法
- URL: 注意 URL 里面的字段含义可能并不如你所期望的那样
- Form: 记得调用前先用 ParseForm ，别忘了请求里面加上 http 头

## 基础语法type
- **type 定义**
    - type 名字 interface {}    
        - 里面只能有方法，方法也不需要 func 关键字  
        - 什么是接口 (interface) ： 接口是一组行为的抽象  
        - 尽量用接口，以实现面向接口编程  

    - type 名字 struct {}  
        - 基本语法 :  
        ``` 
        type Name Struct {
            fieldName FieldType
            // ...
        }  
        ```  
        - 结构体和结构体的字段都遵循大小写控制访问性的原则  

    - type 名字 别的类型  
        - 例如 :  
        `1. type Header map[string][]string`  
        `2. type A B`
            - 基本语法 : type TypeA TypeB 
            - 一般是，在我们使用第三方库又没有办法修改源码的情况下，又想在扩展这个库的结构体方法，就会用这个，例如 ：  

            ``` 
            type Fish struct {
            }

            func (f Fish) Swim() {
                fmt.Println("我会游泳")
            }

            // 定义了一个新类型，注意是新类型
            type FakeFish Fish 

            func (f FakeFish) FakeSwim() {
                fmt.Println("我假装我会游泳")
            }

            // 定义了一个新类型
            type StrongFish Fish   
            func (f StrongFish) Swim() {
                fmt.Println("我是山寨鱼，但我也会游泳")
            }  

            fake := FakeFish{}
            // fake 无法调用原来 Fish 的方法
            // 这一句会编译错误
            // fake.Swim()
            fake.FakeSwim()

            // 转换为 Fish
            td := Fish(fake)
            // 真的变成了鱼 (相当于类型强转)
            td.Swim()

            sFake := StrongFakeFish{}
            // z这里就是调用了自己的方法
            sFake.Swim()

            td = Fish(sFake)
            // 真的变成了鱼
            td.Swim()

            ```
    - type 别名 别的类型 

- **结构体初始化**  
    - go没有构造函数
    - 初始化语法：Struct{}
    - 获取指针： &Struct{}
    - 获取指针2： new(Struct)
    - new 可以理解为 go 会为你的变量分配内存，并且把类型都置为 0 

- **指针与方法接收器**  

- **结构体如何实现接口**

## Server 与 Context 抽象
- Http Server 抽象  
    我想要一个Server的东西，表达一种逻辑上的抽象，它代表的是对某个端口的进行监听的实体，必要的时候，我可以开启多个Server，来监听多个端口   

    **封装一个简单的http Server :** 
```
      type Server interface {  

        // Route 设定一个路由，命中该路由的会执行handlerFunc的代码
        Route(pattern string, handlerFunc http.HandlerFunc)

        // Start 启动我们的服务器
        Start(address string) error

      }  

```  

## 简单支持 RESTFul API