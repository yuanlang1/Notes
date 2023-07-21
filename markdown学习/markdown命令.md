# markdown 语法  

$生成目录L：使用快捷键command+shift+P（windows用户ctrl+shift+P），输入Markdown All in One: Create Table of Contents并回车$

- [markdown 语法](#markdown-语法)
  - [一级标题](#一级标题)
  - [二级标题](#二级标题)
    - [三级标题](#三级标题)
    - [四级标题](#四级标题)

## 一级标题

## 二级标题  

### 三级标题  

### 四级标题  

**使用标题**：使用 **#**，随着标题的级别降低，而增加  
写完一个段落就要空一行

---

**分割线**  **---**，使用三条横线  
**重点加粗**  $**中间**$，四个星号的中间，可以使用快捷键：$ctrl+b$  
***斜体*** $*中间*$，两个星号的中间，可以使用快捷键:$ctrl+i$  
**~~删除线~~** 两个波浪线的中间  
Markdown Preview Enhanced 拓展功能: 使用==高亮== 将需要高亮的部分放在四个=行的中间即可

---

**引用文本**:

> 引用别人的话  
> 就是这样写的  
>
> - 111
> - 222

$<... $ 加上说的话就可以了  
$< - ...$ 在$<$ 的级别下，挑一行后，即为嵌套  

---

**行间代码**： 两个引号的中间，即为行间代码  
`#include <bits/stdc++.h>`

**代码块**：六个引号的中间，记得空行  在开头的三个引号后面添加对应语言，{}括号中添加{.line-numbers} 可以显示行号  

```c++ {.line-numbers}
#include <iostream>
using namespace std;
int main()
{
    cout<<"hello world"<<endl;
    return 0;
}
```

---

**列表**:

``` {.line-numbers}

* 无序列表
  * 嵌套无序列表
  * 嵌套无需列表
* 无序列表
* 无序列表

1. 有序列表
    1.1 嵌套无序列表
    1.2 嵌套无序列表
2. 有序列表
3. 无序列表

Markdown Preview Enhanced 拓展功能：
任务列表：
* [x] 已经完成的事 1
* [x] 已经完成的事 2
* [ ] 未完成的事

```

**下面为演示**：

- 无序列表
  - 嵌套无序列表
  - 嵌套无序列表
- 无序列表
  
1. 有序列表
    1.1 嵌套有序列表
    1.2 嵌套有序列表
2. 有序列表

任务列表拓展：

- [x] 已完成的事件  
  - [x] 嵌套事件  
- [x] 已完成的事件
  - [ ] 嵌套事件  
- [ ] 未完成的事件  

---

**超链接和图片**:

```{.line-numbers}
[超链接名称](超链接地址)
![图片提示语](图片地址)

例如, 可以使用网址和图床:
[OrangeX4's Blog](https://orangex4.cool/)
![OrangeX's Avatar](https://orangex4.cool/images/icons/profile.jpg)

也可以在本地用相对地址:
[Other](other.md)
![OrangeX's Avatar](images/profile.jpg)  

```  

**下面为演示**:

[我的笔记](https://notes.orangex4.cool/?git=github&github=yuanlang1/Notestoken=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)  

**图片**: paste image 插件下,直接使用$ctrl+alt+v$ 即可将图片保存到images文件夹中,并生成地址  

![图片](images/2023-07-20-21-35-08.png)  

$使用尖括号可以很方便地把URL或者email地址变成可点击的链接$
$<fake@example.com>$:<fake@example.com>  
$<https://markdown.com.cn>$:<https://markdown.com.cn>

---

**表格**:  

```{.line-numbers}

表格:
| 表头 | 表头 |
| ---- | ---- |
| 内容 | 内容 |
| 内容 | 内容 |

快捷键: shift + alt + f 自动表格对齐
| 左对齐 | 右对齐 | 居中对齐 |
| :----- | -----: | :------: |
| 内容   |   内容 |   内容   |
```

**一下为上面的演示**:

表格:
| 表头 | 表头 |
| ---- | ---- |
| 内容 | 内容 |
| 内容 | 内容 |

| 左对齐 | 右对齐 | 居中对齐 |
| :----- | -----: | :------: |
| 内容   |   内容 |   内容   |
