# 如何调试
想要开始调试， 需要先确保你的本地有了<a href="https://github.com/finoer/finoer/tree/dev">finoer</a>项目。

并且<span style="color: #4ba8c9">初始化好了gitsubmodule</span>。

之后可以通过fino-cli 克隆下来两个测试用的子模块

1. 安装fino-cli

```shell
sudo npm install @finoer/fino-cli -g
```

2. 通过cli克隆子项目

```powershell
# 进入到目标目录
cd /Users/wangdongxu/Desktop/my-project
# 创建模块项目
fino <projectName>
# 进入到模块项目
cd <projectName>

```

现在有了子模块项目， 第一步，打开`vf2e.config.json`

修改name和项目名称，项目entry等信息

```json

{
    "name": "scene-list-001",
    "domain": "http://localhost:8081",  
    "entry": "/scene-list-001/stats.js",
    "context": "vue",
    "version": "2.6.2"
}

```

3. 将子模块的信息添加到基座中

   打开finoer项目下面`templates`下的`fino-base`

   将项目信息添加到项目下面的project.js中

   ```js
   const projectList = [
       {
           "name": "vue3-test",
           activeWhen: function (location: Location) {
               const pathName = window.location.pathname;
               return new RegExp('phaser1', 'g').test(pathName)
           },
           "domain": "http://localhost:8080",
           "entry": "/vue3-test/stats.js",
           "context": "vue3",
           "version": "3.0.7"
       },
   ]
   
   export default projectList
   
   ```

4. 启动基座

   ```shell
   npm run template fino-base
   ```

   


