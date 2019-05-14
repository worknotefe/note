## 代码规范
### 1. js规范
#### eslint
Linter 是检查代码风格/错误的小工具。其他类似的 Linter 工具还有：TSLint、stylelin
- [文档](https://cn.eslint.org/)
- airbnb
    - [文档]（https://juejin.im/entry/56e8c0c1816dfa0051376758）
- 功能
    - check syntax
    - find problems：前两个可以统称为 Code-quality rules，例如 no-unused-vars 规则
    - enforce code style：最后一个可以称为 Formatting rules ，例如 keyword-spacing 规则

#### prettier
- Prettier 只是一个Formatting rules ，负责 enforce code style
- 为什么要把 ESLint 的 Formatting rules 部分用 Prettier 取代
    - 代码规范比 ESLint 的 Airbnb、Standard 更好更先进
    - 比 ESLint 提供更少的代码风格规则配置项，终极目的是结束关于代码风格的争论
    - 比 ESLint 支持更多的语言

#### standard
- enforce code style：无需配置，无法修改
- [文档](https://standardjs.com/readme-zhcn.html)

### 2. style规范

### 3. 项目实现
#### 绑定git hook
一般强制要求代码提交前，变动代码必须做格式校验，借助成熟的npm包实现

如果想要忽略，增加`--no-verify`

- husky
```
    // package.json
    "husky": {
        "hooks": {
          "pre-commit": "lint-staged"
        }
    },
    "lint-staged": {
        "**/*.js": [
          "sh ./eslint.sh",
          "git add"
        ]
    }
```
- pre-commit
```
    // package.json
   "pre-commit": [
       "lint-staged"
   ],
   "lint-staged": {
       "**/*.{js,jsx,tsx,ts,less,md,json}": [
         "prettier --write",
         "git add"
       ]
   }
```

#### 编译时检测
```
    // webpack.config.js module.rules
    {
        enforce: 'pre',
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'eslint-loader',
        options: {
            ignorePath: path.join(__dirname, '../.eslintignore-tmp'),
            fix: true,
        },
    }
```

### 4. 其他
#### [.editorconfig](http://www.alloyteam.com/2014/12/editor-config/)
```
root = true

[*]
indent_style = tab
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.{json,yml,md}]
indent_style = space
indent_size = 2
```



