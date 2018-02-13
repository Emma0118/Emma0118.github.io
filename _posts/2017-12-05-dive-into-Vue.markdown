---
layout: post
title: 深入理解Vue响应式原理
date: 2017-12-05 19:33:00
---
### 数据响应原理
首先，Vue是基于 Object.defineProperty 来实现数据响应的，而 Object.defineProperty 是 ES5 中一个无法 shim 的特性，这也就是为什么 Vue 不支持 IE8 以及更低版本浏览器的原因；Vue通过Object.defineProperty的 getter/setter 对收集的依赖项进行监听,在属性被访问和修改时通知变化,进而更新视图数据。

在很早之前Javascript已经废弃了Object.observe，因此Vue不能检测到对象属性的添加或删除。由于 Vue 会在初始化实例时对属性执行 getter/setter 转化过程，所以属性必须在 data 对象上存在才能让Vue转换它，这也是必须在data中首先声明属性的原因。

Vue数据响应式变化主要涉及 Observer, Watcher , Dep 这三个主要的类；因此要弄清Vue响应式变化需要明白这个三个类之间是如何运作联系的；以及它们的原理，负责的逻辑操作。

### Vue初始化及实例

#### Init

```bash
    /*初始化生命周期*/
    initLifecycle(vm)
    /*初始化事件*/
    initEvents(vm)Object.defineProperty
    /*初始化render*/
    initRender(vm)
    /*调用beforeCreate钩子函数并且触发beforeCreate钩子事件*/
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    /*初始化props、methods、data、computed与watch*/
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    /*调用created钩子函数并且触发created钩子事件*/
    callHook(vm, 'created')
```
initState中初始化了 props,methods,data,computed和watch，因此在以后的代码中将进一步看它们各自的初始化过程

1、init data
```bash
    /*初始化data*/
    function initData (vm: Component) {
      /*得到data数据*/
      let data = vm.$options.data
      data = vm._data = typeof data === 'function'
        ? getData(data, vm)
        : data || {}defi
      ...
      //遍历data中的数据
      while (i--) {
        /*保证data中的key不与props中的key重复，props优先，如果有冲突会产生warning*/
        if (props && hasOwn(props, keys[i])) {
          process.env.NODE_ENV !== 'production' && warn(
            `The data property "${keys[i]}" is already declared as a prop. ` +
            `Use prop default value instead.`,
            vm
          )
        } else if (!isReserved(keys[i])) {
          /*判断是否是保留字段*/
          /*这里是我们前面讲过的代理，将data上面的属性代理到了vm实例上*/
          proxy(vm, `_data`, keys[i])
        }
      }
      // observe data
      /*这里通过observe实例化Observe对象，开始对数据进行绑定，asRootData用来根数据，用来计算实例化根数据的个数，下面会进行递归observe进行对深层对象的绑定。则asRootData为非true*/
      observe(data, true /* asRootData */)
    }
```
原理很简单，就是做了两件事：
**是将_data上面的数据代理到vm上
**二是通过执行 observe(data, true / asRootData /)将所有data变成可观察的，即对data定义的每个属性进行getter/setter操作，这里就是Vue实现响应式的基础。以后只要实现一个observe就可以了

2、Observer类
```bash
    export class Observer {
      value: any;
      dep: Dep;
      vmCount: number; // number of vms that has this object as root $data
      constructor (value: any) {
        this.value = value
        this.dep = new Dep()
        this.vmCount = 0
        /* 将Observer实例绑定到data的__ob__属性上面去，之前说过observe的时候会先检测是否已经有__ob__对象存放Observer实例了，def方法定义可以参考/src/core/util/lang.js*/
        def(value, '__ob__', this)
        if (Array.isArray(value)) {
          /*如果是数组，将修改后可以截获响应的数组方法替换掉该数组的原型中的原生方法，达到监听数组数据变化响应的效果。这里如果当前浏览器支持__proto__属性，则直接覆盖当前数组对象原型上的原生数组方法，如果不支持该属性，则直接覆盖数组对象的原型。*/
          const augment = hasProto
            ? protoAugment  /*直接覆盖原型的方法来修改目标对象*/
            : copyAugment   /*定义（覆盖）目标对象或数组的某一个方法*/
          augment(value, arrayMethods, arrayKeys)
          /*如果是数组则需要遍历数组的每一个成员进行observe*/
          this.observeArray(value)
        } else {
          /*如果是对象则直接walk进行绑定*/
          this.walk(value)
        },
        walk (obj: Object) {
          const keys = Object.keys(obj)
          /*walk方法会遍历对象的每一个属性进行defineReactive绑定*/
          for (let i = 0; i < keys.length; i++) {
            defineReactive(obj, keys[i], obj[keys[i]])
          }
        }
      }
```

