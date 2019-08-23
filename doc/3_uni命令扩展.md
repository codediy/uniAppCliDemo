# uniService扩展

## TOC
- Basic
- module.exports
- ServeCommand
- BuildCommand
- 总结

## Basic
1. uni-service主要使用了webpack的功能
## module.exports

```js 
//vue-cli-plugin-uni/index.js
module.exports = (api, options) => {
    //注册命令
    initServeCommand(api, options)
    initBuildCommand(api, options)

    //webpack配置修改
}
```

## ServeCommand
### 导入Serve命令包
```js
//vue-cli-plugin-uni/index.js
const initServeCommand = require('./commands/serve')
```
### Serve命令注册
```js
//vue-cli-plugin-uni/commands/serve.js
module.exports = (api, options) => {
   // 注册uni-serve命令 
   api.registerCommand('uni-serve', {
    description: 'start development server',
    usage: 'vue-cli-service uni-serve [options] [entry]',
    options: {
      '--open': `open browser on server start`,
      '--copy': `copy url to clipboard on server start`,
      '--mode': `specify env mode (default: development)`,
      '--host': `specify host (default: ${defaults.host})`,
      '--port': `specify port (default: ${defaults.port})`,
      '--https': `use https (default: ${defaults.https})`,
      '--public': `specify the public network URL for the HMR client`
    }
  }, async function serve (args) {
      //命令实现
      ...<uni-serve-imp>
  } 
}    
```
### Serve命令实现
```js
//vue-cli-plugin-uni/commands/serve.js
// <uni-serve-imp>

// 命令行提示语,在HbuilderX点击运行后会出现
info('Starting development server...');
//一堆包导入
...
//获取已解析的webpack配置
const webpackConfig = api.resolveWebpackConfig()
//检查webpack配置
validateWebpackConfig(webpackConfig, api, options)
//重新生成devServer内容
const projectDevServerOptions = Object.assign(
    webpackConfig.devServer || {},
    options.devServer
)
//仪表盘插件
if (args.dashboard) {
      const DashboardPlugin = require('@vue/cli-service/lib/webpack/DashboardPlugin');
      (webpackConfig.plugins = webpackConfig.plugins || []).push(new DashboardPlugin({
        type: 'serve'
      }))
    }
//webpack入口entry配置
const entry = args._[0]
if (entry) {
    webpackConfig.entry = {
    app: api.resolve(entry)
    }
}
//url解析生成
const useHttps = args.https || projectDevServerOptions.https || defaults.https
const protocol = useHttps ? 'https' : 'http'
const host = args.host || process.env.HOST || projectDevServerOptions.host || defaults.host
portfinder.basePort = args.port || process.env.PORT || projectDevServerOptions.port || defaults
    .port
const port = await portfinder.getPortPromise()
const rawPublicUrl = args.public || projectDevServerOptions.public
const publicUrl = rawPublicUrl
    ? /^[a-zA-Z]+:\/\//.test(rawPublicUrl)
        ? rawPublicUrl
        : `${protocol}://${rawPublicUrl}`
    : null

const urls = prepareURLs(
    protocol,
    host,
    port,
    isAbsoluteUrl(options.publicPath) ? '/' : options.publicPath
)

//代理生成
const proxySettings = prepareProxy(
    projectDevServerOptions.proxy,
    api.resolve('public')
)

//dev模式热加载
if (!isProduction) {
    const sockjsUrl = publicUrl
    // explicitly configured via devServer.public
    ? `?${publicUrl}/sockjs-node`
    : isInContainer
    // can't infer public netowrk url if inside a container...
    // use client-side inference (note this would break with non-root publicPath)
        ? ``
    // otherwise infer the url
        : `?` + url.format({
        protocol,
        port,
        hostname: urls.lanUrlForConfig || 'localhost',
        pathname: '/sockjs-node'
        })
    const devClients = [
    // dev server client
    require.resolve(`webpack-dev-server/client`) + sockjsUrl,
    // hmr client
    require.resolve(projectDevServerOptions.hotOnly
        ? 'webpack/hot/only-dev-server'
        : 'webpack/hot/dev-server')
    // TODO custom overlay client
    // `@vue/cli-overlay/dist/client`
    ]
    if (process.env.APPVEYOR) {
    devClients.push(`webpack/hot/poll?500`)
    }
    // inject dev/hot client
    addDevClientToEntry(webpackConfig, devClients)
}

//webpack打包编译器 ☆☆uniApp的编译入口☆☆
const compiler = webpack(webpackConfig)

//创建serve实例
const server = new WebpackDevServer(compiler,
    //<server-config>见webpackDevServer
    ...config

    //返回Promise
    return new Promise((resolve, reject) => {
        
        cimpiler.hoos.done.tap('vue-cli-service uni-serve',stats => {
            //Hbuiderx 运行提示
            console.log()
            console.log(`  App running at:`)
            console.log(
            `  - Local:   ${chalk.cyan(urls.localUrlForTerminal)} ${copied}`
            )

            //
            if (!isInContainer) {}else{}

            //收藏编译
            if (isFirstCompile) {
                ...
                //打开浏览器
                if (args.open || projectDevServerOptions.open) {
                    const pageUri = (projectDevServerOptions.openPage && typeof projectDevServerOptions
                    .openPage === 'string')
                    ? projectDevServerOptions.openPage
                    : ''
                    openBrowser(urls.localUrlForBrowser + pageUri)
                }
                ...
                resolve({
                    server,
                    url: urls.localUrlForBrowser
                })
            }elseif(){

            }
        });

        //启动服务器监听端口
        server.listen(port, host, err => {
            if (err) {
            reject(err)
            }
        })
    }
);

```

## BuildCommand

### 导入Build命令包
```js
//vue-cli-plugin-uni/index.js
const initBuildCommand = require('./commands/build')
```

### Build命令实现
```js
//vue-cli-plugin-uni/commands/build.js
module.exports = (api, options) => {
  //注册uni-build命令
  api.registerCommand('uni-build', {
    description: 'build for production',
    usage: 'vue-cli-service uni-build [options]',
    options: {
      '--watch': `watch for changes`,
      '--minimize': `Tell webpack to minimize the bundle using the TerserPlugin.`
    }
  }, async (args) => {

    //命令实现  
    for (const key in defaults) {
      if (args[key] == null) {
        args[key] = defaults[key]
      }
    }

    args.entry = args.entry || args._[0]

    process.env.VUE_CLI_BUILD_TARGET = args.target

    //build过程
    await build(args, api, options)

    delete process.env.VUE_CLI_BUILD_TARGET
  })
}
```

### Build过程
```js
async function build (args, api, options) {
    //webpack配置调整
    const targetDir = api.resolve(options.outputDir)

    // resolve raw webpack config
    const webpackConfig = require('@vue/cli-service/lib/commands/build/resolveAppConfig')(api, args, options)

    // check for common config errors
    validateWebpackConfig(webpackConfig, api, options, args.target)

    return new Promise((resolve, reject) => {
        //调用webpack进行打包处理
        webpack(webpackConfigs, (err, stats) => {}
    }
}
```

## 总结
- Serve命令使用`webpackDevServer`开发模式
- Buinld命令使用`webpack`打包功能
