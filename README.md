## Wepack 性能优化

本篇文章我们一起来看看 `Webpack` 的性能优化相关内容。在这之前，再简单介绍下 `Webpack` 的一些相关概念。



### 一、webpack 是什么？

`webpack` 是一种前端资源构建工具，它是一个**静态模块打包器**（`module bundler`）。在 `webpack` 中，前端的所有资源文件（`javascript/json/css/img/less/...`）都会作为模块处理，当 `webpack` 处理应用程序时,它会递归地构建一个依赖关系图（`dependency graph`），其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 `bundle`。

<img src="https://github.com/IDeepspace/ImageHosting/raw/master/FrontEnd/webpack.png" alt="image-20200511205102019" style="zoom:50%;" />

<p align='center'>（图片来自网络）</p>

#### 1、基本功能

- **代码转换**：将 `TypeScript` 编译成 `JavaScript`、`SCSS` 编译成 `CSS` 等。
- **文件优化**：压缩 `JavaScript`、`CSS`、`HTML` 代码，压缩合并图片等。
- **代码分割**：提取多个页面的公共代码、提取首屏不需要执行部分的代码让其异步加载。
- **模块合并**：在采用模块化的项目里会有很多个模块和文件，需要构建功能把模块分类合并成一个文件。
- **自动刷新**：监听本地源代码的变化，自动重新构建、刷新浏览器。
- **代码校验**：在代码被提交到仓库前需要校验代码是否符合规范，以及单元测试是否通过。
- **自动发布**：更新完代码后，自动构建出线上发布代码并传输给发布系统。



#### 2、打包原理

- 识别入口文件，分析代码，获取模块依赖，并且将代码打包为浏览器可以识别的代码；
- 递归地构建一个依赖关系图（`dependency graph`），其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 `bundle`。



### 二、核心概念

#### 1、entry

入口（`Entry`）：指示 `webpack` 应该以哪个文件（模块）为入口起点开始打包，分析构建内部依赖图。



#### 2、output

输出（`Output`）：指示 `Webpack` 打包后的资源（ `bundles ` ）输出到哪里去，以及如何命名。基本上，整个应用程序结构,都会被编译到你指定的输出路径的文件夹中。



#### 3、module

模块，在 `Webpack` 里一切皆模块，**一个模块对应着一个文件**。`Webpack` 会从配置的 `entry` 开始递归找出所有依赖的模块。



#### 4、chunk

代码块，一个 `chunk` 由多个模块组合而成，用于**代码合并与分割**。



#### 5、loader

**`Loader ` 是模块转换器，用于把模块原内容按照需求转换成新内容**。例如：我们会使用 `Loader` 让 `Webpack` 能够去处理那些非 `JavaScript` 文件（`Webpack 自身只理解 JavaScript`）。然后就可以利用 `webpack` 的打包能力，对它们进行处理。



#### 6、plugins

用于扩展 `Webpack` 功能，在 `Webpack` 构建流程中的特定时机会广播出对应的事件，插件可以监听这些事件的发生，在特定时机做对应的事情。



#### 7、mode

模式（`Mode`）指示 `Webpack` 使用相应模式的配置，是使用开发模式（`development`）还是生产模式（`production`）。

- 开发模式
  - 方便于浏览器调试的工具；
  - 可以快速地对增加的内容进行编译；
  - 提供了更精确、更有用的运行时错误提示机制。
- 生产模式
  - 自动压缩构建输出的文件
  - 快速的运行时处理
  - 不暴露源代码和源文件的路径
  - 快速的静态资源输出



#### 8、devServer

通过 `devServer` 启动的 `Webpack` 会开启监听模式，当发生变化时重新执行构建，然后通知 `devServer` 会让 `Webpack` 在构建出的 `JavaScript` 代码里注入一个代理客户端用于控制网页，网页和 `devServer` 之间通过 `WebSocket` 协议通信，以方便 `devServer` 主动向客户端发送命令。`devServer` 在收到来自 `Webpack` 的文件变化通知时，通过注入的客户端控制网页刷新。

