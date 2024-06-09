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

Thrift编译器会根据服务定义生成全功能客户端以及服务端桩（stub）。**服务**的定义与在面向对象编程中定义接口的语义相同，通过如下的方式定义：

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



## 2. Thrift快速开始程序

快速开始程序完整代码：https://raw.githubusercontent.com/Ruijin-Shen/learn-Java/main/resources/thrift-demo.zip

### 2.1 引入Thrift依赖

```xml
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.15.0</version>
    <scope>compile</scope>
</dependency>
```

### 2.2 编写`user.thrift`文件

```idl
namespace java org.example

struct User {
    1:i32 id,
    2:string name
}

service UserService {
    bool addUser(1:User user),
    bool containsUser(1:i32 id),
    User getUserById(1:i32 id)
}
```

### 2.3 编译`user.thrift`文件

```bash
thrift -r -out src/main/java -gen java src/main/resources/thrift/user.thrift
```

### 2.4 实现服务端

类`UserServiceImpl`对`user.thrift`文件中定义的服务进行实现（实现`UserService.Iface`接口）。

```Java
public class UserServiceImpl implements UserService.Iface {
    private final ConcurrentHashMap<Integer, User> table = new ConcurrentHashMap<>();

    @Override
    public boolean addUser(User user) throws TException {
        System.out.println("method addUser is called");
        boolean result = !table.containsKey(user.getId());
        table.put(user.getId(), user);
        return result;
    }

    @Override
    public boolean containsUser(int id) throws TException {
        System.out.println("method containsUser is called");
        return table.containsKey(id);
    }

    @Override
    public User getUserById(int id) throws TException {
        System.out.println("method getUserById is called");
        return table.get(id);
    }
}
```

服务端`Server`

```Java
public class Server {
    public static void main(String[] args) {
        try {
            TServerTransport serverTransport = new TServerSocket(9090);
            TBinaryProtocol.Factory protocolFactory = new TBinaryProtocol.Factory();
            UserService.Processor<UserServiceImpl> processor = new UserService.Processor<>(new UserServiceImpl());

            TSimpleServer.Args targs = new TSimpleServer.Args(serverTransport);
            targs.protocolFactory(protocolFactory);
            targs.processor(processor);

            // Simple single-threaded server for testing
            TServer server = new TSimpleServer(targs);
            System.out.println("Starting the TSimpleServer server.");
            server.serve();
        } catch (TTransportException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### 2.5 客户端进行远程过程调用

客户端`Client`

```Java
public class Client {
    public static void main(String[] args) {
        TTransport transport = null;
        try {
            transport = new TSocket("localhost", 9090);
            TProtocol protocol = new TBinaryProtocol(transport);
            UserService.Client client = new UserService.Client(protocol);

            transport.open();
            // remote procedure call
            User alice = new User(1, "Alice");
            boolean isSuccess = false;
            isSuccess = client.addUser(alice);
            System.out.println("Insterting " + alice.toString() +  (isSuccess ? " succeeds" : " failed"));

            isSuccess = client.addUser(alice);
            System.out.println("Insterting " + alice.toString() +  (isSuccess ? " succeeds" : " failed"));

            System.out.println("User " + alice.toString() + (client.containsUser(alice.getId()) ? " exists" : " does not exists"));
            User bob = new User(2, "Bob");
            System.out.println("User " + bob.toString() + (client.containsUser(bob.getId()) ? " exists" : " does not exist"));

            User result = client.getUserById(1);
            System.out.println("User with id 1: " + result);
        } catch (TTransportException e) {
            throw new RuntimeException(e);
        } catch (TException e) {
            throw new RuntimeException(e);
        } finally {
            if (transport != null) transport.close();
        }
    }
}
```

## 3. Thrift网络栈

Thrift网络栈从下往上分别为：**传输层**（transport layer）、**协议层**（protocol layer）、 **处理器层**（processor layer）和**服务器层**（server layer）。

- **传输层**负责读取和写入数据，定义了数据如何在客户端和服务端之间传输（文件传输或网络传输）。

- **协议层**定义了数据传输格式，负责传输数据的序列化和反序列化，如 JSON、XML、纯文本、二进制和紧凑二进制等。
- **处理器层**由Thrift编译器生成，处理器从输入流读取数据，将处理委托给处理程序（由用户实现），并向输出流写入响应。
- **服务器层**整合上述所有功能：创建传输、为传输创建输入/输出协议、基于输入/输出协议创建处理器、等待连接并将其交给处理器。



## 4. 传输层（Transport Layer）

传输层定义了数据如何在客户端和服务端之间传输，Thrift默认使用基于TCP/IP的流式传输。

//TODO: 传输层内容



## 5. 协议层（Protocol Layer）

//TODO: 协议层内容



## 6. 处理器层（Processor Layer)

**处理器层**由Thrift编译器生成，处理器从输入流读取数据，将处理委托给处理程序（由用户实现），并向输出流写入响应。

处理器层的接口如下：

```Java
public interface TProcessor {
  public void process(TProtocol in, TProtocol out) throws TException;
}
```



## 7. 服务器层（Server Layer)

//TODO: 服务器层内容
