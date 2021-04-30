# invoke api



### 1. 注册应用

> registerApp()

- 参数：Array < moduleInfo  { object }> 

-  用法

  ```js
    registerApps([ {
      name: "scene-list-001",
      domain: "http://localhost:8081",  
      entry: "/scene-list-001/stats.js",
      context: "vue",
      version: "2.6.2",
      activeWhen: (location) => {
        const pathName = window.location.pathname;
        return new RegExp('scene-list-001', 'g').test(pathName)
      }
     
    }])
  
  ```





