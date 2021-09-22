# store-api

## 1. 初始化store

```js
import DataStore from '@finoer/finoer-store'

const store = new DataStore()

invoke.$event.subscribe('beforeAppEnter', () => {
    globalInstance.$data && globalInstance.$data.clear()

    const element = document.getElementById('fino-vue-root');

    element && element.setAttribute('class', `${$data.namespace} ${invoke.app.name}`)

    globalInstance.$data && globalInstance.$data.init(invoke.app.name)
})
```

## 2. 存储数据

> set

```js
$data.set(data, namespace?: string)
```

