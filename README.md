# 我的 Monkey 解释器学习笔记

这是我用来跟读《Writing an Interpreter in Go》（学习实现解释器）的个人练习仓库。下面记录当前实现情况、实现要点、常见陷阱和接下来的学习计划，作为自己的随手笔记。

## 当前实现（状态快照）

- 实现了 `lexer`：识别标识符、整数字面量、运算符（+ - * / ! = == != < >）、分隔符等。
- 实现了 `token` 包及关键字映射（`let`, `fn`, `true`, `false`, `if`, `else`, `return`）。
- 实现了部分 `parser`：前缀/中缀注册框架、标识符与整数字面量解析、`let` 和 `return` 语句的解析框架（还未完成表达式的完整解析与中缀解析）。
- 定义了 `ast` 节点（`Program`、`LetStatement`、`ReturnStatement`、`Identifier`、`IntegerLiteral` 等）。
- 一个最小 REPL（`repl.Start`）当前只把 lexer 输出的 tokens 打印到控制台，尚未把解析/求值接入 REPL。

## 如何运行（我自己常用的命令）

运行 REPL：

```bash
go run main.go
```

运行所有测试：

```bash
go test ./...
```

运行 parser 包的测试（调试解析器时常用）：

```bash
go test ./parser
```

如果需要可编译二进制：

```bash
go build -o monkey main.go
```

## 代码要点（给未来复习用）

- `lexer/lexer.go`：主循环通过读取字符构造 token。注意 `readChar()` 与 `peekChar()` 的配合，处理 `==`、`!=` 这类双字符 token 时先 peek 再决定。
- `token/token.go`：用 `LookupIdent` 将关键字映射为特定 token，减少解析器判断负担。
- `ast/ast.go`：节点实现了 `TokenLiteral()` 与 `String()`，方便调试打印 AST。
- `parser/parser.go`：使用 prefix/infix 解析函数注册表的设计，当前只注册了前缀解析函数（标识符、整数），中缀解析还没完成；`parseExpression` 会检查当前 token 是否有对应的前缀解析函数，否则记录错误。

常见问题记录：

- 当看到 `no prefix parse function for X found` 时，说明忘记为该 token 注册前缀解析函数或 token 类型识别有误。
- 在处理多字符运算符（`==`、`!=`）时，注意先 `peekChar()` 再调用 `readChar()` 更新 `l.ch`。

## 我的下一步计划（短期）

1. 把 `parser` 的中缀解析补上（实现运算符优先级解析）。
2. 实现 `evaluator`：从 AST 求值并返回对象（先实现整数/布尔/返回/环境）。
3. 把 `REPL` 连接到解析器 + 求值器，能输入表达式并看到结果。

## 文件说明（按文件）

下面是我给自己写的每个主要文件的简短说明和常看函数：方便之后快速定位与复习。

- `main.go`：程序入口，获取当前用户并启动 `repl.Start`。
  - 关注点：启动参数、`repl.Start(os.Stdin, os.Stdout)`。

- `repl/repl.go`：交互式循环（REPL）。当前只调用 `lexer` 打印 tokens；后续会替换为解析 + 求值流程。
  - 常看函数：`Start(in io.Reader, out io.Writer)`、`PROMPT`。

- `token/token.go`：定义 `TokenType`、`Token` 结构和所有常量 token。实现关键字查找：`LookupIdent`。
  - 关注点：关键字表 `keywords`、常量命名（保持与 lexer/parser 一致）。

- `lexer/lexer.go`：词法分析器核心。
  - 常看方法：`New(input string) *Lexer`、`(*Lexer) NextToken() token.Token`、`readChar()`、`peekChar()`、`readIdentifier()`、`readNumber()`、`skipWhitespace()`。
  - 注意点：`NextToken` 在遇到 `=` 或 `!` 时需要 `peekChar()` 以识别 `==` / `!=`。

- `ast/ast.go`：AST 节点定义与字符串化方法，便于打印与测试。
  - 关注类型：`Program`、`Statement`、`Expression`、`LetStatement`、`ReturnStatement`、`Identifier`、`IntegerLiteral`。
  - 常用方法：`TokenLiteral()`、`String()`。

- `parser/parser.go`：解析器实现（半成品）。
  - 关注结构体：`Parser`（包含 `l *lexer.Lexer`, `errors []string`, `curToken`, `peekToken`, `prefixParseFns`, `infixParseFns`）。
  - 常看函数：`New(l *lexer.Lexer) *Parser`、`nextToken()`、`ParserProgram()`、`parseStatement()`、`parseExpression(precedence int)`、`parseLetStatement()`、`parseReturnStatement()`、`parseIntegerLiteral()`、`registerPrefix`、`registerInfix`、`Errors()`。
  - TODO：实现中缀解析函数注册与表达式优先级处理。

- 测试文件：`lexer/lexer_test.go`, `parser/parser_test.go`, `ast/ast_test.go`。
  - 建议：运行对应测试以定位解析器/词法器行为变化。
