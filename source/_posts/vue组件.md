# 通信方式

## 父=>子 props

### 特性

- **命名**

  由于HTML中属性大小写不敏感 - prop名字写成camelCase`驼峰命名法`等价于kebab-case`短横线分隔`

- **传参**

  传递不是字符串需要使用`v-bind`来告诉浏览器这是`javascript`表达式而不是字符串

  注：对象和数组等引用类型是传入引用的，子组件进行修改也会影响父组件的值

- **单向数据流**

  所有的prop都使得父子prop之间形成**单向下行绑定**`夫级的prop更新会向下流动到子组件中，反过来不行`

  注：一般不在子组件中修改prop的值，应在父组件进行修改，流向子组件

### 使用

子组件定义props属性

``` javascript
// type: Boolean
//       Number
//       String
//		 Symbol

//       Array
//       Object
//       Date
//       Function
//       自定义对象类型
// 输入先检查类型是否正确, 再执行合法性校验
export default {
	props: {
		name: {
            type: String,
            default: 'xxx',
            validator: function (value) {...}, // 输入合法性校验
        }
	},
}
```

### 意外使用

#### 传入非prop的属性

这些属性直接加入到子组件的根元素上

##### 不希望继承

配置下面信息子组件的根元素就不会继承父组件传入的未知prop属性

``` js
Vue.component('my-component', {
    inheritAttrs: false, // 注：不会影响class/style的合并操作
    // ...
})
```

使用`$attrs`可以把未知prop属性加入到某元素中(`常用于基础组件的封装`)

``` vue
<!-- base-input.vue中定义如下 -->
<label>
	{{ label }}
    <input v-bind="$attrs"/>
</label>
<!-- 父组件使用时 -->
<base-input v-model="username" required placeholder="Enter"></base-input>
```

从上面代码上看，父组件的未知属性直接插入到子组件中的`$attrs`

#### 替换/合并已有属性

如父组件传入`class/style`，子组件会合并里面的内容，并且父组件在后边`父组件的css属性会替换子组件的属性`

## 子=>父 事件监听

### v-on/@

``` vue
<!-- my-button.vue 子组件 -->
<button v-on:click="$emit('enlarge-text')"> // $emit(FuncName, ...agrs)
    ClickButton
</button>
<!-- 父组件 -->
<my-button @enlarge-text="console.log('点击事件')"></my-button>
```

### v-model

``` vue
<input v-model="text" />
<!-- 等价于 -->
<input v-bind:value="text" v-on:input="text = $event.target.value" />
```

子组件实现可让父组件使用v-model

```vue
<!-- props 定义v-model的属性 如value -->
<input :value="value" @input="$emit('input', $event.target.value)" />
```

### .sync

>  注：2.3.0新增

是在v-on的基础上添加, 简化父组件书写

```vue
<my-input v-bind:title.sync="title"></my-input> <!-- 注意这里的title只能是字面量,不可以是js表达式 -->
<!-- 等价于(自动生成下面函数) -->
<my-input :title="title" v-on:update:title="title"></my-input>

<!-- 子组件进行调用函数即可更新 -->
this.$emit('update:title', newTitle);
```

### 注意事项

- 命名

  区分大小写，全部被转换成小写。推荐始终使用kebab-case命名

# 插槽

可以让父组件传递`html`渲染到子组件中的槽位

> 2.6.0 引入 v-slot 取代 slot和slot-scope

## 使用

``` vue
<!-- 父组件 -->
<navigation>
	<!-- 插入的html、文本等 -->
    Your HTML
</navigation>
<!-- 子组件 -->
<a>
	<slot>xxx</slot>  <!-- 父组件所插入的位置, 当未插入默认使用xxx -->
</a>
```

## 数据传递

> **父级模板里面的所有内容都是在父级作用域中编译的**
>
> **子模板里的所有内容都是在子作用域中编译的**

插槽标签在父级，只能访问到父级的数据

### 插槽访问子组件数据

``` vue
<!-- my-slot.vue 子组件 -->
<slot v-bind:user="user"></slot>
<!-- 父组件 -->
<my-slot v-slot:default="props">{{props.user.xxx}}</my-slot> <!-- default是插槽名，default默认插槽; props是变量名，任意名字都可 -->
<!-- default可去掉 -->
<my-slot v-slot="props">{{props.user.xxx}}</my-slot>
```

## 具有名字的插槽

> 2.6.0 引入

当一个子组件有多个槽位，父组件插入的位置需要用`v-slot`指定插槽名

``` vue
<!-- base-layout.vue子组件 -->
<div>
    <slot name="header"></slot>
    <slot></slot>  <!-- 这个槽位接收剩余信息 -->
    <slot name="footer"></slot>
</div>
<!-- 父组件 -->
<base-layout>
	<template v-slot:header>
    	<h1>
            插入到头部槽位
        </h1>
    </template>
    <!-- 缩写 -->
    <tempalte #header></tempalte>
    <!-- 动态插槽名 -->
    <template v-slot:[slotName]></template>
    
    <template v-slot:footer>
    	插入到尾部槽位
    </template>
</base-layout>
```

## 原理

将父组件的插槽内容包裹在一个单参数的函数里

``` js
function(props) {
    // 插槽内容...
}
```

# 动态组件&异步组件

## 动态组件

使用内置组件实现动态组件

```vue
<!-- 可以是一个组件的名字、也可以是一个组件的选项对象 -->
<component :is="name"></component>
```

### 组件缓存

``` vue
<keep-alive>
	<component v-bind:is="name"></component>
</keep-alive>
```

## 异步组件

利用工厂函数来定义组件`单例懒汉式加载`

``` js
Vue.component('async-component', function(resolve, reject) {
    // setTimeout只是一个演示
    setTimeout(function() {
        resolve(() => import('xxx.vue'))
    }, 1000)
})
```

### 推荐

异步组件和`webpack的code-splitting`功能配合使用

``` js
// webpack
Vue.component('async-component', function(resolve){
    // require函数
    //		第一个传入数组，会异步加载数组里面的组件
    //		全部加载完成后回调resolve
    require(['xxx.vue'], resolve)
})

// webpack和ES2015
Vue.component('async-component', ()=>import('xxx.vue'))
```

### 处理加载状态

```js
Vue.component('async-component', ()=>({
    // 需要加载的组件-Promise对象
    component: import('xxx.vue'),
    // 异步组件加载时使用的组件
    loading: LoadingComponent,
    // 加载失败时使用的组件
    error: ErrorComponent,
    // 展示加载时组件的延时时间，默认200ms(200ms后进行加载组件)
    delay: 200,
    // 加载超时，会时候error组件，默认Infinity
    t
}))
```