- 提供 `HTTP` 服务，而非使用本地文件预览
- 监听文件的变化并自动刷新网页，做到实时预览
- 支持 `SourceMap` 方便调试



### 三、构建流程

`Webpack` 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程 :

- 从 `entry` 里配置的 `module` 开始递归解析 `entry` 依赖的所有 `module`

- 每找到一个 `module`，就会根据配置的 `loader` 去找对应的转换规则

- 对 `module` 进行转换后，再解析出当前 `module` 依赖的 `module`

- 这些模块会以 `entry` 为单位分组，一个 `entry` 和其所有依赖的 `module` 被分到一个 `Chunk` 中

- 最后 `webpack` 会把所有 `Chunk` 转换成文件输出

- 在整个流程中 `webpack` 会在恰当的时机执行 `plugin` 里定义的逻辑



### 四、开发环境性能优化

- 优化打包构建速度
  - `HMR`
- 优化代码调试
  - `source-map`

#### 1、HMR

`HMR`（`hot module replacement`，热模块替换 / 模块热替换）的作用是：当一个模块发生变化，只会重新打包这一个模块，而不是打包所有模块，极大提升构建速度。

- 对于样式文件，我们不用手动再启动 `HMR` 功能，因为 `style-loader` 内部已经实现了；
- 对于 `JavaScript` 文件，默认是不能使用 `HMR` 功能的，需要修改 `JavaScript` 代码，添加支持 `HMR` 功能的代码；注意：`HMR` 功能对 `JavaScript` 的处理，只能处理非入口 `JavaScript` 文件的其他文件。

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/HMR



#### 2、source-map

`source-map` 是一种 提供源代码到构建后代码映射技术，如果构建后代码出错了，通过映射可以追踪源代码错误。



**内联和外部的区别：**

- 外部生成了文件，内联没有
- 内联构建速度更快



**用法区别：**

- `source-map`：外部，可以显示错误代码准确信息和源代码的错误位置；

- `inline-source-map`：内联，只生成一个内联 `source-map`，可以显示错误代码准确信息和源代码的错误位置；

- `hidden-source-map`：外部，显示错误代码错误原因，但是没有错误位置，不能追踪源代码错误，只能提示到构建后代码的错误位置；
- `eval-source-map`：内联，每一个文件都生成对应的 `source-map`，都在 `eval`，错误代码准确信息 和 源代码的错误位置，只是多了一个 `hash` 值；
- `nosources-source-map`：外部，显示错误代码准确信息，但是没有任何源代码信息；
- `cheap-source-map`：外部，显示错误代码准确信息和源代码的错误位置，只能精确到行；
- `cheap-module-source-map`：外部，错误代码准确信息 和 源代码的错误位置，`module` 会将 `loader` 的 `source map` 加入。



**如何选择**

- 开发环境：速度快（`eval>inline>cheap>...`），调试更友好（可以显示源代码信息）
  - 速度快：`eval-cheap-source-map`、`eval-source-map`
  - 调试更友好：`source-map`、`cheap-module-source-map`、`cheap-source-map`
  - 平衡点：`eval-source-map`  / `eval-cheap-module-source-map`

- 生产环境：隐藏源代码？调试要不要更友好？
  - 内联会让代码体积变大，所以在生产环境不用内联
  - `nosources-source-map` 全部隐藏
  - `hidden-source-map` 只隐藏源代码，会提示构建后代码错误信息
  - 也需要平衡：`source-map` / `cheap-module-source-map`

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/source-map



### 五、生产环境性能优化

- 优化打包构建速度
  - `oneOf`
  - `babel` 缓存
  - 多进程打包
  - `externals`
  - `dll`
- 优化代码运行的性能
  - 缓存（`hash-chunkhash-contenthash`）
  - `tree shaking`
  - `code splitting`
  - 懒加载/预加载
  - `pwa` 渐进式网络开发应用程序（离线可访问）



#### 1、oneOf

