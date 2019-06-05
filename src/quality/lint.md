## 代码规范
### 1. js规范
#### eslint
Linter 是检查代码风格/错误的小工具。其他类似的 Linter 工具还有：TSLint、stylelint
- [文档](https://cn.eslint.org/)
- airbnb
    - [文档]（https://juejin.im/entry/56e8c0c1816dfa0051376758）
- 功能
    - check syntax
    - find problems：前两个可以统称为 Code-quality rules，例如 no-unused-vars 规则
    - enforce code style：最后一个可以称为 Formatting rules ，例如 keyword-spacing 规则

#### [prettier](https://segmentfault.com/a/1190000012909159)
- Prettier 只是一个Formatting rules ，负责 enforce code style
- 为什么要把 ESLint 的 Formatting rules 部分用 Prettier 取代
    - 代码规范比 ESLint 的 Airbnb、Standard 更好更先进
    - 比 ESLint 提供更少的代码风格规则配置项，终极目的是结束关于代码风格的争论
    - 比 ESLint 支持更多的语言

Prettier is an opinionated code formatter with support for:
- JavaScript, including ES2017
- JSX
- Angular
- Vue
- Flow
- TypeScript
- CSS, Less, and SCSS
- HTML
- JSON
- GraphQL
- Markdown, including GFM and MDX
- YAML

使用，结合各类lint工具
- eslint
    - eslint-plugin-prettier
        ```
            {
              "plugins": ["prettier"],
              "rules": {
                "prettier/prettier": "error"
              }
            }
        ```
    - eslint-config-prettier: 关闭冲突的rule
        ```
            {
              "extends": ["prettier"]
            }
        ```
    - 以上两个结合,可简单使用以下，或者直接组合
        ```
            {
                "extends": ["plugin:prettier/recommended"]
            }
        ```
        ```
            {
                "extends": ["prettier"],
                "plugins": ["prettier"],
                "rules": {
                    "prettier/prettier": "error"
                }
            }
        ```
- stylelint
    - stylelint-config-prettier
        ```
            {
              "extends": ["stylelint-config-prettier"]
            }
        ```
    - stylelint-prettier
        ```
            {
              "plugins": ["stylelint-prettier"],
              "rules": {
                "prettier/prettier": true
              }
            }
        ```

#### standard
- enforce code style：无需配置，无法修改
- [文档](https://standardjs.com/readme-zhcn.html)
- 可结合snazzy使用，美化错误提示
```
    // package.json
    {
     "husky": {
       "hooks": {
         "pre-commit": "standard \"src/**/*.{js,vue,wpy}\" | snazzy",
       }
     }
    }
```
```
    // standard 规范默认错误提示：
    D:\pre-commit\src\test.js:2:19: Extra semicolon.
    ------------------------------------------------
    // 利用 snazzy 美化后的错误提示：
    2:19  error  Extra semicolon
    ✖ 1 problem
```

### 2. style规范
[stylelint](https://stylelint.io/user-guide/) 是一个基于 Javascript 的代码审查工具，它易于扩展，支持最新的 CSS 语法，也理解类似 CSS 的语法，
[参考](https://segmentfault.com/a/1190000008708473)

顺序、属性是否有效等

- [配置说明](https://juejin.im/post/5b4ffd1ef265da0f990d52e8)
- extends
    - stylelint-config-standard
    - [stylelint-config-rational-order](https://www.npmjs.com/package/stylelint-config-rational-order)
    - [stylelint-config-prettier](https://www.npmjs.com/package/stylelint-config-prettier)
- plugins
    - stylelint-order
    - [stylelint-declaration-block-no-ignored-properties](https://www.npmjs.com/package/stylelint-declaration-block-no-ignored-properties)
    - stylelint-prettier

### 3. ts规范
#### [tslint](https://palantir.github.io/tslint/usage/cli/)
- tslint-plugin-prettier
```
    "rulesDirectory": ["tslint-plugin-prettier"],
    "rules": {
        "prettier": true,
    }
```
- tslint-config-prettier
- tslint-eslint-rules
- tslint:recommended
```
    "extends": ["tslint:recommended", "tslint-eslint-rules", "tslint-config-prettier"]
```

### 4. 项目实现
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
    "scripts": {
       "lint-staged": "lint-staged"
    },
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
- indent_style：tab为hard-tabs，space为soft-tabs。
- indent_size：设置整数表示规定每级缩进的列数和soft-tabs的宽度（译注：空格数）。如果设定为tab，则会使用tab_width的值（如果已指定）。
- tab_width：设置整数用于指定替代tab的列数。默认值就是indent_size的值，一般无需指定。
- end_of_line：定义换行符，支持lf、cr和crlf。
- charset：编码格式，支持latin1、utf-8、utf-8-bom、utf-16be和utf-16le，不建议使用uft-8-bom。
- trim_trailing_whitespace：设为true表示会除去换行行首的任意空白字符，false反之。
- insert_final_newline：设为true表明使文件以一个空白行结尾，false反之。
- root：表明是最顶层的配置文件，发现设为true时，才会停止查找.editorconfig文件



