# Scheme 代码风格指南

> 基于 Agentic Interface 思想的严格定义

## 开发环境

我们所采用的 Scheme 解释器是 **Goldfish Scheme**，对应的命令行是 `gf`。

### 版本信息

当前使用的 Goldfish Scheme 版本：**v17.11.37**

在基于 Goldfish Scheme 撰写项目时，不需要再额外拉取 Goldfish 源码到临时目录中查找库定义、测试或示例。
优先直接使用 `gf doc` 和 `gf source` 来获取相应库的介绍与源码。

### 常用库查询方式

常用命令如下：

```bash
gf doc liii/list
gf doc liii/list "append"
gf doc "append"
gf source liii/list
```

说明：

- `gf doc ORG/LIB`：查看一个库的整体介绍
- `gf doc ORG/LIB FUNC`：查看某个函数在指定库下的文档和测试用例
- `gf doc FUNC`：按函数名全局查找文档
- `gf source ORG/LIB`：直接查看对应库的源代码

### 项目骨架

Scheme 的软件包是两层结构的，如果包名是 `(orgid libid)`，那么项目的骨架是：

```
.
orgid/
  libid.scm
tests/
  orgid/
    libid-test.scm
```

### 常用命令

**检查括号匹配**：
```bash
gf fix orgid/libid.scm
gf fix tests/orgid/libid-test.scm
```

**运行测试**：
```bash
gf test                          # 运行所有测试
gf test --only libid             # 运行 tests 目录下，测试文件中含有 libid 的测试
gf test --only libid-test.scm    # 运行 tests 目录下，测试文件的文件名是 libid-test.scm 的测试
gf tests/orgid/libid-test.scm    # 直接运行单个测试
```

---

## 目录

