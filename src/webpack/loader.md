## loader
### loader配置
loader用于对模块的源代码进行转换，在import或者加载模块时预处理文件
- 支持链式传递，一组链式的 loader 将按照相反的顺序执行（！）
- 可以是同步的，也可以是异步的
- 接收查询参数（？）

-----

### webpack module
这些选项决定了如何处理项目中的不同类型的模块

#### noParse
[RegExp] | function : 防止 webpack 解析那些任何与给定正则表达式相匹配的文件
```
    noParse: /jquery|lodash/

    // 从 webpack 3.0.0 开始
    noParse: function(content) {
      return /jquery|lodash/.test(content);
    }
```

#### rules 【array】
- 条件：test, include, exclude 和 resource，当使用多个条件时，所有条件都匹配
- 结果：
    - loader : 是 `Rule.use: [ { loader } ] `的简写
    - loaders : Rule.loaders 是 Rule.use 的别名
    - options : Rule.options 和 Rule.query 是 Rule.use: [ { options } ] 的简写
    - query : Rule.options 和 Rule.query 是 Rule.use: [ { options } ] 的简写
    - use :
    ```
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1
            }
          },
          {
            loader: 'less-loader',
            options: {
              noIeCompat: true
            }
          }
        ]
    ```
    - enforce:pre | post，没有指定，普通loader，所有 loader 通过 前置, 行内, 普通, 后置 排序，并按此顺序使用
    - parser : 解析选项对象，false，将禁用解析器
    ```
        //默认
        parser: {
          amd: false, // 禁用 AMD
          commonjs: false, // 禁用 CommonJS
          system: false, // 禁用 SystemJS
          harmony: false, // 禁用 ES2015 Harmony import/export
          requireInclude: false, // 禁用 require.include
          requireEnsure: false, // 禁用 require.ensure
          requireContext: false, // 禁用 require.context
          browserify: false, // 禁用特殊处理的 browserify bundle
          requireJs: false, // 禁用 requirejs.*
          node: false, // 禁用 __dirname, __filename, module, require.extensions, require.main 等。
          node: {...} // 在模块级别(module level)上重新配置 node 层(layer)
        }
    ```

- 嵌套的rule

------

### 如何编写loader
loader是一个导出为函数的 javaScript 模块，只做单一任务

#### 编写原则
- 简单易用
- 链式传递
- 模块化的输出
- 确保无状态
- 使用 loader utilities
```
    import { getOptions } from 'loader-utils';
    import validateOptions from 'schema-utils';

    const schema = {
      type: 'object',
      properties: {
        test: {
          type: 'string'
        }
      }
    }

    export default function(source) {
      const options = getOptions(this);

      validateOptions(schema, options, 'Example Loader');

      // 对资源应用一些转换……

      return `export default ${ JSON.stringify(source) }`;
    };
```
- 记录 loader 的依赖:如果一个 loader 使用外部资源（例如，从文件系统读取），必须声明它。这些信息用于使缓存 loaders 无效，以及在观察模式(watch mode)下重编译
```
    import path from 'path';

    export default function(source) {
      var callback = this.async();
      var headerPath = path.resolve('header.js');

      this.addDependency(headerPath);   // addDependency：声明

      fs.readFile(headerPath, 'utf-8', function(err, header) {
        if(err) return callback(err);
        callback(null, header + "\n" + source);
      });
    };
```
- 解析模块依赖关系
    - 通过把它们转化成 require 语句
    - 使用 this.resolve 函数解析路径
- 提取通用代码
- 避免绝对路径
- 使用 peer dependencies

#### API
- loader函数被loader runner调用
- 参数：上一个loader的产物或者源文件，可选的 SourceMap 结果，meta 参数
- this：webpack填充，可以获取很多方法
- 处理结果：
    - String 或者 Buffer
    - SourceMap：JSON 对象

##### 具体API

1. 同步loader:无论是 return 还是 this.callback 都可以同步地返回转换后的 content 内容
```
    module.exports = function(content, map, meta) {
      return someSyncOperation(content);
    };

    module.exports = function(content, map, meta) {
      this.callback(null, someSyncOperation(content), map, meta);
      return; // 当调用 callback() 时总是返回 undefined
    };
```

2. 异步 loader:使用 this.async 来获取 callback 函数，推荐使用
```
    module.exports = function(content, map, meta) {
      var callback = this.async();
      someAsyncOperation(content, function(err, result) {
        if (err) return callback(err);
        callback(null, result, map, meta);
      });
    };
```

