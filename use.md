## Fino项目架构

> 目标导航

1. Fino架构简介
2. 目录简介
3. fino架构优势
4. fino架构详解
5. 开发方式



#### 1.  fino架构简介

fino是一个专注于一键式微前端框架的解决方法。

整个项目采用了基于reference和gitmodule的monorepo的架构， 即单一仓库管理多个模块。

同时解决了monorepo仓库代码体积过大和开发体验不好的问题.



#### 2. 目录简介

```js
├─templates // 微前端项目模版

|     ├─fino-child-vue // vue子项目模版

|     ├─fino-child // 子项目模版

|     ├─fino-base // 基座模版

├─src

|  └index.ts // 脚手架工具

├─scripts  // 启动命令

|    ├─build.js

|    ├─dev.js

|    ├─docs.js

|    └utils.js

├─packages  // 项目依赖npm包

|    ├─finoer-core // 微前端框架主npm包

|    ├─finoer-component-vue // 子项目npm包
```



#### 架构优势

> 开发模版

模版项目基本包含了我们微前端框架的所有功能（包）

在日常开发场景中， 不可避免的需要同时开发**多个依赖包**和**模版功能**的场景。

基于这种场景， 如果是传统方式开发。 需要以下4步

![](./static/images/fino1.png)

但是fino架构可以做到, 只要2步

<img src="./static/images/fino2.png" alt="2" style="width:600px;float:left" />


>开发npm包

npm包就是我们微前端fino框架中的需要发布到npm上的package包

fino架构简化了开发流程， 只需要在根目录下即可实现一键开发， 一键打包，一键发布等功能。

#### 开发方式

> 开发模版

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

  