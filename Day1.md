# C++ 14 天学习计划｜Day 1 学习总结

> **主题：** 从 C 语言过渡到 C++ 基础  
> **学习目标：** 掌握 C++ 基本输入输出、`string`、函数重载和 `bool`，能够独立编写简单程序，为后续学习数据结构做准备。  
> **建议用时：** 1.5～2 小时  
> **当前状态：** 已提供课程；练习尚未提交，掌握程度待验证。

## 1. 第一个 C++ 程序

C 语言常用 `stdio.h`、`printf`；C++ 可以使用 `iostream`、`cout`。

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello World" << '\n';
    return 0;
}
```

- `#include <iostream>`：引入标准输入输出功能。
- `std`：标准库命名空间；学习阶段可以写 `using namespace std;`，也可以明确写 `std::cout`。
- `cout`：向标准输出写入内容；`cin`：从标准输入读取内容。
- `\n`：换行；一般输出时可优先使用它，而不是每次都用 `endl`（后者还会刷新输出缓冲区）。

## 2. `cout`：输出

| C 语言 | C++ |
|---|---|
| `printf("age = %d\n", age);` | `cout << "age = " << age << '\n';` |

```cpp
int age = 18;
double height = 175.5;
cout << "age = " << age << ", height = " << height << '\n';
```

**要点：** `<<` 可连续连接输出内容，无需为普通整数和浮点数分别写 `%d`、`%f`。

## 3. `cin`：输入

| C 语言 | C++ |
|---|---|
| `scanf("%d", &a);` | `cin >> a;` |

```cpp
int a, b;
cin >> a >> b;
cout << a + b << '\n';
```

示例输入：

```text
10 20
```

输出：

```text
30
```

**注意：** `cin >>` 默认以空白字符（空格、换行等）分隔输入。

## 4. `string`：更方便的字符串

C 常用字符数组 `char name[100]`；C++ 标准库提供 `std::string`。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string name;
    cin >> name;
    cout << "Hello " << name << '\n';
}
```

常用操作：

```cpp
string first = "Hello";
string second = "World";
string result = first + " " + second; // 拼接
cout << result << '\n';             // Hello World
cout << result.length() << '\n';    // 11
cout << result[0] << '\n';          // H
```

遍历字符串：

```cpp
string s = "hello";
for (int i = 0; i < static_cast<int>(s.size()); ++i) {
    cout << s[i] << '\n';
}
```

**要点：** `s[i]` 可以访问字符；索引从 `0` 开始。`cin >> s` 只能读取一个不含空白的词，整行文本以后可以用 `getline(cin, s)`。

## 5. 函数与函数重载

普通函数的写法和 C 语言类似：

```cpp
int add(int a, int b) {
    return a + b;
}
```

C++ 支持**函数重载**：同名函数可以通过不同的参数列表区分。

```cpp
int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}
```

```cpp
cout << add(1, 2) << '\n';     // 3
cout << add(1.5, 2.7) << '\n'; // 4.2
```

**注意：** 不能仅靠返回值类型不同来构成函数重载。

## 6. `bool`：布尔类型

`bool` 只有两个逻辑值：`true` 和 `false`。

```cpp
bool isAdult(int age) {
    return age >= 18;
}
```

```cpp
int age;
cin >> age;
if (isAdult(age)) {
    cout << "成年" << '\n';
} else {
    cout << "未成年" << '\n';
}
```

以后实现数据结构时，常用 `bool empty()` 表示容器是否为空，或用 `bool find(...)` 表示是否找到目标。

## 7. 今日练习（暂不附答案）

### 题目 1：基本运算

输入两个整数 `a`、`b`，依次输出 `a+b`、`a-b`、`a*b`，每个结果单独占一行。必须使用 `cin`、`cout`。

输入：

```text
10 3
```

输出：

```text
13
7
30
```

### 题目 2：个人信息

输入一个不含空格的姓名和一个年龄，按指定格式输出。

输入：

```text
Tom
18
```

输出：

```text
Hello Tom
You are 18 years old.
```

要求使用 `string name;` 和 `int age;`。

### 题目 3：两个数中的较大值

实现函数：

```cpp
int maxNumber(int a, int b);
```

输入 `10 20`，输出 `20`。不得使用现成的 `max()`。

### 题目 4：判断质数

实现函数：

```cpp
bool isPrime(int n);
```

输入一个整数，若是质数输出 `YES`，否则输出 `NO`。注意 `n <= 1` 不属于质数。

### 题目 5（Boss）：统计字符

输入一个不含空格的字符串，统计字符 `'l'` 出现的次数。

输入：

```text
hello
```

输出：

```text
2
```

提示：遍历 `string`，逐个检查 `s[i]`。

## 8. 完成标准与后续安排

- [ ] 能独立写出使用 `cin`、`cout` 的完整程序。
- [ ] 能声明、拼接和遍历 `string`。
- [ ] 理解普通函数、函数重载和 `bool` 返回值。
- [ ] 完成以上 5 道练习，并检查边界情况。

**当前练习进度：0/5 已提交批改。** 完成后可进入 **Day 2：引用 `&`、`const`、`vector`**。

> **学习方法：** 每天先读知识点并运行示例，再关闭参考资料独立做题，最后提交代码检查错误。建议约 30 分钟学概念、60 分钟写代码、30 分钟练习与复盘。
