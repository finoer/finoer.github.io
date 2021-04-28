# 如何开发

### 1. 项目目录

```makefile
├─templates // 微前端项目模版
|     ├─fino-child-vue // vue子项目模版
|     ├─fino-child // 子项目模版
|     ├─fino // 基座模版
├─src
|  └index.ts
├─scripts
|    ├─build.js
|    ├─dev.js
|    ├─docs.js
|    └utils.js
├─packages  // 项目依赖npm包
|    ├─finoer-core // 微前端框架主npm包
|    ├─finoer-component-vue // 子项目npm包
```



### 2. 联合开发

联合开发就是packages和templates一起开发，

或者说框架开发者进行开发

```shell
git clone https://github.com/finoer/finoer.git
```

初始化所有git submodule

```powershell
git submodule init && git submodule update --remote
```

初始化

```powershell
npm run init
```

启动模版

```powershell
npm run template <模版名>
```

运行某一个package包

```powershell
npm run dev <包名>
```



### 2. 独立开发

启动包

```powershell
npm run dev <包名>
```

编译包

```powershell
npm run build <包名>
```

生成文档

```powershell
npm run doc finore-core
```



## 如何进行开发

> 安装依赖

```powershell
npm run init 
```

node /Users/wangdongxu/Desktop/butterfly/core/main.js
> 开发已经存在模版

- 在根目录下的tsconfing.json中的reference字段中确定需要监控的依赖包

  ```json
  // tsconfing.json
  {
    "references": [
        { "path": "./packages/finoer-core" }
     ]
  }
  ```

- 运行启动命令

  ```shell
  # 开发base模版
  npm run template:base
  # 其他模版项目, 在package.json中添加命令
  npm run template:other
  ```

> 开发package包

- 直接npm run dev <package名称>

  ```shell
  # 启动
  npm run dev finoer-core
  ```

- 打包 npm run build <package名称>

  ```shell
  # 打包
  npm run build finoer-core
  ```

- 生成文档npm run docs <package名称>

  ```shell
  # 打包
  npm run docs finoer-core
  ```

