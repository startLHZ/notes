# protobuf嵌入方式

## SFINAE

- 编译器先匹配类模板全特化和类模板偏特化，然后再匹配普通模板
- 本质：if else  ,是在编译的时候确定下来的，函数/类，并且只要有一个函数或类匹配成功，就不会编译报错
- SFINAE是C++模板编译的一个原则，全名叫做Substitution Failure Is Not An Error （匹配失败不是错误）

### `std::enable_if`

- If B is true, `std::enable_if` has a public member typedef type, equal to T; otherwise, there is no member typedef.
- 第一个模板参数为 false 的时候并不会有成员变量：type。只有在第一模板参数为 true 的时候才会有定义成员变量：type

### `std::is_base_of`

- std::is_base_of 是 C++ 标准库 <type_traits> 中的一个模板类，用于判断一个类是否是另一个类的基类。
- 两个模版参数
- 有一个成员常量: value，用于表示第一个类型是否是第二个类型的基类。
- 如果第一个类型是第二个类型的基类，value 将为 true，否则为 false。

## 方法

利用函数模版编译时匹配的特性，给原 rosmsg 的模版加一个参数 enable = void。protobuf 的模板类判断参数类型是否以 protobuf 为基类，不是的话没有第二个 enable 参数，偏特化匹配失败，匹配 rosmsg 模版。匹配顺序，特化 -> 偏特化 -> 普通模版。

```C++
template <typename T, typename Enable = void>
struct Serializer {
    // rosmsg成员
}

// protobuf
template <typename T>
struct Serializer<T, typename std::enable_if<std::is_base_of<::google::protobuf::Message, T>::value>::type> {
    // protobuf成员
}
```