每个不同类型的文件在 `loader` 转换时，都会被命中，遍历 `module` 中 `rules` 中所有 `loader`，这也会影响性能。使用 `oneOf` 之后，当规则匹配时，只使用第一个匹配规则。

```javascript
module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.css$/,
        oneOf: [
          {
            resourceQuery: /inline/, // foo.css?inline
            use: 'url-loader'
          },
          {
            resourceQuery: /external/, // foo.css?external
            use: 'file-loader'
          }
        ]
      }
    ]
  }
};
```

**注意：**

- 使用 `oneOf` 根据文件类型加载对应的 `loader` 时，只要能匹配一个即可退出；
- 对于同一类型文件，比如处理 `js`，如果需要多个 `loader`，可以单独抽离 `js` 处理，确保 `oneOf` 里面一个文件类型对应一个 `loader` ；
- 可以配置 `enforce: 'pre',` 指定优先执行。

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/oneOf



#### 2、缓存

##### babel 缓存

因为在生产环境中不能使用 `HMR`，要想也达到 `HMR` 的效果，在更改一个模块之后，不至于让别的模块也重新打包，可以使用 `babel` 缓存。

```javascript
 use: {
   loader: 'babel-loader',
   options: {
     cacheDirectory: true
   }
 }
```

这样在第二次构建的时候，会读取之前的缓存，加快打包速度。



##### 文件资源缓存

> 一般情况下，对于前端静态资源，浏览器访问的时候希望资源都能够进行缓存，当第二次及以后进入页面的时候，页面就可以直接使用缓存资源，这样的话，页面打开速度很快，提高了用户体验同时也节省了带宽资源。而其中最为常见的一种最大化利用缓存的形式就是为静态资源加上 `hash`，使用一个不会重复的标识符来达到资源可以永久缓存的目的。

在 `Webpack` 打包的时候，我们可以给打包的文件名加上一个 `hash` 值，有两种方式：

- `hash`：每次 `wepack` 构建时会生成一个唯一的 `hash` 值
  - 问题：因为 `js` 和 `css` 同时使用一个 `hash` 值；如果重新打包，会导致所有缓存失效（可能我却只改动一个文件）

- `chunkhash`：根据 `chunk` 生成的 `hash` 值。如果打包来源于同一个 `chunk`，那么 `hash` 值就一样
  - 如何 `css` 是在 `js` 中被引入的，所以同属于一个 `chunk`，`js` 和 `css` 的 `hash` 值还是一样的。

- `contenthash`：根据文件的内容生成 `hash` 值。不同文件 `hash` 值一定不一样，这会让代码上线运行缓存更好使用（上线代码性能优化）。

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/cache



#### 3、多进程打包

`thread-loader` 一般是给 `loader` 用的。下面是一个在 `babel-loader` 中使用多进程打包的例子：

```javascript
{
  test: /\.js$/,
    exclude: /node_modules/,
      use: [
        {
          loader: 'thread-loader',
          options: {
            workers: 2 // 进程2个
          }
        },
        {
          loader: 'babel-loader',
          options: {
            // 预设：指示babel做怎么样的兼容性处理
            presets: [
              [
                '@babel/preset-env',
                {
                  // 按需加载
                  useBuiltIns: 'usage',
                  // 指定core-js版本
                  corejs: {
                    version: 3
                  },
                  // 指定兼容性做到哪个版本浏览器
                  targets: {
                    chrome: '60',
                    firefox: '60',
                    ie: '9',
                    safari: '10',
                    edge: '17'
                  }
                }
              ]
            ],
            // 开启babel缓存
            // 第二次构建时，会读取之前的缓存
            cacheDirectory: true
          }
        }
      ]
},
```

但是要注意，启动进程是需要时间的，进程间的通信也是需要时间的，如果本来构建时间就很快，这种情况下，不需要使用 `thread-loader` 。

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/thread-loader



#### 4、externals

`externals` 的作用是防止将某一些包打包到我们的输出中。假设我们的项目中使用了 `JQuery`，我们不希望将它打包到输出文件中，而是使用外部的 `CDN` 链接。

