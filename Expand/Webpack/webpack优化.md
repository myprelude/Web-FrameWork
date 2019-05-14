# webpack优化方案
>作者：myprelude@github  
原文链接： https://github.com/myprelude/Web-FrameWork/blob/master/README.md  
转载请注明出处，保留原文链接和作者信息。

### 缩小文件范围
通常我们配置如下:
```
module.exports = {
  //...
  module: { 
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader'
      }
    ]
  }
};
```
优化后配置如下：
```
module.exports = {
  //...
  //字段告诉Webpack不必解析哪些文件，可以用来排除对非模块化库文件的解析
  noParse:[/jquery|chartjs/, /react\.min\.js$/]
  module: {
    rules: [
      {
        test: /\.js$/,
        // 缩小查找范围
        include: path.resolve(__dirname, 'src'),
        loader: 'babel-loader'
      }
    ]
  },
  ,
  resolve:{
      // 使webpack直接使用库的min文件，避免库内解析
        alias:{
            'react':patch.resolve(__dirname, './node_modules/react/dist/react.min.js')
        }
    }
};
```
### 开启代码切割
* 公共代码单独打包
```
module.exports = {
    entry:{
        main:'./src/index.js',
        common:'./src/commom/index.js' // 公共js代码
    }
```
* 对引入相同的模块进行提取
```
module.exports = {
    //...
    optimization: {
        minimize: false,
        splitChunks: {
            chunks: 'async',
            minSize: 30000,
            maxSize: 0,
            minChunks: 1,
            maxAsyncRequests: 5,
            maxInitialRequests: 3,
            automaticNameDelimiter: '~',
            name: true,
            cacheGroups: {
                vendors: {
                    test: /[\\/]node_modules[\\/]/,
                    priority: -10
                },
                default: {
                    minChunks: 2,
                    priority: -20,
                    reuseExistingChunk: true
                }
            }
        }
    }
};
```

### DllPlugin减少基础模块编译次数
DllPlugin动态链接库插件，其原理是把网页依赖的基础模块抽离出来打包到dll文件中，当需要导入的模块存在于某个dll中时，这个模块不再被打包，而是去dll中获取。
```
const path = require('path');
const DllPlugin = require('webpack/lib/DllPlugin');
module.exports = {
 output:{
     //  ...
     library:'_dll_[name]',  //dll的全局变量名
 },
 plugins:[
     new DllPlugin({
         name:'_dll_[name]',  //dll的全局变量名
         path:path.join(__dirname,'dist','[name].manifest.json'),//描述生成的manifest文件
     })
 ]
}
```
### happypack
由于运行在 Node.js 之上的 Webpack 是单线程模型的，所以Webpack 需要处理的事情需要一件一件的做，不能多件事一起做。
我们需要Webpack 能同一时间处理多个任务，发挥多核 CPU 电脑的威力，HappyPack 就能让 Webpack 做到这点，它把任务分解给多个子进程去并发的执行，子进程处理完后再把结果发送给主进程。
```
const HappyPack = require('happypack');
const os = require('os');
const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });

module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        //把对.js 的文件处理交给id为happyBabel 的HappyPack 的实例执行
        loader: 'happypack/loader?id=happyBabel',
        //排除node_modules 目录下的文件
        exclude: /node_modules/
      },
    ]
  },
plugins: [
    new HappyPack({
        //用id来标识 happypack处理那里类文件
      id: 'happyBabel',
      //如何处理  用法和loader 的配置一样
      loaders: [{
        loader: 'babel-loader?cacheDirectory=true',
      }],
      //共享进程池
      threadPool: happyThreadPool,
      //允许 HappyPack 输出日志
      verbose: true,
    })
  ]
}
```
`由于HappyPack 对file-loader、url-loader 支持的不友好，所以不建议对该loader使用。还有注意点就是使用happypack时候需要就use里面的option除去。`
### tree shaking
tree shaking 指的是将我们代码中没有用到大代码在打包的时候删除了；如下：
```
// until.js
export function square(x) {
  return x * x;
}
export function cube(x) {
  return x * x * x;
}
// index.js
import { cube } from './math.js';
console.log(cubu(4))

打包后until中的square将不会被打包到bundle中。
```
配置如下：
```
module.exports = {
    optimization: {
        usedExports: true
    }
}
```
### 启用 HMR 
它允许在运行时更新所有类型的模块，而无需完全刷新，极大的方便开发效率。
```
const CleanWebpackPlugin = require('clean-webpack-plugin');
const webpack = require('webpack');
module.exports = {
    entry: {
        app: './src/index.js'
    },
    devtool: 'inline-source-map',
    devServer: {
        contentBase: './dist',
        hot: true  // 开启
    },
    plugins: [
        // 开启
     new webpack.HotModuleReplacementPlugin()
    ]
};
```
### 动态加载
当涉及到动态代码拆分时，webpack 提供了两个类似的技术。第一种，也是推荐选择的方式是，使用符合 ECMAScript 提案 的 import() 语法 来实现动态导入。第二种，则是 webpack 的遗留功能，使用 webpack 特定的 require.ensure。

