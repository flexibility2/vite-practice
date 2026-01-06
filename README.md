# vite-practice

# 面试考点

## vite 的核心概念

1. 快速冷启动
2. 即时热更新
3. 原生 ES 模块支持
4. 插件体系
5. 按需编译
6. 生产构建优化
7. 环境变量
8. 现代浏览器支持

## 说下 vite 为什么开发环境使用 esbuild，线上构建使用 rollup？

## 说说你对 bundless 的理解？

# 基础概念

1. vite 配置里面没有 entry，入口默认是 src
   vite 默认是 web 应用， **index.html 的 type="module",src=""**是入口文件。但是只能处理 js 文件

2. vite 在开发环境使用的 esbuild 进行编译，代码转换和优化，esbuild 是用 go 语言编写的，性能非常高，能够极大提升冷启动速度和热更新速度。esbuild 支持 js，ts，react，vue 等多种语言和框架。
   之前的 js 的使用的 Babel 是不行的，Babel 是用 js 编写的，性能较低，无法满足 vite 对性能的要求。
   vite 里面装的 react，vue 的插件只是用来做优化的

   对于样式文件，vite 使用 postcss 进行处理，postcss 是用 js 编写的，性能较低，但是样式文件的处理时间相对较短，对整体性能影响不大。

   综上，在模块解析时，vite 是按照需要进行解析，并会将结果缓存起来，避免重复解析，提高性能。

3. 另外一些配置

- 针对性功能增加，plugin 配置
- 模块解析，resolve
- 本地开发构建服务，server
- 样式额外处理，css 配置
- vite 环境变量，define 配置（一般用 env 文件，.env， 且以 VITE 开头的变量）
- 产物构建，build，esbuild 配置

## vite 的插件体系

和 webpack 不同
webpack 的插件是用来强化构建过程，通过暴露的插件时机钩子在不同的时机做一些处理
需要和 loader 区分开。loader 的工作是模块编译和解析，插件是用于功能增强。

vite 的插件本质是一个函数。
vite 没有 loader，但是 vite 插件在开发服务启动时，会调用 Rollup 的构建钩子：
options
buildStart
resolveId,load, transform
允许插件在不同的阶段介入构建过程

vite 的插件体系是基于 rollup 的插件体系进行扩展的，vite 插件可以使用 rollup 插件，也可以使用 vite 专用的插件。vite 插件可以分为两类，一类是开发环境插件，一类是生产环境插件。开发环境插件主要用于处理模块解析，热更新等功能，生产环境插件主要用于代码压缩，优化等功能。vite 插件可以通过配置文件进行配置，也可以通过命令行参数进行配置。

## vite 的原理

**借助了浏览器的 module 支持**。会开一个服务，把代码以模块的形式提供给浏览器。

浏览器请求模块时，vite 会根据请求的模块路径，找到对应的文件，并进行编译和转换，然后将编译后的代码返回给浏览器。浏览器接收到代码后，会进行解析和执行。vite 会对模块进行按需编译，只有当模块被请求时，才会进行编译和转换，这样可以大大提高冷启动速度和热更新速度。
**需要编译的内容做额外的编译处理** css,ts

**两端**

- 本地通过开发服务器借助 esbuild 实现
- Rollup 打包

例子：

1. 插件处理 config，配置问题，比如修改端口
2. 插件处理编译工作, generateBundle
3. 插件处理编译内容输出，transform

## 整体架构设计

- 开发服务器
- 模块解析与热更新
- 构建工具链（生产环境 Rollup 打包）
- 插件系统

### 开发服务器原理

Vite 的开发服务器是基于 Node.js 的 http 模块实现的。它会监听文件系统的变化，当文件发生变化时，会通知浏览器进行热更新。

- 启动服务器时，createServer 会创建一个 http 服务器，并监听指定的端口。
  vite 会扫描项目目录下的所有文件，并建立一个文件映射表，用于快速定位文件路径。
- 处理请求: 添加中间件，包括静态资源处理，模块解析，热更新等功能。

### 模块解析与转化

- 模块解析，使用 es-module-lexer 进行静态分析，提取模块的导入导出信息。
- 模块转化，使用 esbuild 进行代码转换和优化。
- 缓存机制，vite 会对编译后的模块进行缓存，避免重复编译，提高性能。
- 文件类型处理，通过 plugins 处理不同类型的文件，如 vue，react，css 等。

### 构建工具链

- 生产环境使用 Rollup 进行打包构建。TreeShaking 是 Rollup 的核心功能之一，可以有效减少最终产物的体积。
- 代码分割，Rollup 支持动态导入，可以实现代码分割，

# webpack

## 1. webpack 核心概念

1. 入口 Entry
2. 输出 Output
3. 加载器 Loaders
4. 插件 Plugins
5. 模块 Module
6. 模式 Mode

## 2. webpack 的 loader 和 plugin

1. 常见的 loader

   - babel-loader： 用于将 ES6+代码转换为向后兼容的 JavaScript 代码。(ES5)
   - css-loader：使你可以使用类似@import 和 url()的方法实现 require()的功能来引入 CSS 文件。
   - style-loader：将 CSS 插入到 DOM 中的 `<style>` 标签中。
   - sass-loader：将 Sass/SCSS 文件编译为 CSS。
   - file-loader：webpack5 内置了 Asset Modules，可以处理文件资源。
   - url-loader：类似于 file-loader，但可以将小文件转换为 base64 URI。

