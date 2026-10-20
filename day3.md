# Day 3 学习总结：C++ 指针与动态内存管理

> 14 天 C++ 速成 + 数据结构入门计划  
> 学习进度：Day 1 ✅ / Day 2 ✅ / **Day 3 ✅（4/4 题通过）**  
> Day 3 主题：指针、`nullptr`、`new/delete`、动态数组与内存管理

## 一、四部分内容回顾

| 部分 | 知识点 | 需要掌握的核心 |
|---|---|---|
| 第 1 部分 | 指针、解引用、指针与引用 | 分清地址和值；引用不能重新绑定 |
| 第 2 部分 | `nullptr`、指针安全、栈与堆、`new/delete` | 指向有效对象才能解引用；谁分配、谁正确管理 |
| 第 3 部分 | `new[]/delete[]`、`size/capacity`、扩容 | 分配新空间 → 复制数据 → 释放旧空间 → 更新指针 |
| 第 4 部分 | 总结与四道编程题 | 独立实现可扩容整数数组管理器 |

## 二、指针、引用与 `nullptr`

```cpp
int a = 10;
int* p = &a;    // p 保存 a 的地址
int& r = a;     // r 是 a 的引用（别名）

*p = 20;        // a 变成 20
r = 30;         // a 变成 30

int b = 40;
p = &b;         // p 可以改为指向 b
r = b;          // 不是重新绑定：把 b 的值赋给 a
```

- `a`：整数值；`&a`：`a` 的地址；`p`：指针保存的地址；`*p`：指针所指对象的值；`&p`：指针变量自己的地址。
- `int* p` 表示指针；`int& r` 表示引用；`int*& p` 表示**指针变量的引用**，可在函数中改变调用者的指针。
- `nullptr` 表示空指针：`int* p = nullptr;`。**不能解引用空指针**。
- `p != nullptr` 只表示非空，不能保证它所指对象仍然存活。释放后留下的旧指针可能成为**悬空指针**。
- 指针不需要动态分配才能使用，例如 `int* p = &a;` 指向普通局部变量，此时**不能**对 `p` 使用 `delete`。

### 函数参数选型

| 函数需求 | 推荐参数示例 | 原因 |
|---|---|---|
| 读取数组元素 | `const int* arr, int size` | 不修改数据、不修改外部指针 |
| 修改数组元素 | `int* arr` | 能通过指针修改所指元素 |
| 修改外部整数变量 | `int& size` | 同步更新调用者状态 |
| 改变调用者指针指向 | `int*& arr` | 扩容后能更新外部指针 |

## 三、动态内存与 C/C++ 对比

| 用途 | C 风格 | C++ 手动管理 |
|---|---|---|
| 分配单个整数对象 | `malloc(sizeof(int))`（分配原始内存） | `new int{10}` |
| 分配整数数组 | `malloc(n * sizeof(int))` | `new int[n]{}` |
| 释放 | `free(ptr)` | 单个对象 `delete ptr`；数组 `delete[] ptr` |
| 扩展存储空间 | `realloc`（对其支持的分配使用） | 新建更大数组、复制、删除旧数组 |

**分配与释放必须正确匹配：**

```cpp
int* one = new int{10};
delete one;
one = nullptr;

int* arr = new int[5]{};
delete[] arr;
arr = nullptr;
```

- `new int[n]{}` 将 n 个整数初始化为 0；`new int[n]` 不会自动把这些整数初始化为 0。
- 普通局部变量一般具有自动存储期，离开作用域后生命周期结束；`new` 创建的动态对象需要正确管理其释放。栈/堆只是常用的理解模型，C++ 标准并不强制所有局部变量在物理栈上。
- `delete` 后当前指针**不会自动**变成 `nullptr`；设置空指针不能自动修复其他指针中保存的旧地址。
- 不要混用 `new/free`、`malloc/delete`、`new[]/delete`。
- `realloc` 不能用于 `new[]` 分配的空间，也不适合通用 C++ 对象的管理。
- 普通 `new` 分配失败通常抛出 `std::bad_alloc`；`malloc` 分配失败返回空指针。

## 四、动态数组：`size`、`capacity` 和扩容

- **`size`**：实际有效元素数量。
- **`capacity`**：已经分配、可容纳的整数个数。
- 始终维持 `0 <= size <= capacity`。
- 例：`[10, 20, 30, _, _]`，则 `size = 3`、`capacity = 5`。
- 删除元素只需移动后面的有效元素并递减 `size`；**本题不要求缩容**。
- 当 `size == capacity` 且准备添加元素时，需要先扩容；常见策略是容量翻倍（如 `2 → 4 → 8`）。
- 扩容步骤：**确定新容量 → `new[]` 申请新空间 → 复制前 `size` 个元素 → `delete[]` 旧空间 → 更新指针与容量 → 添加元素、更新 `size`**。
- 一次扩容可能复制 O(n) 个元素；倍增策略下，连续末尾添加的摊还时间复杂度为 O(1)。
- 扩容复制的是元素的**值**，不是旧地址；旧数组释放后，指向旧元素的指针或引用都会失效。
- `sizeof(arr)` 对动态数组指针得到的是指针类型大小，**不是数组长度**；需要自己维护长度与容量。

### 两个关键边界条件

