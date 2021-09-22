# 如何接入

## 基座应用

### 1. 安装fino

安装调度包 `finoer-invoke`

这个包的功能是负责调度注册到fino框架里面的应用的。

```shell
npm install @finoer/finoer-invoke --save
```

这个相当于一个入口包。

这个包里面包含了**注册，路由， 沙箱， 调度，数据通信**等功能。

### 2. 基座应用

```js
import { invoke, registerApps } from '@finoer/finoer-invoke'

// 声明所有子模块的列表
const projectList = [
  {
    {
      // 子模块的名字
      name: 'plantform_list', 
      // 子模块激活时机
      activeWhen: function (location: Location) {
        const route = window.location.href.split[1]
        return route === 'plantform_list'
      }, 
      // 子模块所在域名
      domain: "http://localhost:8080", 
      // 子模块入口文件(固定)  
      entry: '/plantform_list/stats.js', 
      // 子模块运行环境
      context: "vue", 
      // 子模块运行环境的入口
      version: "2.6.2" 
  	},
  }
]

// 注册应用
registerApps(projectList)

// 两种模式， 一种是speed, 一种是safe模式
invoke.mode = "safe"

```



## 子应用

### 1. 配置打包工具插件

安装插件

```shell
npm i webpack-stats-plugin -D
```

工程配置中增加如下代码

```js
module.exports = {
  ...,
  plugins: [
  	new StatsWriterPlugin({
        filename: `${moduleInfo.name}/stats.js`, // Default
        transform(data, opts) {
          const allAssets = Object.keys(opts.compiler.assets)

          let i = allAssets.length - 1;

          while (i > 0) {
            const item = allAssets[i]
            if (item.indexOf('.map') > -1 || item.indexOf('.html') > -1) {
              allAssets.splice(i, 1)
            }
            i--
          }

          const content = `$event.notify('baseInfoLoaded', ${JSON.stringify(allAssets)})`
          return content
        }
      }),
  ]
}
```

### 2. 增加子应用配置文件

新增一个vf2e.config.json

```json
{
    "name": "yueqi",
    "domain": "http://localhost:8080", 
    "entry": "/yueqi/stats.js",
    "context": "vue",
    "version": "2.6.2"
}
```

入口文件进行兼容, 判断是独立运行还是在基座用运行

```js
import { invoke, registerApps, $data } from '@finoer/finoer-invoke'
import { routerArray, } from './router'
// @ts-ignore
import mouduleInfo from '../vf2e.config.json'

const runtime = invoke.isInFinoRuntime()
// 如果是独立运行， 那么作为基座注册自己
if(!runtime) {
  registerApps([ {
    ...mouduleInfo,
    activeWhen: (location) => {
        const route = window.location.href.split[1]
        return route === mouduleInfo.name
    }
  } ])
}

const finoApp = {
    name: mouduleInfo.name,
    init: (instance: any) => {
        // @ts-ignore
        instance.injectionRouter(routerArray)
    }
};

invoke.$event.notify('childAppLoaded', finoApp);
```