1. [概述](#1-概述)
2. [命名规范](#2-命名规范)
3. [代码格式](#3-代码格式)
4. [代码组织](#4-代码组织)
5. [注释规范](#5-注释规范)
6. [函数设计](#6-函数设计)
7. [测试规范](#7-测试规范)
8. [示例代码](#8-示例代码)
9. [参考](#9-参考)

---

## 1. 概述

本指南定义三鲤网络（Liii Network）所有 Scheme 代码的编写规范。它基于 **Agentic Interface** 指导思想，旨在提升代码的可读性、可维护性和 AI 生成准确性。

### 1.1 S 表达式的 Agentic Interface

**S 表达式（Symbolic Expression）**：Scheme 语言的基本语法单元，由括号包围的列表或原子组成。

```scheme
;; S 表达式的两种形式
123              ; 原子（atom）
(+ 1 2 3)        ; 列表（list）：括号包围的多个元素
```

每个 S 表达式列表都是一个**环境（Environment）**。

**环境的定义**：左括号 `(` 和与之匹配的右括号 `)` 所包含的代码块。

```scheme
;; 这是一个环境
(define (foo x)
  (+ x 1)
) ;define
;; ↑          ↑
;; 左括号    右括号（匹配）
```

**Agentic Interface 的核心**：为每个环境提供**显式的左侧标记和右侧标记**，让代码结构一目了然。

**左侧标记（Left Tag）**：环境开始的标记，由 `(` + 环境类型组成。
```scheme
(define (foo x)    ; (define 是左侧标记
  (+ x 1)
) ;define

(let ((x 1))       ; (let 是左侧标记
  (+ x 2)
) ;let
```

**右侧标记（Right Tag）**：环境结束的标记，由 `) ;` + 环境类型组成。
```scheme
(define (foo x)
  (+ x 1)
) ;define        ; ) ;define 是右侧标记

(let ((x 1))
  (+ x 2)
) ;let           ; ) ;let 是右侧标记
```

**标记规则**：
- **左侧标记**：`(` + 环境类型（如 `define`, `let`, `if`）
- **右侧标记**：`) ;` + 环境类型（注意：`;` 左边一个空格，右边**没有**空格）
- 环境跨行时，右括号**必须**带右侧标记
- 环境不跨行（单行）时，不需要右侧标记

```scheme
;; ✅ 正确：跨行必须带右侧标记
(define (foo x)
  (+ x 1)
) ;define

;; ❌ 错误：跨行了但缺少右侧标记
(define (foo x)
  (+ x 1))

;; ✅ 正确：不跨行，单行形式
(define (foo x) (+ x 1))
```

### 1.2 核心原则

#### 1.2.1 显式环境（Environment）

**原则**：
- 标记所在的左括号在行首
- 通过左侧标记和右侧标记的匹配，让代码结构显式呈现
- 所有跨行的环境都必须有右侧标记

```scheme
;; 嵌套结构：每层都标记
(define (foo x y)
  (let ((sum (+ x y)))
    (if (> sum 0)
      (positive-case sum)
      (negative-case sum)
    ) ;if
  ) ;let
) ;define
```

#### 1.2.2 扁平化结构

嵌套深度越少越好，避免深层嵌套导致可读性下降。

```scheme
;; ❌ 错误：深层嵌套难以阅读
(define (foo x)
  (bar (baz (qux x)))
) ;define

;; ✅ 正确：简单代码直接嵌套
(define (foo x)
  (bar (baz (qux x)))
) ;define

;; ❌ 错误：复杂代码深层嵌套难以阅读
(define (process data)
  (reduce combine (map transform (filter valid? data)))
) ;define

;; ❌ 错误：嵌套 let 增加嵌套深度
(define (process data)
  (let ((filtered (filter valid? data)))
    (let ((transformed (map transform filtered)))
      (reduce combine transformed)
    ) ;let
  ) ;let
) ;define

;; ✅ 正确：使用 let* 扁平化结构
(define (process data)
  (let* ((filtered (filter valid? data))
         (transformed (map transform filtered)))
    (reduce combine transformed)
  ) ;let*
) ;define
```

**原则**：
- 简单代码直接嵌套，保持简洁
- 复杂代码（难以一眼看清结构时）拆分为扁平结构
- 目标是可读性，根据复杂度灵活处理

#### 1.2.3 统一缩进

使用 2 空格缩进，不使用 Tab。

**缩进层级示例**（以函数定义为例）：

```scheme
;; 第0层（0空格）：顶层定义
(define (factorial n)
  ;; 第1层（2空格）：控制结构
  (if (= n 0)
    ;; 第2层（4空格）：分支体
    1
    ;; 第2层（4空格）：分支体
    (* n (factorial (- n 1)))
  ) ;if
) ;define
```

**通用缩进层级表**：
- 第0层（0空格）：顶层定义（define, define-library 等）
- 第1层（2空格）：控制结构（if, let, begin 等）
- 第2层（4空格）：结构体内部
- 第3层（6空格）：嵌套结构内部
- 依此类推...

---

## 2. 命名规范

### 2.1 函数命名

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| 普通函数 | `kebab-case` | `parse-markdown`, `render-html` |
| 谓词函数（返回布尔值） | `kebab-case?` | `cmark-node?`, `blank-line?` |
| 副作用函数 | `kebab-case!` | `cmark-node-set-type!`, `append-child!` |
| 构造函数 | `make-kebab-case` | `make-cmark-node`, `make-cmark-parser` |
| 类型检查函数 | `type?` | `cmark-block-type?`, `cmark-inline-type?` |
| 常量 | `X_YY_ZZ`（大写，下划线分隔） | `MAX_DEPTH`, `DEFAULT_WIDTH`, `CMARK_OPT_UNSAFE` |

```scheme
;; ✅ 正确
(define (compute-value x)
  ...
) ;define

(define (positive? x)
  ...
) ;define

(define (set-value! x v)
  ...
) ;define

;; ❌ 错误
(define (computeValue x) ...)  ; 驼峰命名
(define (is-positive x) ...)   ; 不是谓词风格
```

### 2.2 变量命名

- 使用 `kebab-case`
- 避免单字母变量名（循环变量除外）
- 布尔变量使用 `?` 后缀：`hard-break?`, `is-valid?`

```scheme
;; ✅ 正确
(define max-value 100)
(define item-count 0)

;; ❌ 错误
(define maxValue 100)  ; 驼峰命名
(define n 0)           ; 单字母（除非循环）
```

### 2.3 模块命名

- 使用 `(liii 模块名)` 格式
- 模块名使用 `kebab-case`
- 示例：`(liii cmark)`, `(liii cmark-node)`, `(liii cmark-parser)`

---

## 3. 代码格式

### 3.1 缩进规则

#### 3.1.1 使用 2 个空格缩进（强制）

```scheme
;; ✅ 正确：2 空格缩进
(define (process-data x)
  (let ((validated (validate x)))
    (if (valid? validated)
      (transform validated)
      (default-value)
    ) ;if
  ) ;let
) ;define

;; ❌ 错误：4 空格缩进
(define (process-data x)
    (let ((validated (validate x)))
        (if (valid? validated)
            (transform validated)
            (default-value)
        ) ;if
    ) ;let
) ;define
```

#### 3.1.2 缩进层级表

```scheme
;; 层级 0: define-library (0 空格)
(define-library (liii cmark)
  ;; 层级 1: begin, import, export (缩进 2 空格)
  (import (scheme base))
  (export parse-markdown)
  
  (begin
    ;; 层级 2: define, define-record-type (缩进 4 空格)
    (define (parse-markdown str)
      ;; 层级 3: if, let, cond (缩进 6 空格)
      (if (string? str)
        ;; 层级 4: 函数体 (缩进 8 空格)
        (process-string str)
        (error "Expected string")
      ) ;if
    ) ;define
  ) ;begin
) ;define-library
```

### 3.2 括号规则

#### 3.2.1 开闭括号同列（强制）

```scheme
;; ✅ 正确：同列
(define (process-data x)
  (let ((validated (validate x)))
    (if (valid? validated)
      (transform validated)
      (default-value)
    ) ;if
  ) ;let
) ;define

;; ❌ 错误：不同列
(define (process-data x)
  (let ((validated (validate x)))
    (if (valid? validated)
      (transform validated)
      (default-value)
      ) ;if
    ) ;let
  ) ;define
```

#### 3.2.2 右括号注释（强制）

**原则：只要一个环境换行了，都必须用尾部注释显式标记**

**重要说明**：单个 `;` 的尾部注释（如 `) ;define`）**仅用于** Agentic Interface 规范中环境的右侧标记，不作为普通行内注释使用。

| 情况 | 需要注释？ |
|------|-----------|
| 单行（<20字符） | ❌ 不需要，且**必须**单行 |
| 单行（≥20字符） | ❌ 不需要，但可酌情换行 |
| 多行 | ✅ 强制（显式标记） |

> **为什么是 20？** 20 是行长度限制 100 的 1/5，是一个经验性的阈值。短于 20 字符的代码通常足够简单，强制换行反而降低可读性。

**示例说明：**

```scheme
;; 单行（<20字符）：不需要注释，且必须单行
(define max-value 100)
(define pi 3.14159)

;; 多行（≥20字符或复杂）：强制注释
(define (process-item item)
  (let ((validated (validate item)))
    (if (valid? validated)
      (transform validated)
      (default-value)
    ) ;if
  ) ;let
) ;define

;; 嵌套结构：每层都标记
(define (compute-result x y)
  (let ((sum (+ x y))
        (product (* x y)))
    (if (> sum 0)
      (positive-case sum product)
      (negative-case sum product)
    ) ;if
  ) ;let
) ;define
```

**常见环境标记示例：**

```scheme
;; define - 复杂函数体
(define (process-item item)
  (let ((validated (validate item)))
    (if (valid? validated)
      (transform validated)
      (default-value)
    ) ;if
  ) ;let
) ;define

;; let - 多行绑定
(let ((input (read-input))
      (config (load-config))
      (logger (get-logger)))
  (process-with-config input config logger)
) ;let

;; if
(if (> x 0)
  x
  0
) ;if

;; cond
(cond
  ((> x 0) (positive-case x))
  ((< x 0) (negative-case x))
  (else 0)
) ;cond

;; lambda - 复杂函数体跨行
(map
  (lambda (item)
    (let ((processed (process-item item)))
      (if (valid? processed)
        processed
        (default-value)
      ) ;if
    ) ;let
  ) ;lambda
  items
) ;map

;; do
(do ((i 0 (+ i 1)))
    ((>= i 10))
  (display i)
) ;do
```

**named let (let loop) 示例：**

```scheme
(define (sum-list lst)
  (let loop ((items lst) (acc 0))
    (if (null? items)
      acc
      (loop (cdr items) (+ acc (car items)))
    ) ;if
  ) ;let
) ;define
```

### 3.3 行长度

- **限制**：每行不超过 100 个字符
- **原因**：LLM tokenize 机制对长行处理效果较差
- **拆分策略**：使用多行 `let` 或提取子函数

### 3.4 嵌套深度

- **原则**：嵌套深度越少越好，不设硬性上限
- **判断标准**：当代码难以一眼看清结构时，就应该拆分
- **方法**：使用辅助函数或 `let` 提取中间结果

```scheme
;; ❌ 错误：深层嵌套难以阅读
(define (process data)
  (map (lambda (x) (if (positive? x) (* x 2) 0)) data)
) ;define

;; ✅ 正确：拆分为扁平结构
(define (transform x)
  (if (positive? x) (* x 2) 0)
) ;define

(define (process data)
  (map transform data)
) ;define
```

---

## 4. 代码组织

### 4.1 文件结构

```scheme
;;; 文件头注释
;;; 版权声明
;;; 模块描述

(define-library (liii example)
  ;;; ---------- 导入模块 ----------
  (import (scheme base))
  (import (liii string))

  ;;; ---------- 导出接口 ----------
  (export public-function-1)
  (export public-function-2)

  (begin
    ;;; ---------- 常量定义 ----------
    (define MAX_DEPTH 100)
    (define DEFAULT_WIDTH 80)

    ;;; ---------- 类型定义 ----------
    (define-record-type node
      (make-node type data)
      node?
      (type node-type node-set-type!)
      (data node-data node-set-data!))

    ;;; ---------- 私有函数 ----------
    (define (helper-func x)
      ...)

    ;;; ---------- 公共函数 ----------
    (define (public-function x)
      ...)
  ) ;begin
) ;define-library
```

### 4.2 函数长度

- **建议**：函数不超过 30-50 行
- **超过 50 行**：考虑拆分为多个小函数

### 4.3 文件长度

- **限制**：单个 Scheme 代码文件不超过 500 行
- **超过 500 行**：考虑拆分为多个模块或文件
- **原因**：保持文件可读性，便于代码审查和维护
- **超过 100 行**：必须拆分

### 4.3 参数数量

- **建议**：不超过 5 个参数
- **超过 5 个**：使用 `let` 或 `define-record-type` 封装参数

### 4.4 具体结构规范

#### 4.4.1 define（函数定义）

```scheme
;; 简单函数（单行）
(define (square x) (* x x))

;; 复杂函数（多行）
(define (factorial n)
  (if (= n 0)
    1
    (* n (factorial (- n 1)))
  ) ;if
) ;define
```

#### 4.4.2 define（变量定义）

```scheme
;; 简单值（单行）
(define pi 3.14159)

;; 复杂表达式（多行）
(define config
  (make-config
    (host "localhost")
    (port 8080)
    (timeout 30000)
    (retry 3)
    (debug #t)
  ) ;make-config
) ;define
```

#### 4.4.3 if 表达式

```scheme
;; 简单条件（单行）
(if (> x 0) x 0)

;; 复杂条件（多行）
(if (and (positive? x) (even? x))
  (handle-even-positive x)
  (handle-other x)
) ;if
```

#### 4.4.4 let 表达式

```scheme
;; 简单绑定（单行）
(let ((x 1) (y 2)) (+ x y))

;; 复杂绑定（多行）
(let ((x (compute-x))
      (y (compute-y))
      (z (compute-z)))
  (+ x y z)
) ;let

;; 双层括号：绑定值跨行
(let ((x (map (lambda (n) (* n n)) '(1 2 3 4 5)))
      (y (filter (lambda (n) (> n 0)) numbers)))
  (+ x y)
) ;let

;; 绑定值是复杂表达式
(let ((result
        (if condition
          (then-branch)
          (else-branch)
        ) ;if
      ) ;result
      (processed
        (cond
          ((> x 0) (positive-case))
          ((< x 0) (negative-case))
          (else (zero-case))
        ) ;cond
      ) ;processed
  )
  (combine result processed)
) ;let
```

#### 4.4.5 cond 表达式

```scheme
;; 简单条件
(cond
  ((> x 0) "positive")
  ((< x 0) "negative")
  (else "zero")
) ;cond

;; 复杂条件：条件或结果跨行
(cond
  ((and (> x 0) (even? x))
   (map (lambda (n) (* n 2)) (list x y z))
  )
  ((or (< x 0) (odd? x))
   (filter (lambda (n) (> n 0)) numbers)
  )
  (else default-value)
) ;cond

;; cond 中使用 let
(cond
  ((> x 0)
   (let ((doubled (* x 2)) (tripled (* x 3)))
     (list doubled tripled)
   ) ;let
  )
  ((< x 0)
   (process-negative (abs x))
  )
  (else 'zero)
) ;cond
```

#### 4.4.6 define-library

```scheme
(define-library (liii example)
  ;; 基础导入
  (import (scheme base))
  ;; 选择性导入：只导入需要的函数
  (import (only (liii list) map filter))
  ;; export 一个函数一行，方便添加注释
  (export foo)
  (export bar)
  (begin
    (define (foo x)
      (+ x 1)
    ) ;define

    (define (bar x)
      (* x 2)
    ) ;define
  ) ;begin
) ;define-library
```

---

## 5. 注释规范

### 5.1 注释类型

| 类型 | 格式 | 用途 |
|------|------|------|
| 文件头注释 | `;;;` | 版权声明、模块描述 |
| 章节注释 | `;;; ---------- 章节名 ----------` | 代码分节 |
| 函数文档 | `;; 函数名: 描述` | 函数说明 |
| 任务标记 | `;; TODO: 描述` | 待办事项 |
| 任务标记 | `;; FIXME: 描述` | 需要修复的问题 |

**注意**：单个 `;` 的尾部注释仅用于 Agentic Interface 规范中环境的右侧标记（如 `) ;define`），不作为普通行内注释使用。详见 3.2.2。

### 5.2 块注释

使用 `;;` 进行段落说明：

```scheme
;; 计算阶乘
;; n: 非负整数
;; 返回: n 的阶乘
(define (factorial n)
  ...
) ;define
```

### 5.3 代码文件的头部注释

使用 `;;;` 进行代码文件的头部注释，**一般用于声明版权信息**：

```scheme
;;; Markdown 解析器
;;;
;;; Copyright (c) 2024 Liii Network
;;; All Rights Reserved
;;;
;;; 作者: Xiao Liyu
;;; 日期: 2024-03-07
```

**禁止使用的注释格式**：
- 禁止使用 `#| ... |#` 块注释格式，因为会增加 goldfix 的实现难度

### 5.4 函数文档格式

**重要原则**：
- **接口文档**（函数签名、参数、返回值、示例）→ **写在测试文件中**
- **实现细节**（算法、复杂度、注意事项）→ **写在实现文件中**

**实现文件中的注释**（关注实现细节）：

```scheme
;; 实现: 使用尾递归优化
;; 注意: 输入必须为非负整数
(define (factorial n)
  (let loop ((n n) (acc 1))
    (if (= n 0)
      acc
      (loop (- n 1) (* acc n))
    ) ;if
  ) ;let
) ;define
```

**测试文件中的注释**（关注接口文档，采用当前项目的函数说明块风格）：

**格式规范**：
- 章节分隔符统一使用四个 `-`（如 `语法 ----`）
- 注释文档应直接放在 `import` 和 `(check-set-mode! 'report-failed)` 之后

```scheme
(check-set-mode! 'report-failed)

;; square
;; 计算平方
;;
;; 语法
;; ----
;; (square x)
;;
;; 参数
;; ----
;; x : number?
;; 任意数值。
;;
;; 返回值
;; ----
;; number?
;; x 的平方。
;;
;; 说明
;; ----
;; 单元测试本身就是最佳示例

(check (square 3) => 9)
```

```scheme
;; xcons
;; 交换参数顺序的 cons 操作。
;;
;; 语法
;; ----
;; (xcons obj1 obj2)
;;
;; 参数
;; ----
;; obj1 : any
;;   任意对象。
;; obj2 : any
;;   任意对象。
;;
;; 返回值
;; ----
;; pair
;;   返回 (obj2 . obj1) 组成的对。
;;
;; 注意
;; ----
;; xcons 是 SRFI-1 中的一个实用工具函数，便于从右向左构建列表。
;; 当与函数组合使用时特别有用。
;;
;; 错误处理
;; ----
;; wrong-number-of-args 如果参数数量不为 2。
;;
;; 说明
;; ----
;; 单元测试本身就是最佳示例，无需在文档中重复示例代码。
;; 请参考 tests/liii/list-test.scm 中的测试用例。
```

### 5.6 复杂算法注释

```scheme
;; 算法: 二分查找
;; 复杂度: O(log n)
;; 步骤:
;;   1. 初始化左右边界
;;   2. 计算中间位置
;;   3. 比较并调整边界
;;   4. 重复直到找到或边界交叉
(define (binary-search vec target)
  ...
) ;define
```

---

## 6. 函数设计

### 6.1 纯函数优先

- 优先编写纯函数（无副作用）
- 副作用函数必须明确标注（使用 `!` 后缀）
- 分离纯逻辑和副作用操作

### 6.2 递归 vs 迭代

- **尾递归优先**：Scheme 优化尾递归
- **使用命名 let**：复杂循环使用命名 `let`

```scheme
;; 好的做法：尾递归
(define (sum-list lst)
  (let loop ((items lst) (acc 0))
    (if (null? items)
      acc
      (loop (cdr items) (+ acc (car items)))
    ) ;if
  ) ;let
) ;define

;; 避免：非尾递归
(define (sum-list-bad lst)
  (if (null? lst)
    0
    (+ (car lst) (sum-list-bad (cdr lst)))
  ) ;if
) ;define
```

### 6.3 错误处理

- 使用 `error` 报告致命错误（需要 `(import (liii error))`）
- 使用 `assert` 验证前置条件
- 返回 `#f` 或特定值表示失败（非致命）

```scheme
(import (liii error))

(define (parse-integer str)
  (if (string? str)
    (string->number str)
    (error "(liii parser) parse-integer: Expected string, got" str)
  ) ;if
) ;define
```

---

## 7. 测试规范

### 7.1 目录结构

新增某个库、模块或命名空间节点的单元测试时，统一使用“两层结构”：

```text
tests/<group>/<library>-test.scm
tests/<group>/<library>/<case>-test.scm
```

其中：

- `tests/<group>/<library>-test.scm` 是顶层入口说明文件
- `tests/<group>/<library>/` 目录下放实际的单元测试文件

例如：

```text
tests/scheme/eval-test.scm
tests/scheme/eval/environment-test.scm
tests/scheme/eval/eval-test.scm

tests/liii/hash-table-test.scm
tests/liii/hash-table/hash-table-ref-test.scm
tests/liii/hash-table/hash-table-update-bang-slash-default-test.scm
```

### 7.2 顶层入口文件写法

顶层入口文件不是具体断言测试文件，而是“模块说明 + 用法示例 + 函数分类索引”的入口文件。

它的职责是：

- 用几句话说明这个库是什么、适合做什么
- 给出 1 到 3 个最常见、最能代表该库风格的用法示例
- 告诉读者如何用 `gf doc` 继续查看具体函数文档
- 用分类索引列出该库的主要导出函数，方便快速导航

它不应该承担这些职责：

- 不写 `check`
- 不写 `check-report`
- 不把多个独立函数的测试逻辑堆在这个文件里
- 不在这里做具体函数的断言测试

推荐结构：

1. 文件标题：说明这是某个库的模块索引
2. 一两段简短介绍：说明该库的用途、特点、与相近库的区别
3. `==== 常见用法示例 ====`
4. `import` 被说明的库
5. 1 到 3 个短小、稳定、代表性强的示例
6. `==== 如何查看函数的文档和用例 ====`
7. 给出 `bin/gf doc ...` 示例命令
8. `==== 函数分类索引 ====`
9. 按类别罗列该模块的主要函数

推荐样式：

```scheme
;; (liii range) 模块函数分类索引
;;
;; range 是一种惰性序列，适合表示按需生成的数据。
;; 与立即构造完整列表的接口相比，它更适合大范围遍历和切片。

;; ==== 常见用法示例 ====
(import (liii range))

;; 示例1：遍历区间
(range-for-each
  (lambda (x) (display x) (newline))
  (numeric-range 0 5)
) ;range-for-each

;; 示例2：折叠求和
(range-fold + 0 (numeric-range 1 11)) ; => 55

;; ==== 如何查看函数的文档和用例 ====
;;   bin/gf doc liii/range "range?"
;;   bin/gf doc liii/range "numeric-range"

;; ==== 函数分类索引 ====
;;
;; 一、构造函数
;;   range          - 创建 range 对象
;;   numeric-range  - 创建数值范围
;;
;; 二、谓词函数
;;   range?         - 判断是否为 range
```

### 7.3 单个测试文件放哪、怎么写

实际测试文件放在对应库目录下，一般一份文件测试一个导出过程、构造器、谓词或语法入口。

默认规则是：

- 一个测试文件通常只测试一个对应目标
- 如果测试目标是构造器、谓词或语法入口，也按“一个文件对应一个目标”处理
- 不要把多个不直接相关的导出过程混写在同一个文件里

例如：

- `environment-test.scm` 只测试 `environment`
- `eval-test.scm` 只测试 `eval`
- `hash-table-ref-test.scm` 只测试 `hash-table-ref`

单个测试文件建议遵循以下结构：

1. 导入 `liii check` 与被测库
2. 需要时导入依赖库
3. 调用 `(check-set-mode! 'report-failed)`
4. 写函数说明块
5. 写实际 `check` / `check-true` / `check-false` / `check-catch`
6. 结尾调用 `(check-report)`

典型骨架：

```scheme
(import (liii check)
        (<namespace> <library>)
) ;import

(check-set-mode! 'report-failed)

;; function-name
;; 一句简短说明。
;;
;; 语法
;; ----
;; (function-name arg1 arg2 ...)
;;
;; 参数
;; ----
;; arg1 : type
;; 参数说明。
;;
;; 返回值
;; ----
;; return-type
;; 返回值说明。
;;
;; 描述
;; ----
;; 对行为做简短说明。
;;
;; 错误处理
;; ----
;; type-error
;; 当输入不合法时抛出。

(check ...)

(check-report)
```

补充约定：

- 注释块标题应直接写被测目标名
- 带符号名字的目标，标题仍写原始 Scheme 名字，例如 `char-ci=?`、`njson-set!`
- 注释文档应尽量简洁、可扫描，不写成长篇模块综述
- 如果某一项没有意义，可以省略
- 如果一个文件开始系统性测试第二个独立导出过程，就应该拆成新的 `xxx-test.scm`

### 7.4 文件命名规则

实际测试文件统一使用：

```text
<exported-name-normalized>-test.scm
```

其中 `normalized` 的转换规则如下。

#### 7.4.1 普通标识符

- 普通字母、数字、连字符 `-` 保持原样

例如：

- `environment` -> `environment-test.scm`
- `digit-value` -> `digit-value-test.scm`

#### 7.4.2 特殊符号转义

- `?` -> `-p`
- `!` -> `-bang`
- `/` -> `-slash-`
- `->` -> `-to-`
- `*` -> `-star`

例如：

- `char-upper-case?` -> `char-upper-case-p-test.scm`
- `njson-set!` -> `njson-set-bang-test.scm`
- `hash-table-update!/default` -> `hash-table-update-bang-slash-default-test.scm`
- `string->number` -> `string-to-number-test.scm`
- `trie-ref*` -> `trie-ref-star-test.scm`

#### 7.4.3 运算符名称转义

纯运算符或比较符，按可读英文缩写转义：

- `+` -> `plus`
- `-` -> `minus`
- `*` -> `star`
- `/` -> `slash`
- `=` -> `eq`
- `<` -> `lt`
- `<=` -> `le`
- `>` -> `gt`
- `>=` -> `ge`

例如：

- `+` -> `plus-test.scm`
- `vector=` -> `vector-eq-test.scm`
- `isubset>=` -> `isubset-ge-test.scm`

### 7.5 新增测试时的检查清单

新增测试前请确认：

1. 测试是否放在了正确的 `tests/<group>` 目录下
2. 是否已经存在对应的顶层 `<library>-test.scm` 入口文件
3. 单个测试文件是否放在 `<library>/` 子目录下
4. 单个测试文件是否只围绕文件名对应的一个目标展开
5. 文件名是否按照导出名字做了统一转义
6. 顶层入口文件是否保持为说明入口，而不是混入具体断言
7. 测试文件末尾是否调用了 `(check-report)`

---

## 8. 示例代码

### 8.1 完整示例

**实现文件** (`liii/math.scm`):

```scheme
;;; 数学工具模块
;;;
;;; Copyright (c) 2024 Liii Network
;;; All Rights Reserved
;;;
;;; 作者: Xiao Liyu
;;; 日期: 2024-03-07

(define-library (liii math)
  (import (scheme base))
  ;; 导出数学函数
  (export square)
  (export factorial)
  (export power)

  (begin
    ;; 实现: 平方函数
    (define (square x)
      (* x x)
    ) ;define

    ;; 实现: 阶乘函数，使用尾递归优化
    (define (factorial n)
      (let loop ((n n) (acc 1))
        (if (= n 0)
          acc
          (loop (- n 1) (* acc n))
        ) ;if
      ) ;let
    ) ;define

    ;; 实现: 幂函数
    (define (power x n)
      (if (= n 0)
        1
        (* x (power x (- n 1)))
      ) ;if
    ) ;define
  ) ;begin
) ;define-library
```

**顶层入口文件** (`tests/liii/math-test.scm`):

```scheme
;; (liii math) 模块函数分类索引
;;
;; math 提供一组基础数学函数，适合放置纯计算逻辑。
;; 如果函数具有清晰的数值语义，建议优先放到这里统一管理。

(import (liii math))

;; ==== 常见用法示例 ====
;; (square 12) ; => 144
;; (factorial 5) ; => 120
;; (power 2 10) ; => 1024

;; ==== 如何查看函数的文档和用例 ====
;;   bin/gf doc liii/math "square"
;;   bin/gf doc liii/math "factorial"

;; ==== 函数分类索引 ====
;;   square     - 计算平方
;;   factorial  - 计算阶乘
;;   power      - 计算幂
```

**函数级测试文件** (`tests/liii/math/square-test.scm`):

```scheme
(import (liii check)
        (liii math)
) ;import

(check-set-mode! 'report-failed)

;; square
;; 计算平方。
;;
;; 语法
;; ----
;; (square x)
;;
;; 参数
;; ----
;; x : number?
;; 任意数值。
;;
;; 返回值
;; ----
;; number?
;; 返回 x 的平方。

(check (square 3) => 9)
(check (square -4) => 16)

(check-report)
```

---

## 9. 参考

- [Databricks Scala Style Guide](https://github.com/databricks/scala-style-guide)
- [Scheme R7RS Specification](https://small.r7rs.org/)
- [Google Common Lisp Style Guide](https://google.github.io/styleguide/lispguide.xml)

---

## 10. 术语表

| 术语 | 英文 | 定义 |
|------|------|------|
| 环境 | Environment | 左括号和与之匹配的右括号所包含的 S 表达式块 |
| 标记 | tag | 环境的标识符，如 `define`, `let`, `if`, `lambda` 等 |
| 尾部注释 | trailing comment | 位于右括号后的注释，用于标记环境，如 `) ;define` |
| 扁平化结构 | flat structure | 避免深层嵌套，通过拆分函数或使用 `let*` 使代码结构扁平 |
| 谓词函数 | predicate function | 返回布尔值的函数，以 `?` 结尾，如 `positive?` |
| 副作用函数 | side-effect function | 有副作用的函数，以 `!` 结尾，如 `set-value!` |

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0 | 2026-03-07 | 初始版本，参考 Scala Style Guide |
| 1.1 | 2026-03-10 | 整合 scheme-style-guide.md，添加明确编号系统(1-10章节)，标注 goldfix 为规划中，基于 Agentic Interface 思想 |
| 1.2 | 2026-03-24 | 更新右侧标记格式：`) ; define` → `) ;define`（分号后无空格），避免 LLM tokenize 问题 |
| 1.3 | 2026-03-24 | 移除第9节验证工具（goldfix），在文档顶部添加 Goldfish Scheme 开发环境说明 |

---

**三鲤网络（Liii Network）**

*让 S-expression 在 AI 时代焕发新生*

