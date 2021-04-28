# invoke设计开发文档

首先， 会判断当前应用有没有启动。

启动方法很简单， 就是调用performAppChange已经注册进fino的应用。

```javascript
public start() {
   setStore('root', 'root')
   this.performAppChnage(this.appList)
}
```

### 1. performanceAppChange

接下来看一下performanceAppChange中都发生了什么(伪代码)

可以看到， 核心就是获取到需要挂载的app, 卸载需要卸载的app。

然后创建当前app的运行环境

之后就是触发应用的生命周期函数。

```javascript
async performAppChnage(apps: Array<Project>) {
    // Get the application that needs to be mounted
    const activeApp = getAppShouldBeActive(apps);
        
    // uninstall apps that do not require activation
    const unmountApps = getAppShouldBeUnmount(apps);

    if (unmountApps) {
        await unmount(unmountApps, this.sandbox, this.mode)
    }

    if (this.app.status === MOUNT) {
        globalContext.activedApplication = this.app
    }

    globalContext.activeContext = await this.setRuntimeContext(this.app);

    this.app = await lifecircleFn[(lifecircle as LifeCircle)](this.app, this.sandbox, this.getModuleJs.bind(this))

    // At this time, the various information of the app is ready, merge the cached information to the current app
    this.app = Object.assign(this.app, globalContext.activeAppInfo, globalContext.activedApplication)
}
```

来看一下app的几个生命周期：



### 2. life circle - bootstrap

bootstrap很简单， 

- 状态变为bootstraping

- 就是加载应用资源。 

- 开启应用沙箱。
- 触发mount生命周期函数

```javascript
export async function bootstrap(app: Project, sandbox: SnapshotSandbox, loadFn: (baseDomain: string, assetsData: any) => Promise<Project>) {
    app.status = "bootstraping"
    let activeProject: Project['appInfo'];

    const entryStatePath = app.entry.indexOf('http') > -1 ? app.entry : app.domain + app.entry;

    // Get application js packaging information
    activeProject = await getEntryJs(entryStatePath);

    // activate the sandbox
    if (!sandbox.appCache.includes(app.name)) {
        sandbox.name = app.name
        sandbox.active()
    }

    // load application entry file { init: () => {}, name: string, destory: () => {} }
    globalContext.activedApplication = await loadFn(app.domain, globalContext.activeAppInfo);

    removeChild(app.domain + app.entry);

    // Execute mount life cycle
    return mount(app, sandbox)
}

```

### 2. life circle - mount

- 开启沙箱（应用可能一开始就是mount状态）
- 执行init函数

```javascript
export async function mount(app: Project, sandbox: SnapshotSandbox,) {
    const runtime = globalContext.activeContext

    // The runtime context is initialized successfully, and the route is injected
    if (app.status === MOUNT) {
        globalContext.activedApplication = app
        sandbox.name = app.name
        sandbox.active()
    }

    // app.dynamicElements.css && (await loadCss(app.dynamicElements.css))
    if (app.name === globalContext.activedApplication.name) {
        // injection router
        globalContext.activedApplication.init && globalContext.activedApplication.init((runtime as VueRuntimeContext), globalContext.activeContext)
    }

    app.status = MOUNT

    invoke.$event.notify('appMount')

    return app
}
```

### 3. life circle - unmount

- 卸载应用
- 关闭沙箱
- 还原全局状态函数

```javascript
export async function unmount(apps: MatchAppType[], sandbox: SnapshotSandbox, mode: string) {
    return new Promise<void>((resolve) => {
        if (!apps) { resolve() }

        for (let i = 0; i < apps.length; i++) {
            if (!apps[i].app.dynamicElements.js) continue
            // 卸载应用标签
            for (let j = 0; j < apps[i].app.dynamicElements.js.length; j++) {
                removeChild(apps[i].app.dynamicElements.js[j])
            }

            if (mode === 'safe') {
                apps[i].app.dynamicElements.js = []
                // 将状态设置位UNMOUNT
                apps[i].app.status = UNMOUNT;
            }
        }
        sandbox.inactive()
        resolve()
    })
}
```



### 5. 创建运行环境

```javascript
/**
 * @function { Initialize the runtime context }
 * @param { context - runtime context type: like vue }
 * @param { version - runtime context version: string }
 *
 */
async function initRuntimeContext(context: string, version: string): Promise<ContextType['context']> {
    const runtimePool: any = {
        vue: VueRuntimeContext,
        phaser: PhaserRuntimeContext,
        // vue3: Vue3RuntimeContext
    }

    const runtime: ContextType['context'] = new runtimePool[context]();

    await runtime.getContextResource(version);

    runtime.instance = runtime.createContext(version)

    return runtime
}

export default initRuntimeContext
```

#### 