做三件事
**首先将Observer实例绑定到data的ob属性上面去，防止重复绑定；
**若data为数组，先实现对应的变异方法再将数组的每个成员进行observe，使之成响应式数据；
**否则执行walk()方法，遍历data所有的数据，进行getter/setter绑定，这里的核心方法就是 defineReative(obj, keys[i], obj[keys[i]])


3、Watcher
是一个观察者对象，实现原理就是观察者模式。数据变动的时候Dep会通知Watcher实例，然后由Watcher实例回调cb进行视图的更新。

```bash

    export default class Watcher {
      constructor (
        vm: Component,
        expOrFn: string | Function,
        cb: Function,
        options?: Object
      ) {
        this.vm = vm
        /*_watchers存放订阅者实例*/
        vm._watchers.push(this)
        // options
        if (options) {
          this.deep = !!options.deep
          this.user = !!options.user
          this.lazy = !!options.lazy
          this.sync = !!options.sync
        } else {
          this.deep = this.user = this.lazy = this.sync = false
        }
        this.cb = cb
        this.id = ++uid // uid for batching
        this.active = true
        this.dirty = this.lazy // for lazy watchers
        this.deps = []
        this.newDeps = []
        this.depIds = new Set()
        this.newDepIds = new Set()
        this.expression = process.env.NODE_ENV !== 'production'
          ? expOrFn.toString()
          : ''
        // parse expression for getter
        /*把表达式expOrFn解析成getter*/
        if (typeof expOrFn === 'function') {
          this.getter = expOrFn
        } else {
          this.getter = parsePath(expOrFn)
          if (!this.getter) {
            this.getter = function () {}
            process.env.NODE_ENV !== 'production' && warn(
              `Failed watching path: "${expOrFn}" ` +
              'Watcher only accepts simple dot-delimited paths. ' +
              'For full control, use a function instead.',
              vm
            )
          }
        }
        this.value = this.lazy
          ? undefined
          : this.get()
      }
      /**
       * Evaluate the getter, and re-collect dependencies.
       */
       /*获得getter的值并且重新进行依赖收集*/
      get () {
        /*将自身watcher观察者实例设置给Dep.target，用以依赖收集。*/
        pushTarget(this)
        let value
        const vm = this.vm
        /*执行了getter操作，看似执行了渲染操作，其实是执行了依赖收集。
          在将Dep.target设置为自生观察者实例以后，执行getter操作。
          譬如说现在的的data中可能有a、b、c三个数据，getter渲染需要依赖a跟c，
          那么在执行getter的时候就会触发a跟c两个数据的getter函数，
          在getter函数中即可判断Dep.target是否存在然后完成依赖收集，
          将该观察者对象放入闭包中的Dep的subs中去。*/
        if (this.user) {
          try {
            value = this.getter.call(vm, vm)
          } catch (e) {
            handleError(e, vm, `getter for watcher "${this.expression}"`)
          }
        } else {
          value = this.getter.call(vm, vm)
        }
        // "touch" every property so they are all tracked as
        // dependencies for deep watching
        /*如果存在deep，则触发每个深层对象的依赖，追踪其变化*/
        if (this.deep) {
          /*递归每一个对象或者数组，触发它们的getter，使得对象或数组的每一个成员都被依赖收集，形成一个“深（deep）”依赖关系*/
          traverse(value)
        }
        /*将观察者实例从target栈中取出并设置给Dep.target*/
        popTarget()
        this.cleanupDeps()
        return value
      }
      /**
       * Add a dependency to this directive.
       */
       /*添加一个依赖关系到Deps集合中*/
      addDep (dep: Dep) {
        const id = dep.id
        if (!this.newDepIds.has(id)) {
          this.newDepIds.add(id)
          this.newDeps.push(dep)
          if (!this.depIds.has(id)) {
            dep.addSub(this)
          }
        }
      }
      /**
       * Clean up for dependency collection.
       */
       /*清理依赖收集*/
      cleanupDeps () {
        /*移除所有观察者对象*/
        ...
      }
      /**
       * Subscriber interface.
       * Will be called when a dependency changes.
       */
       /*
          调度者接口，当依赖发生改变的时候进行回调。
       */
      update () {
        /* istanbul ignore else */
        if (this.lazy) {
          this.dirty = true
        } else if (this.sync) {
          /*同步则执行run直接渲染视图*/
          this.run()
        } else {
          /*异步推送到观察者队列中，下一个tick时调用。*/
          queueWatcher(this)
        }
      }
      /**
       * Scheduler job interface.
       * Will be called by the scheduler.
       */
       /*
          调度者工作接口，将被调度者回调。
        */
      run () {
        if (this.active) {
          /* get操作在获取value本身也会执行getter从而调用update更新视图 */
          const value = this.get()
          if (
            value !== this.value ||
            // Deep watchers and watchers on Object/Arrays should fire even
            // when the value is the same, because the value may
            // have mutated.
            /*
                即便值相同，拥有Deep属性的观察者以及在对象／数组上的观察者应该被触发更新，因为它们的值可能发生改变。
            */
            isObject(value) ||
            this.deep
          ) {
            // set new value
            const oldValue = this.value
            /*设置新的值*/
            this.value = value
            /*触发回调*/
            if (this.user) {
              try {
                this.cb.call(this.vm, value, oldValue)
              } catch (e) {
                handleError(e, this.vm, `callback for watcher "${this.expression}"`)
              }
            } else {
              this.cb.call(this.vm, value, oldValue)
            }
          }
        }
      }
      /**
       * Evaluate the value of the watcher.
       * This only gets called for lazy watchers.
       */
       /*获取观察者的值*/
      evaluate () {
        this.value = this.get()
        this.dirty = false
      }
      /**
       * Depend on all deps collected by this watcher.
       */
       /*收集该watcher的所有deps依赖*/
      depend () {
        let i = this.deps.length
        while (i--) {
          this.deps[i].depend()
        }
      }
      /**
       * Remove self from all dependencies' subscriber list.
       */
       /*将自身从所有依赖收集订阅列表删除*/
      teardown () {
       ...
      }
    }
```

4、Dep
被Observer的data在触发 getter 时，Dep 就会收集依赖的 Watcher ，通过 Dep 给 Watcher 发通知进行更新。核心是通过一个订阅者设计模式实现这种通知功能
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;
  constructor () {
    this.id = uid++
    this.subs = []
  }
  /*添加一个观察者对象*/
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
  /*移除一个观察者对象*/
  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }
  /*依赖收集，当存在Dep.target的时候添加观察者对象*/
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
  /*通知所有订阅者*/
  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

## 总结
1.在 Vue 中模板编译过程中的指令或者数据绑定都会实例化一个 Watcher 实例，实例化过程中会触发 get() 将自身指向 Dep.target;
2.data在 Observer 时执行 getter 会触发 dep.depend() 进行依赖收集;依赖收集的结果：1、data在 Observer 时闭包的dep实例的subs添加观察它的 Watcher 实例；2. Watcher 的deps中添加观察对象 Observer 时的闭包dep；
3.当data中被 Observer 的某个对象值变化后，触发subs中观察它的watcher执行 update() 方法，最后实际上是调用watcher的回调函数cb，进而更新视图。

最后以图明志
![](/assets/images/2.png)
