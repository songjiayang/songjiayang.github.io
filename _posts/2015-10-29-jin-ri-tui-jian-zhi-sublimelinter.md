---
title: "今日推荐之 SublimeLinter"
date: 2015-10-29 23:31:00 +0800
tags: [效率]
---

![linter.png](/images/linter.png)

### SublimeLinter 是什么？

SublimeLinter 是一个 Sublime Text 3 上的代码诊断框架，用于代码错误和不良的代码风格高亮提示。
值得注意的是，它只是个框架，接口，不包含具体某种语言的诊断，针对特定语言代码的诊断需要依赖以它为基础的插件，
比如 ES6 的`SublimeLinter-contrib-eslint-d`, Ruby 的 `SublimeLinter-contrib-ruby-lint`， 等等。

### SublimeLinter 安装

1. 使用命令`cmd+shift+p` 打开命令面板（ Linux/Windows 使用 `ctrl+shift+p`）。
2. 选择 `Package Control: Install Package` 项 。
3. 在可用 Sublime 插件包列表中输入 `linter`， 选择 `SublimeLinter` 。
4. 安装完毕，重启 Sublime 。

### SublimeLinter 配置

1. `Tools -> SublimeLinter -> Lint Mode -> Save only`
2. `Tools -> SublimeLinter -> Mark Style -> Outline`

### 具体 linter 插件安装

按照如上 `SublimeLinter 安装` 方式，在 Sublime 插件包列表中输入 `SublimeLinter-`, 就能找到你想要的语言插件了。

至此，你已可以在 Sublime 中愉快使用 linter 了。

这么好的东西，你还等什么，赶快去安装吧！

<hr />

PS: 以下是我们项目中 ES6 和 Ruby 规则的配置文件，（配置文件放在项目根目录下）：

.eslintrc
```
{
  "extends": "airbnb",
  "rules": {
    "valid-jsdoc": 2,

    // Disable until Flow supports let and const
    "no-var": 0,
    "vars-on-top": 0,

    // Disable Airbnb-specific rules unless need to support it
    "comma-dangle": 0,
    "id-length": 0,

    // Disable in prefer to single quote for consistence
    "jsx-quotes": 0,

    // Disable temporarily
    "react/prop-types": 0,

    // Disable cause of the name convention of Redux actions and action creators
    "no-shadow": 0,
    "no-unused-expressions": [2, { allowTernary: true }]
  }
}

```

.rubocop.yml
```
AllCops:
  RunRailsCops: true
  DisplayCopNames: true
  DisplayStyleGuide: true

Style/ClassAndModuleChildren:
  Description: 'Checks style of children classes and modules.'
  Enabled: false

Metrics/ClassLength:
  Description: 'Avoid classes longer than 100 lines of code.'
  Enabled: false

Metrics/ModuleLength:
  Description: 'Avoid modules longer than 100 lines of code.'
  Enabled: false

Style/Documentation:
  Description: 'Document classes and non-namespace modules.'
  Enabled: false

Style/HashSyntax:
  Description: >-
                 Prefer Ruby 1.9 hash syntax { a: 1, b: 2 } over 1.8 syntax
                 { :a => 1, :b => 2 }.
  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#hash-literals'
  Enabled: true

Metrics/MethodLength:
  CountComments: false  # count full line comments?
  Max: 20

Metrics/LineLength:
  Max: 120
  # To make it possible to copy or click on URIs in the code, we allow lines
  # contaning a URI to be longer than Max.
  AllowURI: true
  URISchemes:
    - http
    - https

Style/StringLiterals:
  Enabled: false

Style/SpaceInsideHashLiteralBraces:
  Description: "Use spaces inside hash literal braces - or don't."
  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#spaces-operators'
  Enabled: false

Style/BracesAroundHashParameters:
  Description: 'Enforce braces style around hash parameters.'
  Enabled: false

Lint/AssignmentInCondition:
  Enabled: false

Style/AlignParameters:
  EnforcedStyle: with_fixed_indentation
  SupportedStyles:
    - with_first_parameter
    - with_fixed_indentation

Style/SingleSpaceBeforeFirstArg:
  Description: >-
                 Checks that exactly one space is used between a method name
                 and the first argument for method calls without parentheses.
  Enabled: false

Style/RescueModifier:
  Description: 'Avoid using rescue in its modifier form.'
  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-rescue-modifiers'
  Enabled: false

```