2. 常见的 plugin
   - html-webpack-plugin：用于简化 HTML 文件的创建，自动引入打包后的资源。
   - mini-css-extract-plugin：用于将 CSS 提取到单独的文件中，支持 CSS 代码分割。
   - clean-webpack-plugin：用于在每次构建前清理输出目录。
   - webpack-bundle-analyzer：用于可视化分析打包后的文件大小。
   - define-plugin：用于定义全局常量，替换代码中的变量。

## 3. 请详细说下 webpack5 的构建过程

webpack5 的构建过程主要包括以下几个阶段：

1. 初始化阶段
   - 读取并合并配置文件（webpack.config.js）。
   - 根据配置文件中的 entry 选项，确定入口文件。
2. 构建依赖图
   - 从入口文件开始，分析模块之间的依赖关系，构建出一个完整的依赖图。
   - 解析模块。使用 Loaders 处理非 JS 文件，如 CSS、图片等。
   - 创建模块对象。Webpack 为每个模块创建一个模块对象，并保存在内存中
3. 模块编译
   Webpack 使用 Loader 将模块的源代码转换为可以在浏览器中运行的 JS 代码。
   - 处理模块
   - 生成 AST
   - 收集依赖
4. 生成代码块（Chunks）
   Webpack 会根据依赖图将模块分组，形成不同的代码块（Chunks）。这些代码块会被打包成一个或者多个最终的输出文件。
   - 代码拆分： 根据配置里面的 optimization.splitChunks 进行代码拆分。
   - 生成代码块，Chunk 对象
5. 优化阶段
   - 代码压缩：使用 TerserPlugin 对 JS 代码进行压缩，减少文件大小。
   - CSS 压缩：使用 css-minimizer-webpack-plugin 对 CSS 代码进行压缩。
   - Tree Shaking：去除未使用的代码，减小打包体积。
   - 作用域提升：模块合并，提升运行效率。
   - 其他优化：如懒加载、预加载等。
6. 输出阶段
   Webpack 将每个代码块转换为一个或者多个输出文件，并将其写入到磁盘上。
   - 生成输出文件：根据 output 配置生成最终的输出文件。
   - 写入磁盘：将生成的文件写入到指定的输出目录。
   - 应用插件：会调用相关的插件，如 HtmlWebpackPlugin，来处理输出文件

## webpack 笔记

### 1 基础配置

1. 入口 Entry

- 单入口
- 多入口： 使用列表
- 对象形式： 使用对象定义（多个产物）

2. 输出配置（output）

- 输出为位置： path
- 输出文件名： filename
- 输出 chunk 文件名：chunkFilename
- 环境配置： environment
- 清理输出目录： clean

3. 模块解析(module,里面配置对不同文件的 rule)
   针对不同的文件类型，做转换处理，不同的 loader
   -js
   -ts
   -image
   -font,
   -css,
   -file

4. 辅助解析(resolve)
   resolve 配置了之后，在 import 的时候，可以省略文件扩展名

5. devtool

6. plugins

7. optimization
   比如 splitChunks

8. devServer

### 2 自定义配置

1. 自定义 loader
   本质是函数

2. 自定义 plugin
   本质是类

# webpack 和 vite 的区别

前提
**浏览器原生的 ES Module 能力，指的是现代浏览器直接支持通过 <script type="module"> 标签加载和解析 JavaScript 模块（ESM），并且可以使用 import 和 export 语法来进行模块化开发。**

主要特点：

模块化语法：可以在 JS 文件中直接使用 import 和 export。
按需加载：浏览器会自动根据依赖关系加载模块文件，实现懒加载和代码分割。
作用域隔离：每个模块有自己的作用域，变量不会污染全局。
网络请求：浏览器会根据 import 路径自动发起网络请求加载对应的模块文件。
这意味着开发环境下，浏览器可以直接加载和运行模块化的 JS 文件，无需像 webpack 那样先打包合并。Vite 就是利用了这一能力，实现了更快的开发体验。

# 说下你使用过哪些构建工具

1. Webpack

- 打包产物是 iife/umd 格式的

2. Rollup

- 多打包产物构建（umd,cmj,es）【多模块化规范输出】
- 完全拥抱插件化，常规的功能都是功过插件实现的 （插件都是函数）

3. 基于 go 语言的 esbuild
   但是针对 ts 无法生成 dts（可参考 tsup，利用 rollup 的插件）

4. Vite

# 模块化规范

1. CommonJS
   这个规范主要用于服务端编程，Node.js 就是基于 CommonJS 规范实现的。加载模块是同步的，不适合浏览器环境，同步意味着阻塞加载，浏览器是异步加载的
   因此有了 AMD，CMD 的方案。
2. AMD （require.js）
   这个规范主要用于浏览器端编程，支持异步加载模块，适合浏览器环境。AMD 使用 define 函数定义模块，使用 require 函数加载模块。
   AMD 的缺点是语法复杂，不够直观，而且不支持循环依赖。
3. CMD （sea.js）
   和 AMD 类似，并实现了就近依赖。
4. UMD (工厂函数) 可同时满足 AMD，CMD 和 CommonJS 规范
5. ES Module （ECM， vite 能实现 bundless，完全依赖于这个特性）。ES6 在语言标准层面上，实现了模块功能，而且实现的简单，完全可以取代 commonJS 和 AMD。
   成为浏览器和服务器通用的模块化规范。

## 补充

tree-shaking 必须是 esm 规范

# 一些工程规范

- ESLint
- Prettier
- Commitlint
- Husky
- Lint-staged
- Editorconfig
- Spell-checker
