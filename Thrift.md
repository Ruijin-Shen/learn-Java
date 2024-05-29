# Thrift

**Thrift**是一个跨语言的**RPC框架**（Remote Procedure Call，**远程过程调用**）框架。Thrift使用**IDL**（Interface Definition Language，接口定义语言）定义数据类型和服务接口。



## 1. 类型

Thrift 类型系统不引入任何特殊的动态类型或包装对象。

### 1.1 基础类型（Base Types）

Thrift支持以下几种**基本类型**：

- `bool` 布尔值，true或false
- `byte` 有符号字节
- `i16` 16位有符号整数
- `i32` 32位有符号整数
- `i64` 64位有符号整数
- `double` 64位浮点数
- `string` 与编码无关的文本或二进制字符串
- `binary` 未编码的字节序列，是`string`的一种特殊形式，对应Java的`Java.nio.ByteBuffer`

### 1.2 集合容器（Containers）

Thrift的**集合容器**是强类型的（使用时必须指定泛型）。在Java中，泛型的基本类型会被替换为对应的包装类型。Thrift支持以下几种容器：

- `list<type>`  元素有序可重复的列表，默认对应C++的`vector`或Java的`ArrayList`
- `set<type>` 元素无序不可重复的集，默认对应C++的`set`或Java的`HashSet`
- `map<type1, type2>` 唯一键到值的映射，默认对应C++的`map`或Java的`HashMap`

容器元素可以是任何 Thrift 类型，包括其他容器或结构体。

### 1.4 结构体（struct）

Thrift的**结构体**定义了跨语言使用的通用对象。结构体有一组**强类型字段**，每个字段都有一个唯一的名称标识符。字段可以用**整数字段标识符**和可选的默认值进行注解。

```idl
struct User {
	1:i32 id,
	2:string name="thrifty",
}

struct Example {
	1:i32 number=10,
	2:map<string, User> usermap
}
```

可用`optional`或`required`修饰字段。如果`optional`字段没有设置值，那么这个字段就不会被序列化和反序列化；如果`required`字段没有设置值，那么序列化和反序列化操作会抛出异常。

每个定义都会在目标语言中生成一个包含`read`和`write`方法的类型，这两个方法使用Thrift的TProtocol对象执行对象的序列化和传输。

### 1.5 枚举（enum）

可用`enum`关键字声明枚举类型，枚举常量必须是32位的整数。如果你没有为枚举常量指定值，那么Thrift会自动为它分配一个值，这个值是上一个枚举常量的值加1，第一个枚举常量的默认值是0。

```idl
enum HttpStatus {
	OK=200,
	NOTFOUND=404
}
```

### 1.6 异常（Exception）

**异常**在语法和功能上与结构体相同，但异常使用 `exception` 关键字而不是` struct `关键字声明。**生成的异常对象继承目标语言的异常基类**。

### 1.7 服务（Service）

Thrift编译器会根据服务定义生成全功能客户端以及服务端桩（stub）。服务的定义与在面向对象编程中定义接口的语义相同，通过如下的方式定义：

```idl
service <name> {
	<returntype> <name>(<arguments>) [throws (<exceptions>)]
	...
}

service StringCache {
	void set(1:i32 key, 2:string value),
	string get(1:i32 key) throws (1:KeyNotFound knf),
	void delete(1:i32 key)
}
```

可用`async`关键字修饰返回值为`void`的函数，使得函数无需等待服务端的响应。纯`void`函数会向客户端返回一个响应，以确保服务器端的操作已经完成。`async`方法调用只能保证请求在传输层（transport layer）成功。因此，只有在传输可靠或可以接受方法调用丢失的情况下才使用异步优化。



## 2. Thrift的架构

Thrift的架构分为如下几层：
