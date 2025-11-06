# Markdown 和 HTML 混合使用示例

---



## 1. 基础 Markdown 语法

### 文本样式

* **粗体**: `**这是粗体**` -> **这是粗体**
* **斜体**: `*这是斜体*` -> *这是斜体*
* **粗斜体**: `***这是粗斜体***` -> ***这是粗斜体***
* **删除线**: `~~这是删除线~~` -> ~~这是删除线~~

### 标题

# 一级标题
## 二级标题
### 三级标题
#### 四级标题

### 列表

#### 无序列表

* 列表项一
* 列表项二
    * 嵌套列表项
* 列表项三

#### 有序列表

1. 列表项一
2. 列表项二
    1. 嵌套列表项
3. 列表项三

---

## 2. 代码块和引用

### 行内代码

在文本中插入代码：`const myVar = 'Hello';`

### 代码块

```javascript
function greet(name) {
    console.log(`Hello, ${name}!`);
}

greet("World");
```

<p>
<span style="color: red;">这是一段红色的文字。</span>
<br>
<span style="background-color: #ffff00;">这是带黄色高亮的文字。</span>
</p>

<p>
<span style="font-size: 24px;">这是一个字号为 24px 的文字。</span>
<br>
<u>这是一段带有下划线的文字。</u>
</p>

<h3 align="center">图片居中对齐</h3>
<p align="center">
<img src="/pic/profile.png">
</p>
