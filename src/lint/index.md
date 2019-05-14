## 代码规范
### 1. js规范
#### eslint
- [文档](https://cn.eslint.org/)
- airbnb
    - [文档]（https://juejin.im/entry/56e8c0c1816dfa0051376758）

#### prettier

#### standard


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
        },
    }
```


