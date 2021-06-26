---
layout: post
title: "LaTex Syntax"
date: 2020-12-04 22:00:00 +0800
tags: Algorithm
---

$$\Huge LaTex \quad Syntax$$

## 通用规则

- 空格会被忽略，添加空格有助于读者理解，需要显示时用`\quad`
- 特殊标识或对象以`\`开头
- 多个变量放在一组: `{放在}_{一组}`, ${放在}_{一组}$
- 行内函数`$f(x)$`, 行内函数$f(x)$
- 跨行函数`$$f(x)$$`
  跨行

  $$
  f(x)
  $$

  函数

- multiple lines
  ```
  b =
  \begin{cases}
      b_1, & \text{if}\ b_1 > 0  \\
      b_2, & \text{if}\ b_2 > 0  \\
      \frac{b_1 + b_2}{2} & \text{otherwise}
  \end{cases}
  ```
  $$
  b =
  \begin{cases}
      b_1, & \text{if}\ b_1 > 0  \\
      b_2, & \text{if}\ b_2 > 0  \\
      \frac{b_1 + b_2}{2} & \text{otherwise}
  \end{cases}
  $$

## 示例

| LaTex Syntax                        | Sample                                | Description               |
| ----------------------------------- | ------------------------------------- | ------------------------- |
| `a \quad b`                         | $$a \quad b$$                         | quad space                |
| `a \qquad b`                        | $$a \qquad b$$                        | double quad space         |
| `x_i`                               | $$x_i$$                               | subscript                 |
| `\text{text value}`                 | $$\text{text value}$$                 | text                      |
| `e^{i\pi}`                          | $$e^{i\pi}$$                          | upperscript               |
| `x_i^2`                             | $$x_i^2$$                             | subscript and upperscript |
| `\sqrt{2}`                          | $$\sqrt{2}$$                          | square root               |
| `\frac{1}{2}`                       | $$\frac{1}{2}$$                       | fraction                  |
| `\textstyle \sum_{i=1}^n w_ix_i`    | $$\textstyle \sum_{i=1}^n w_ix_i$$    | sum                       |
| `\displaystyle \sum_{i=1}^n w_ix_i` | $$\displaystyle \sum_{i=1}^n w_ix_i$$ | sum                       |
| `\because`                          | $$\because$$                          | because                   |
| `\therefore`                        | $$\therefore$$                        | therefore                 |
| `=`                                 | $$=$$                                 | equal to                  |
| `>`                                 | $$>$$                                 | great than                |
| `<`                                 | $$<$$                                 | less than                 |
| `\geqslant`                         | $$\geqslant$$                         | great than and equal to   |
| `\leqslant`                         | $$\leqslant$$                         | less than and equal to    |
| `\geq`                              | $$\geq$$                              | great than and equal to   |
| `\leq`                              | $$\leq$$                              | less than and equal to    |
| `\neq`                              | $$\neq$$                              | not equal to              |
| `\lVert w \rVert`                   | $$\lVert w \rVert$$                   | vertical                  |
| `\langle x, y \rangle`              | $$\langle x, y \rangle$$              | angle                     |
| `\underset{a}{max}`                 | $$\underset{a}{max}$$                 | under set                 |
| `\bar{\gamma}`                      | $$\bar{\gamma}$$                      | bar                       |
| `A \cdot B`                         | $$A \cdot B$$                         | center dot                |
| `A \cdots B`                        | $$A \cdots B$$                        | center dots               |
| `\infty`                            | $$\infty$$                            | infinity                  |

- absolute value `\left|x\right|`
  $$\left|x\right|$$

## 设置字体大小：

```
\tiny
\scriptsize
\footnotesize
\small
\normalsize
\large
\Large
\LARGE
\huge
\Huge
```
