# 二.第二周内容

Kotlin文件以 .kt 为后缀

首先创建一个.kt文件方便练手

<img width="182" height="233" alt="image" src="https://github.com/user-attachments/assets/42c13442-b8aa-48b9-9797-23c86c4ffbe7" /><img width="195" height="234" alt="image" src="https://github.com/user-attachments/assets/4bb1be17-4468-42cc-81ca-2134700a7e26" />
<img width="142" height="114" alt="image" src="https://github.com/user-attachments/assets/a948bdb8-8a22-47fe-9eca-f97bdfe88597" />

## 1.Kotlin 基础语法
### 1.1.包声明
package：几乎所有的package名字都对应文件系统上的文件夹路径，从根本上避免了类名冲突的问题，可以理解为命名空间。
import：使用其他地方的代码，于C++include功能类型，不同的是更高效，不需要“复制粘贴”，更类似于创建快捷方式，编译单元大小不变，类文件只需编译一次，其他模块直接引用即可，并且可以通过封装类的访问权限来控制使用范围
```kotlin
package com.example.myapplication
import android.os.Bundle
```
### 1.2.输入输出
可以通过print打印，这样是不会自动换行的
- println打印是会自动换行的
- readln可以读取用户输入

$：是字符串模板，用来在字符串中插入变量/表达式的值  
```kotlin
fun main() {
    val res = add(a = 3, b = 5)
    println("res is $res")
}
```
