# webpack基本配置
>作者：myprelude@github  
原文链接： https://github.com/myprelude/Web-FrameWork/blob/master/README.md  
转载请注明出处，保留原文链接和作者信息。

### 前言
随着前端的发展，前端的项目工程目录越来越大，对前端工程化需求越来越强烈，从grunt到gulp再到现在的webpack无不体现了前人们对工程化所作出对努力。现在出去面试，关于webpack的面试题是必考项目。虽说现在很多集成好的cli命令一键生成配置，但是作为一个合格的前端工程师还是有必要自己配置打包方式和了解整个过程原理。

### webpack 4.x 的变动
随着大量对webpack配置麻烦的吐槽，webpack官方团队终于认识到打包工具应该有默认配置。下面我们对webpack4.0后的变动简单的总结。

* 零配置 -- 提供了默认的入口和打包配置，不需要我们引入webpack.config.js文件。当然我们仍然可以自己去配置。默认入口是当前运行目录src文件中的index.js。默认打包到当前运行目录dist文件中。

* 区分环境 -- 添加了mode属性，默认是production。在4.0以前我们通过设置evn来区分环境；4.0以后通过mode 设置为production/development来区分环境；而且webpack会根据不同的环境来开启打包优化。

* 优化配置 -- 移除了CommonsChunkPlugin，并用内置的optimization.splitChunks和optimization.runtimeChunk.为我们提供最优的公共代码提取方式。以前很多通过loader实现对功能都转手到webpack自己。

* 插件变动 -- 整个Plugin的事件注册和事件触发机制完全重写了，导致第三方插件需要重写。如Extract-text-webpack-plugin 改成mini-css-webpack-plugin。

### webpack 核心配置

`注意以下的配置都是在下面代码块中`

```
module.exports = {
    mode:'',
    entry:'',
    output:{/** **/},
    module:{/** **/},
    plugins:{/** **/}
}
```

* entry -- 打包入口（webpack根据这个入口开始打包项目）

```
// 单页配置
entry:{
    main:'./src/index.js'
}

entry:'./src/index.js' // 单页配置的简写

entry:['.src/index.js'] // 数组形式

// 多页配置
entry:{
    index:'./src/index.js',
    about:'./src/about.js',
    detail:'./src/detail.js'
}

entry:['.src/index.js','./src/about.js'.'./src/detail.js']

// 动态配置
entry:{
    main:()=>'.src/index.js'
}
```
* output -- 打包出口 (webpack根据这个出口输出打包项目)

```
output:{
    // 输出路径
    path: path.resolve(__dirname, 'dist'),

    // 输出文件名 提供占位符
    // [id] 模块标识符(module identifier)
    // [hash] 模块标识符(module identifier)的 hash
    // [name] 模块名称
    // [chunkhash] chunk 内容的 hash
    // [query] 模块的 query，例如，文件名 ? 后面的字符串
    filename: 'webpack.[name].bundle.js'

    更多其他配置：https://www.webpackjs.com/configuration/output/
}
```
* loader -- 将所有类型的文件转换为 webpack 能够处理的有效模块
```
module:{
    // 防止 webpack 解析那些任何与给定正则表达式相匹配的文件
    noParse:/jquery|lodash/

    // loader 规则
    rules:[
        {
            test:/\.css$/,
            // 使用的loader集合
            use:[
                'style-loader', // 支持字符串
                {
                    loader:'css-loader',
                    options:{

                    }
                }
            ],
            include: [
                path.resolve(__dirname, "src/css")
            ],
            exclude: [
                path.resolve(__dirname, "node__module")
            ],
        }
    ]
}
更多配置项：https://www.webpackjs.com/configuration/module/
loader集合：https://www.webpackjs.com/loaders/
```

* plugin 插件 -- 处理各种各样的loader无法完成的任务

```
const HtmlWebpackPlugin = require('html-webpack-plugin');
...
plugins:[
    new HtmlWebpackPlugin({template: './src/index.html'})
]
更多插件：https://www.webpackjs.com/plugins/
```
* mode -- 开发环境/模式 (webpack4.0 使用相应模式的内置优化)

```
mode: 'production' // 生产模式 
// webpack 默认生成环境开启的插件优化
-  plugins: [
-    new UglifyJsPlugin(/* ... */),
-    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
-    new webpack.optimize.ModuleConcatenationPlugin(),
-    new webpack.NoEmitOnErrorsPlugin()
-  ]

mode: 'development' // 开发模式
// webpack 默认开发环境开启的插件
- plugins: [
-   new webpack.NamedModulesPlugin(),
-   new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
- ]

// 通过 webpack-cli 参数指定模式
webpack --mode=production
```
### webpack 其他配置
* devServer -- 启动本地服务器方便开发

```
desServer:{
    //告诉服务器从哪里提供内容。只有在你想要提供静态文件时才需要
    contentBase: path.join(__dirname, "dist"),

    // 开启压缩
    compress: true,

    // 开启浏览器
    open: true,

    // 启用 webpack 的模块热替换特性
    hot: true,

    // 开启端口
    port: 9000,

    // 开启代理
    proxy:{
        "/api": {
            target: "http://localhost:3000",
            pathRewrite: {"^/api" : ""}
        }
    }
}

更多配置：https://www.webpackjs.com/configuration/dev-server/
```
* resolve -- 设置模块如何被解析

```
resolve:{
    // 通过别名来映射真实模块地址 
    alias: {
        Utilities: path.resolve(__dirname, 'src/utilities/'),
        Templates: path.resolve(__dirname, 'src/templates/')
    },

    // 自动解析确定的扩展
    extensions:[json,js],
} 
// 和resolve配置相同不过是解析 loader模块
resolveLoader:{
 
}
```

* optimization -- 从 webpack 4 开始，会根据你选择的 mode 来执行不同的优化。个人觉得官方提供的默认的方案已经能满足我们大部分情况了，如果需要自己配置可以查看：[优化optimization](https://webpack.docschina.org/configuration/optimization/)


### 总结
 哈哈我猜到很多看官要拿40米大刀来砍我，这个说的根本谈不上教程就是copy官网上的代码；本文并没有和网上其他教程一样将webapck的配置写一遍分别说下这个配置是什么；本篇说明webpack最重要的几个核心配置并附上详细的官方文档信息；之所以这样做是因为webpack更新比较快，每次更新网上都一大堆更新后的配置，搜索配置后还要看看是不是我当前使用的版本；我们给定核心配置保证你能执行起来，同时提供官方文档详细配置信息这样我们可以边查边配置；这样webpack不管怎么升级我们都可以搞定。