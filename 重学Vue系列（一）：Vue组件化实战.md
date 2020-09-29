# 重学Vue系列（一）：Vue组件化实战

## 组件化

组件化是vue的核心思想，它能提高开发效率，方便**重复**使用,简化调试步骤，**提升整个项目的可维护性**，便于多人协同开发

### 组件通信

#### 父组件 => 子组件：

- 属性props

  ```vue
  // child
  props: {msg: String}

  // parent
  <Helloworld msg="welcome to Your Vue.js App"
  ```

- 特性$attrs

  ```vue
  // child
  <p>{{$attrs.foo}}</p>

  // parent
  <Helloworld foo="foo" />
  ```

- 引用refs

  ```vue
  // parent
  <Helloworld ref="hw"/>

  mounted() {
  	this.$refs.hw.xx = 'xxx'
  }
  ```

- 子元素$children

  ```vue
  // parent
  this.$children[0].xx='xxx'
  ```

  > 子元素不保证顺序

#### 子组件=>父组件：自定义事件

```vue
// child
this.$emit('add',good)

// parent
<Cart @add="cartAdd($event)"></Cart>
```

#### 兄弟组件：通过共同祖辈组件

通过共同的祖辈组件搭桥,  $parent或$root.

```javascript
// brother1
this.$parent.$on('foo',handle)
// brother2
this.$parent.$emit('foo')
```

#### 祖先和后代之间

由于嵌套层数过多，传递props不切实际，vue提供了```provide/inject```API完成该任务

- provide/inject: 能够实现祖先给后代传值

  ```javascript
  // ancestor
  provide(){
  	return {foo: 'foo'}
  }
  // descendant
  inject: ['foo']
  ```

- elementui写法：通过广播的形式，不仅能够实现祖先通知后代，也能实现后代通知祖先

  ```javascript
  // emitter.js
  function broadcast(componentName, eventName, params) {
    this.$children.forEach(child => {
      var name = child.$options.componentName;

      if (name === componentName) {
        child.$emit.apply(child, [eventName].concat(params));
      } else {
        broadcast.apply(child, [componentName, eventName].concat([params]));
      }
    });
  }
  export default {
    methods: {
      dispatch(componentName, eventName, params) {
        var parent = this.$parent || this.$root;
        var name = parent.$options.componentName;

        while (parent && (!name || name !== componentName)) {
          parent = parent.$parent;

          if (parent) {
            name = parent.$options.componentName;
          }
        }
        if (parent) {
          parent.$emit.apply(parent, [eventName].concat(params));
        }
      },
      broadcast(componentName, eventName, params) {
        broadcast.call(this, componentName, eventName, params);
      }
    }
  };
  ```



#### 任意两个组件之间：事件总线 或 vuex

- 事件总线：创建一个Bus类负责事件派发、监听和回调管理

  ```javascript
  // Bus: 事件派发、监听和回调
  class Bus{
      constructor(callbacks={}){
          this.callbacks=callbacks
      }
      $on(event,callback){
          this.callbacks[event]=this.callbacks[event] || []
          this.callbacks[event].push(callback)
      }
      $emit(event,args){
          if(!this.callbacks[event]){
              return
          }
          this.callbacks[event].forEach(cb =>cb(args))
      }
  }
  // main.js
  Vue.prototype.$bus = new Bus()

  // child1
  this.$bus.$on('foo',handle)
  // child2
  this.$bus.$emit('foo')

  ```

  > 实践中可以用Vue代替Bus，因为它已经实现了相应功能

  ```javascript
  // bus.js
  import Vue from 'vue'
  export default new Vue()

  // comp1
  import bus from 'bus.js'
  Bus.$on('foo',handle)
  // comp2
  import bus from 'bus.js'
  Bus.$emit('foo')

  ```



- [vuex](https://vuex.vuejs.org/zh/): 创建唯一的全局数据管理者store,通过它管理数据并通知组件状态变更

  ![vuex](.\images\vuex.png)
