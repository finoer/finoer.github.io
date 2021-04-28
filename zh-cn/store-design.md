# store 设计文档

![](https://tva1.sinaimg.cn/large/008i3skNly1gpzjmhl6qxj31z40tmafn.jpg)



### 1. 初始化模块命名空间

- 初始化两个命名空间， 一个是global， 另一个是模块的命名空间

```javascript
 /**
   * Initialize the namespace of the currently activated project
   * @param { string } - name namespace
   */
  init(spacename: string) {
    if(!spacename) { throw new Error('请传入命名空间'); return }

    // 重置上一个命名空间的状态
    this.namespace = spacename

    // 从缓存中恢复数据
    if(!this.data[spacename]) {
      const dataString = window.localStorage.getItem(`finoData_${spacename}`)

      const data = dataString && JSON.parse(dataString);

      this.set(data, spacename)
    }

    if(!this.data.global) {
      const dataString = window.localStorage.getItem(`finoData_global`)

      const data = dataString && JSON.parse(dataString);

      this.set({ ...data }, 'global')
    }

    this.set(this.data[this.namespace], this.namespace)
  }

```



### 2. 设置数据

- 向当前命名空间下设置数据

```javascript
 /**
   * set up reactive data
   * @param { object } - data 用户需要设置的数据
   * @param { string? } - namespace 命名空间
   */
  set(data: DataType, namespace?: string) {
    const currentSpace: string = namespace || this.namespace;

    let initialData = this.data[currentSpace] || {};

    // 设置数据
    this.data[currentSpace] = Object.assign(initialData, data, {
      _spacename: currentSpace
    })

    // 代理访问链
    for(let key in data) {
      this.proxyOfprototype(data, currentSpace, key)
    }

    // 代理内容
    this.data[currentSpace] = this.proxyOfcontent(currentSpace);

    this.synchronizeDataInCache(currentSpace)
  }
```

代理分为两种。 一种是代理访问链， 一种是安全代理。

先看安全代理。


安全代理是为了避免其他模块修改数据。是基于proxy做的。

```javascript
 setProxy(data) {
   let proxy = new Proxy(data, {
     get: (target, key, receiver) => {
       const res = Reflect.get(target, key, receiver);
       return res;
     },
     set: (target, key, value, receiver) => {
       const oldValue = target[key];
       if (target._spacename !== this.namespace && target._spacename !== 'global') {
         console.error('不允许修改其他数据仓库的数据');
         return oldValue;
       }
       const result = Reflect.set(target, key, value, receiver);
       return result;
     }
   });
   return proxy;
 }
```

安全代理完成了，但是还不是那么完美。 

因为数据仓库的数据是这样的

```javascript
declare class Database {
    namespace: string;
    data: {
      global: {},
      moduleA: {},
      moduleB: {}
    };
    existingProxy: Array<string>;
}
```

所以如果你想访问某一个命名空间下面的数据， 访问链是这样的

```js
$data.data[spaceName].A
```

所以想缩短一下访问链表， 代理过后， 访问链就变成了这样。

当然， 这里只会代理global命名空间下面的和当前被激活的模块的代理链

```javascript
$data.A
```

```javascript
/**
   * Proxy access chain
   * @param { object } - data 用户需要设置的数据
   * @param { string } - namespace 命名空间
   * @param { string } - key 数据的key
   */
proxyOfprototype(data: DataType, namespace: string, key: string) {
    if(key === '_spacename') { return }

    const _namespace = namespace || this.namespace;

    const sharePropertyDefinition = {

      get: () => {
        return this.data[_namespace][key]
      },

      set: (newvalue: unknown) => {
        this.data[key] = newvalue
        this.data[_namespace][key] = newvalue
      },

      enumerable: true,
      configurable: true,
    }

    Object.defineProperty(this, key, sharePropertyDefinition)
}
```