1. **删除元素**：合法下标为 `0 <= index < size`。若要执行 `arr[i] = arr[i + 1]`，循环只应覆盖需要前移的元素，不能读取 `arr[size]`。
2. **调整到长度 0**：先处理零长度特殊情况，释放旧数组，设置 `arr = nullptr`，并将 `length = 0`，不要事先无意义地 `new int[0]` 后又遗忘释放。

## 五、本次练习成果与纠错记录

| 题目 | 结果 | 关键收获 |
|---|---|---|
| 1. 指针交换两个整数 | ✅ 通过 | 临时保存整数值应用 `int temp`；不能解引用 `nullptr`；空指针测试要真正传入空指针 |
| 2. 动态数组统计器 | ✅ 通过 | `new[]/delete[]` 配对；最大值用 `arr[0]` 初始化以支持全负数；逆序输出不必额外分配数组 |
| 3. 动态数组长度调整 | ✅ 通过 | `int*&` 更新外部指针；先复制再释放；调整为零长度时指针与长度必须同步更新 |
| 4. Boss：可扩容整数数组管理器 | ✅ 通过 | 自行完成添加、扩容、删除、查找、输出、状态查看、退出释放和菜单控制 |

### Boss 题特别容易错的地方

- 删除时，原先写成循环中始终使用固定下标 `x`，后来已改为使用 `i`。
- 删除循环原先可能读取 `arr[size]`，后来通过正确的边界 `i < size - 1` 修复。
- 查询函数返回了下标，主程序也必须接收并输出结果。
- `switch` 的 `case` 标签**不会自动创建独立作用域**；在 `case` 内声明并初始化局部变量，必要时用 `{ ... }` 包住分支。
- 删除非法下标要提示失败，不能让 `size` 无条件减一。
- `new[]` 必须与 `delete[]` 配对，退出程序之前要释放动态数组。

## 六、Boss 题：最终提交的完整源码（原样留档）

> 以下是 Day 3 最后一次提交的版本，便于日后复制、复习、对照。按本题前提（从容量 2 开始、整数输入有效、申请成功、容量不会溢出）完成核心功能。实际项目应进一步处理输入失败、分配异常和容量溢出。

```cpp
#include <iostream>
using namespace std;

void addInt(int x, int *&arr, int &capacity, int &size)
{
    if (size == capacity)
    {
        capacity *= 2;
        int *newarr = new int[capacity]{};
        for (int i = 0; i < size; i++)
        {
            newarr[i] = arr[i];
        }
        delete[] arr;
        arr = newarr;
    }
    arr[size] = x;
    size++;
}

bool deleteInt(int x, int *&arr, int &size)
{
    if (x < 0 || x > size - 1)
    {
        return false;
    }
    for (int i = x; i < size - 1; i++)
    {
        arr[i] = arr[i + 1];
    }
    size--;
    return true;
}

int searchInt(int x, const int *arr, int size)
{
    for (int i = 0; i < size; i++)
    {
        if (arr[i] == x)
        {
            return i;
        }
    }
    return -1;
}

void printInt(int *&arr, int &size)
{
    if (size == 0)
    {
        cout << "数组为空" << endl;
        return;
    }
    for (int i = 0; i < size; i++)
    {
        cout << arr[i] << ' ';
    }
    cout << endl;
}

void checkStatus(int size, int capacity)
{
    cout << "size=" << size << ' ' << "capacity=" << capacity << endl;
}

void exitprogram(int *&arr)
{
    delete[] arr;
    arr = nullptr;
}

int main()
{
    int size = 0;
    int capacity = 2;
    int *arr = new int[capacity]{};
    cout << "1:添加整数" << endl;
    cout << "2:删除指定下标" << endl;
    cout << "3:查找整数" << endl;
    cout << "4:输出数组" << endl;
    cout << "5:查看 size 和 capacity" << endl;
    cout << "0:退出程序" << endl;
    int x;
    while (true)
    {
        int n;
        cin >> n;
        switch (n)
        {
        case 1:
            cin >> x;
            addInt(x, arr, capacity, size);
            break;
        case 2:
        {
            cin >> x;
            bool y = deleteInt(x, arr, size);
            if (!y)
            {
                cout << "删除失败" << endl;
            }
            break;
        }
        case 3:
        {
            cin >> x;
            int m = searchInt(x, arr, size);
            if (m == -1)
            {
                cout << "元素不存在" << endl;
            }
            else
            {
                cout << "下标是" << m << endl;
            }
            break;
        }
        case 4:
            printInt(arr, size);
            break;
        case 5:
            checkStatus(size, capacity);
            break;
        case 0:
            exitprogram(arr);
            return 0;
        default:
            cout << "数字无效" << endl;
            break;
        }
    }
    return 0;
}
```

## 七、Day 3 完成标志与 Day 4 衔接

- [x] 理解 `p`、`*p`、`&a`、引用和指针引用。
- [x] 理解 `nullptr`、未初始化指针、悬空指针和释放后使用的风险。
- [x] 能够正确使用 `new/delete`、`new[]/delete[]`。
- [x] 能够区分 `size` 与 `capacity` 并实现数组倍增扩容。
- [x] 四道编程练习完成并通过。

**下一步指令：** `开始 Day 4`

继续遵守教学要求：一次只学习一天，每个知识点包含 C/C++ 对比、完整代码、运行结果和解释；每天 1.5～2 小时、3～5 道分级练习；不提前公布练习答案；交代码后逐行批改，先给修改提示。