```
// index.js
document.body.addEventListener('click',()=>{
    // import
    import('loadsh').then((_)=>{
        console.log(_.add(1+2))
    }).catch(e){
        console.log(`loadsh加载失败`)
    }
    // require.ensure
    require.ensure(['loadsh'],(_)=>{
        console.log(_.add(1+2))
    })
})
```

### Scope Hoisting
Scope Hoisting 可以让 Webpack 打包出来的代码文件更小、运行的更快，它又译作 "作用域提升",下面是详细介绍：

util.js
```
export default 'Hello,Webpack';
```
main.js
```
import str from './util.js';
console.log(str);
```
直接打包部分代码如下：
```
[
  (function (module, __webpack_exports__, __webpack_require__) {
    var __WEBPACK_IMPORTED_MODULE_0__util_js__ = __webpack_require__(1);
    console.log(__WEBPACK_IMPORTED_MODULE_0__util_js__["a"]);
  }),
  (function (module, __webpack_exports__, __webpack_require__) {
    __webpack_exports__["a"] = ('Hello,Webpack');
  })
]
```
在开启 Scope Hoisting 后，同样的源码输出的部分代码如下
```
[
  (function (module, __webpack_exports__, __webpack_require__) {
    var util = ('Hello,Webpack');
    console.log(util);
  })
]
```
从中可以看出开启 Scope Hoisting 后，函数申明由两个变成了一个，util.js 中定义的内容被直接注入到了 main.js 对应的模块中。

`由于 Scope Hoisting 需要分析出模块之间的依赖关系，因此源码必须采用 ES6 模块化语句，不然它将无法生效。`

配置如下：
```
module.exports = {
    resolve: {
        // 针对 Npm 中的第三方模块优先采用 jsnext:main 中指向的 ES6 模块化语法的文件
        mainFields: ['jsnext:main', 'browser', 'main']
    },
    plugins: [
        // 开启 Scope Hoisting 功能
        new webpack.optimize.ModuleConcatenationPlugin()
    ]
};
```
### 预取/预加载模块
在声明 import 时，使用下面这些内置指令，可以让 webpack 输出 "resource hint(资源提示)"，来告知浏览器：

* prefetch(预取)：将来某些导航下可能需要的资源
* preload(预加载)：当前导航下可能需要资源

下面这个 prefetch 的简单示例中，有一个 HomePage 组件，其内部渲染一个 LoginButton 组件，然后在点击后按需加载 LoginModal 组件。
```
import(/* webpackPrefetch: true */ 'LoginModal');
```
`这会生成 <link rel="prefetch" href="login-modal-chunk.js"> 并追加到页面头部，指示着浏览器在闲置时间预取 login-modal-chunk.js 文件。`