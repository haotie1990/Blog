
### Markdown操作手册

***
#### 1. 标题

标题采用 **\#\+空格\+文字** 的方式

    # 一级标题
    ## 二级标题
    ### 三级标题

#### 2. 段落和换行

```
一个段落或者需要另起一行必须要有：空行
第一行
（中间空一行）
第一行
```

#### 3. 列表

>无序的列表使用符号： __\*&nbsp;&nbsp;&nbsp;&nbsp;\+&nbsp;&nbsp;\-&nbsp;&nbsp;后面跟空格__

    * 第一行
    * 第二行

    + 第一行
    + 第二行

    - 第一行
    - 第二行

>有序的列表使用： **数字&nbsp;&nbsp;&nbsp;&nbsp;**+&nbsp;&nbsp;&nbsp;&nbsp;**.**&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;&nbsp;&nbsp;&nbsp;**空格**

    1. 第一行
    2. 第二行

#### 4.强调
**加黑**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*斜体*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;==高亮==&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;~~删除线~~&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;++下划线++

```
使用两个*或者_包围是加黑：**加黑**  __加黑__

使用一个*或者_包围是斜体: *斜体*    _斜体_

使用两个=包围是高亮：==高亮==

使用两个~包围是删除线：~~删除线~~

使用两个+包围是下划线：++下划线++

```

#### 5. 分隔线

---

***

    分隔线采用三个连续的-或者*字符

#### 6.代码和代码块
`public void main(String args){};`
```Java
public void main(String args){
    System.out.printfln("this is code region");
}
```
    对于简单的代码可以使用`包围
    如果是代码块则可以使用Tab（四个空格）或者三个`包围（```）
    其中可以采用如下方式，实现代码语法高亮
    ```Java(代码语言)
    ```

#### 7. 图片和链接

![图片](https://upload.wikimedia.org/wikipedia/commons/4/48/Markdown-mark.svg)

[**百度**](https://www.baidu.com)

```
图片的插入格式为：![imagetile](imageurl)
中括号为图片的描述文字,普通括号里为图片的链接地址

链接的格式为：[text](url)
中括号里为链接的文字,普通括号里为链接地址
```

#### 8. 表格

|标题|标题|标题|
|------|----|----------|
|普通文本|短文本|长文本|
```text
|标题|标题|标题|
|------|----|----------|
|普通文本|短文本|长文本|
```

| 左对齐 | 右对齐 | 居中对其 |
| :------ | ----: | :----------: |
|用很长的内容显示|用很长的内容显示|用很长的内容显示|
```
| 左对齐 | 右对齐 | 居中对其 |
| :------ | ----: | :----------: |
|用很长的内容显示|用很长的内容显示|用很长的内容显示|
```

`-:`表示右对齐
`:-`表示左对齐
`:-:`表示居中对其


#### 9. 层级或区域

>* 第一层级

>>* 第二层级

```
>* 第一层级

>>* 第二层级
```

>这是一块区域
>
>这块区域有竖线标注
>
>只有>的行可形成空行

