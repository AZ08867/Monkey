# Monkey Interpreter in Go

跟着《Writing an Interpreter in Go》（Thorsten Ball）写的学习项目，用 Go 从零实现 Monkey 语言解释器。

## 结构

```sh
token/    Token 类型定义、关键字映射
lexer/    词法分析器
ast/      AST 节点
parser/   语法解析器（Pratt parsing）
repl/     REPL
main.go
```

## 运行

```bash
go run main.go   # 启动 REPL
go test ./...    # 跑全部测试
```

## 进度

- [x] Chapter 1 — Lexing
- [x] Chapter 2 — Parsing（进行中，基础表达式已完成）
- [ ] Chapter 3 — Evaluation
- [ ] Chapter 4 — Extending the Interpreter

## 笔记

### Lexer

- `peekChar()` 用来处理双字符 token（`==`、`!=`），先 peek 再 `readChar`
- `LookupIdent` 区分关键字和普通 identifier

### Pratt Parser

- 每种 token 注册 `prefixParseFn` 或 `infixParseFn`
- `precedences` 表控制运算优先级，`peekPrecedence()` 驱动循环
- 报错 `no prefix parse function for X` → 基本是漏注册了

### AST

- 节点都实现 `Node` 接口：`TokenLiteral()` + `String()`
- `String()` 主要用来 debug，测试里直接打印 AST 对比结构