3. "Raw" loader：默认情况下，资源文件会被转化为 UTF-8 字符串，然后传给 loader。通过设置 raw，loader 可以接收原始的 Buffer。
每一个 loader 都可以用 String 或者 Buffer 的形式传递它的处理结果。Complier 将会把它们在 loader 之间相互转换
```
    module.exports = function(content) {
        assert(content instanceof Buffer);
        return someSyncOperation(content);
        // 返回值也可以是一个 `Buffer`
        // 即使不是 raw loader 也没问题
    };
    module.exports.raw = true;
```

4. 越过 loader**<font color="#f55d59">(Pitching loader)</font>**:
loader 总是从右到左地被调用。有些情况下，loader 只关心 request 后面的元数据(metadata)，并且忽略前一个 loader 的结果。
在实际（从右到左）执行 loader 之前，会先从左到右调用 loader 上的 pitch 方法。对于以下 use 配置：
```
    use: [
      'a-loader',
      'b-loader',
      'c-loader'
    ]
```
将会发生这些步骤：
```
    |- a-loader `pitch`
      |- b-loader `pitch`
        |- c-loader `pitch`
          |- requested module is picked up as a dependency
        |- c-loader normal execution
      |- b-loader normal execution
    |- a-loader normal execution
```
那么，为什么 loader 可以利用 "跳跃(pitching)" 阶段呢？
首先，传递给 pitch 方法的 data，在执行阶段也会暴露在 this.data 之下，并且可以用于在循环时，捕获和共享前面的信息。
```
    module.exports = function(content) {
        return someSyncOperation(content, this.data.value);
    };

    module.exports.pitch = function(remainingRequest, precedingRequest, data) {
        data.value = 42;
    };
```
其次，如果某个 loader 在 pitch 方法中给出一个结果，那么这个过程会回过身来，并跳过剩下的 loader。在我们上面的例子中，如果 b-loader 的 pitch 方法返回了一些东西：
```
    module.exports = function(content) {
      return someSyncOperation(content);
    };

    module.exports.pitch = function(remainingRequest, precedingRequest, data) {
      if (someCondition()) {
        return "module.exports = require(" + JSON.stringify("-!" + remainingRequest) + ");";
      }
    };
```
上面的步骤将被缩短为：
```
    |- a-loader `pitch`
      |- b-loader `pitch` returns a module
    |- a-loader normal execution
```
5. loader上下文：this
- this.version：版本号，处理兼容很有用
- this.context：模块所在目录，可以用作解析其他模块路径的上下文
- this.request：被解析出来的 request 字符串
- this.query：loader配置options或者query字符串
- this.data：在 pitch 阶段和正常阶段之间共享的 data 对象
- this.callback：一个可以同步或者异步调用的可以返回多个结果的函数，如果被调用，务必返回undefined
```
    this.callback(
      err: Error | null,
      content: string | Buffer,
      sourceMap?: SourceMap,    // 可选的：第三个参数必须是一个可以被这个模块解析的 source map
      meta?: any    // 可选的：第四个选项，会被 webpack 忽略，可以是任何东西（例如一些元数据）
    );
```
- this.async：异步回调，返回this.callback
- this.cacheable：设置是否可缓存标志的函数,`cacheable(flag = true: boolean)`,
  默认情况下，loader 的处理结果会被标记为可缓存。调用这个方法然后传入 false，可以关闭 loader 的缓存
- this.loaders：所有 loader 组成的数组,`loaders = [{request: string, path: string, query: string, module: function}]`
- this.loaderIndex：当前 loader 在 loader 数组中的索引
- this.resource：request 中的资源部分，包括 query 参数
- this.resourcePath：资源文件的路径
- this.resourceQuery：资源的 query 参数
- this.target：编译的目标。从配置选项中传递过来的
- this.webpack：是否由webpack编译
- this.sourceMap：应该生成一个 source map
- this.emitWarning：发出一个警告
- this.emitError：发出一个错误
- this.loadModule
- this.resolve
- this.addDependency：加入一个文件作为产生 loader 结果的依赖，使它们的任何变化可以被监听到
```
    addDependency(file: string)
    dependency(file: string) // 简写
```
- this.addContextDependency：把文件夹作为 loader 结果的依赖加入
- this.clearDependencies：移除 loader 结果的所有依赖
- this.emitFile：产生一个文件。这是 webpack 特有的。`emitFile(name: string, content: Buffer|string, sourceMap: {...})`
- this.fs：用于访问 compilation 的 inputFileSystem 属性。
