```javascript
// 忽略打包的文件
externals: {
  // 拒绝jQuery被打包进来
  // 忽略库名 -- npm 包名
  jquery: 'jQuery'
}
```

注意，当我们忽略打包文件需要在 `HTML` 中使用 `script` 标签将其引入进来的。

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/externals



#### 5、dll

使用 `dll` 技术，可以对某些库（一般是第三方库：`jquery`、`react`、`vue`...）进行单独打包。

**基本过程**

- 第一次的时候把请求的内容存储起来存储在映射表中；

- 再次请求时，先从映射表中找请求的内容，看是否有缓存，有则加载缓存（类似浏览器的缓存策略命中缓存），没有就正常打包。

**缺点**：配置很麻烦。

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/dll





#### 5、tree shaking

`tree shaking` 的作用是去除无用代码，让代码的体积更小。开启 `production` 环境后，默认就会启动 `tree shaking` 。

但是对于 `JavaScript` 文件来说，还有一个前提条件：必须使用 `ES6` 模块化。

最后还有一个问题需要注意，可能由于 `Webpack` 版本的原因，会把一些 `css` 资源给 `shaking` 掉，所以最好在 `package.json` 中配置一下：

```javascript
"sideEffects": [
  "*.css",
  "*.less"
]
```

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/tree-shaking





#### 6、code splitting

在最开始使用 `Webpack` 的时候，都是将所有的 `js` 文件全部打包到一个 `build.js` 文件中，但是在大型项目中，`build.js` 可能过大, 导致页面加载时间过长。这个时候就需要 `code splitting`。

`code splitting` 就是将文件分割成块（`chunk`）, 我们可以定义一些分割点（`split point`），根据这些分割点对文件进行分块，并实现按需加载。

大致有三种方式来配置：

- 添加多个入口 `entry`
  - 缺点：不好维护，每次都需要新添加入口配置
- 使用 `splitChunks` 配置
  - 可以将 `node_modules` 中代码单独打包一个 `chunk` 最终输出
  - 它也会自动分析多入口 `chunk` 中，有没有公共的文件，如果有则会打包成单独一个 `chunk`
- 通过 `js` 代码，让某个文件被单独打包成一个 `chunk`
  - `import` 动态导入语法：能将某个文件单独打包

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/code-splitting



#### 7、懒加载和预加载

- 懒加载（延迟加载）：当文件需要使用时才加载。

  ```javascript
  document.getElementById('btn').onclick = function() {
    import(/* webpackChunkName: 'test' */'./test').then(({ mul }) => {
      console.log(mul(4, 5));
    });
  };
  ```

- 预加载 `prefetch`：会在使用之前，提前加载 `js` 文件，等其他资源加载完毕，浏览器空闲了，再偷偷加载资源（兼容性比较差）。

  ```javascript
  document.getElementById('btn').onclick = function() {
    import(/* webpackChunkName: 'test', webpackPrefetch: true */'./test').then(({ mul }) => {
      console.log(mul(4, 5));
    });
  };
  ```

> https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/lazy-loading



#### 8、PWA

渐进式网络开发应用程序（离线可访问），使用 `workbox-webpack-plugin` 插件来实现。有两个步骤：

1、在 `Webpack` 中配置：

```javascript
new WorkboxWebpackPlugin.GenerateSW({
  /*
    作用：
      1. 帮助serviceworker快速启动
      2. 删除旧的 serviceworker，生成一个 serviceworker 配置文件
  */
  clientsClaim: true,
  skipWaiting: true
})
```

2、注册 `serviceWorker`，并处理兼容性问题：

```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker
      .register('/service-worker.js')
      .then(() => {
        console.log('sw注册成功了~');
      })
      .catch(() => {
        console.log('sw注册失败了~');
      });
  });
}
```

另外，`serviceWorker` 必须在服务器上。

> 例子：https://github.com/IDeepspace/webpack-performance-enhancement-example/tree/master/pwa
