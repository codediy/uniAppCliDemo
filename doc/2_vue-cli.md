# vue-cli

## TOC
- Basic
- vue-cli
- uni的cli扩展

## Basic
- `uni_service`通过扩展`vue-cli`注册了`vue-cli-service`的`uni-service`和`uni-build`两个命令

## `vue-cli`
- [vue-cli插件开发官方文档](https://cli.vuejs.org/zh/dev-guide/plugin-dev.html#%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5)

### 核心概念
1. `@vue/cli-service` 暴露了`vue-cli-service`命令
2. `Service`是调用`vue-cli-service <command> [...args]`时创建的类,
3. `Service`主要用来管理内部的webpack配置,提供服务和构建项目的命令

### cli插件
1. cli插件主要为`@vue/cli`提供扩展功能,
2. cli插件必须包含一个`Service插件`作为主要导出,可以额外提供`Generator`和`Prompt`作为`vue create <app>`命令的扩展
3. cli插件目录结构
```
plugin\
    index.js        # service扩展入口
    generator.js    # generator扩展入口(可选)
    prompts.js      # promopt扩展入口(可选)
    package.json
    README.md
```

### `service`插件结构
1. `vue-cli-service`命令执行时,创建`service`的实例
2. `service`插件导出一个函数,用来扩展`vue-cli`的默认`service`实例
3. `service`插件基本结构(index.js)
```js
//@param api            默认service,在插件内部修改webpack和扩展命令
//@param projectOptions 项目配置信息
module.exports = (api, projectOptions) => {
  // 通过 webpack-chain 修改 webpack 配置
  api.chainWebpack(webpackConfig => {
  })
  
  // 
  // 通过 webpack-merge 修改 webpack 配置
  api.configureWebpack(webpackConfig => {
  })
  
  // 注册命令 `vue-cli-service test`
  api.registerCommand('test', args => {
  })
}
```
4. `service`扩展命令可以指定在特定模式下运行
```js
module.exports = api => {
  //注册build命令
  api.registerCommand('build', () => {
    // ...
  })
}
//build只可以在production模式下运行
module.exports.defaultModes = {
  build: 'production'
}
```
5. `service`中webpack的操作
```js
module.exports = api => {
  api.registerCommand('my-build', args => {
    //获得一个新生成的链式配置
    const configA = api.resolveChainableWebpackConfig()
    const configB = api.resolveChainableWebpackConfig()
    //获取已解析好的webpack配置
    const configC = api.resolveWebpackConfig()
    const configD = api.resolveWebpackConfig()
    // 针对不同的目的修改 `configA` 和 `configB`...
  })
}

// 请确保为正确的环境变量指定默认模式
module.exports.defaultModes = {
  'my-build': 'production'
}
```

### `service`插件发布
1. `cli`插件使用`vue-cli-plugin-<name>`格式发布到npm上 
2. 命名约定的插件包可以被`@vue/cli-service`发现,
3. 命令约定的插件包可以使用`vue add <name>`或者`vue invoke <name>`安装

## uni的cli扩展
1. uni的`vue-cli`扩展包在`vue-cli-plugin-uni`中

