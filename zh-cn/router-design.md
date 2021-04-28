### 提供路由跳转方法

**路由跳转**

```tsx
// url为web应用的路由
push(url: string) {
   window.history.pushState(null, '', url)
}
```

**重启应用**

当前存在运行应用情况下，重启已注册的APP。

```tsx
reroute() {
   this.invoke.app && this.invoke.performAppChnage(Apps)
}
```

---

如何使用：

```tsx
import { router } from "@finoer/finoer-invoke";

router.push(`应用的名称、app对应的路由`);

router.reroute();
```

---

router做了什么？

重写history的pushState()、replaceState()方法

1.如果在运行的应用上发生了history事件，则调用内部的reroute()方法；

2.触发浏览器的前进后退事件触发reroute();

```tsx
hijackHistory() {
        const me = this
        window.history.pushState = function (state: any, title: string, url: string | null | undefined, ...rest) {
            let result = originalPushState.apply(this, [state, title, url]);
            me.reroute()
            return result
        }

        window.history.replaceState = function (state: any, title: string, url: string | null | undefined, ...rest) {
            let result = originalReplaceState.apply(this, [state, title, url])
            me.reroute();
            return result;
        }

        window.onpopstate = (event: PopStateEvent) => {
            me.reroute();
        }

        window.addEventListener('popstate', () => {
            me.reroute();
        }, false);
    }
```

集中管理事件监听,且保持监听事件的唯一性：

1.创建词典，存放事件目录;

```tsx
export const routingEventsListeningTo = [
  'popstate', 'hashChange'
]

const captureEventListeners: any = {
  'popstate': [],
  'hashChange': [],
}
```

    2.构造一个方法用来捕获词典中的事件，有则返回true，无则返回false；

```tsx
export function find(list: Array<any>, element: any) {
  if (!Array.isArray(list)) {
    return false;
  }

  return list.filter(item => item === element).length > 0;
}
```

 3.事件的添加，如果触发的事件是一个`function`&&`不在事件词典目录中`,则将时间存放入词典；

```tsx
// EventListenerOrEventListenerObject
hijackEventListener() {
        window.addEventListener = (eventName: string, fn: EventListenerOrEventListenerObject, purpol?: boolean | AddEventListenerOptions) => {
            if (
                typeof fn === 'function' &&
                routingEventsListeningTo.indexOf(eventName) > -1 &&
                !isInCapturedEventListeners(eventName, fn)
            ) {
                addCapturedEventListeners(eventName, fn);
                return;
            }

            return originalAddEventListener.apply(window, [eventName, fn, purpol]);

        }

       
    }
```

  4.事件的移除，首先判断`funciton`&&`查询词典`，存在即移除；

```tsx
hijackEventListener() { 

  window.removeEventListener = (eventName: string, fn: EventListenerOrEventListenerObject, purpol?: boolean | AddEventListenerOptions) => {
        if (typeof fn === 'function' && routingEventsListeningTo.indexOf(eventName) >= 0) {
           removeCapturedEventListeners(eventName, fn);
           return;
        }

       return originalRemoveEventLister.apply(window, [eventName, fn, purpol]);
  };
}
```