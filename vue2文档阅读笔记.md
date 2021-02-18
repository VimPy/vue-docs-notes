1. 只有当实例被创建时就已经存在于data中的property才是响应式的。如果添加一个新的property，比如vm.b = 'test'，那么b的改动将不会触发任何视图的更新。唯一的例外是使用Object.freeze()，会阻止修改现有的property，意味着响应系统无法再追踪变化。

```vue
var obj = {
  foo: 'bar'
}

Object.freeze(obj)

new Vue({
  el: '#app',
  data: obj
})
<div id="app">
  <p>{{ foo }}</p>
  <!-- 这里的 `foo` 不会更新！ -->
  <button v-on:click="foo = 'baz'">Change it</button>
</div>
```

2. 实例生命周期钩子

每个Vue实例在被创建时都要经过一系列的初始化过程---设置数据监听、编译模板、将实例挂载到DOM并在数据变化时更新DOM、运行生命周期钩子函数等。生命周期钩子的 `this` 上下文指向调用它的 Vue 实例。**生命周期钩子函数能使用箭头函数吗？为什么？**因为箭头函数并没有this，this会作为变量一直向上级词法作用域查找，直至找到为止，经常导致`Uncaught TypeError: Cannot read property of undefined` 或 `Uncaught TypeError: this.myMethod is not a function` 之类的错误。

3. 模板

Vue 将模板编译成虚拟 DOM 渲染函数。结合响应系统，Vue 能够智能地计算出最少需要重新渲染多少组件，并把 DOM 操作次数减到最少。也可以不用模板，[直接写渲染 (render) 函数](https://cn.vuejs.org/v2/guide/render-function.html)，使用可选的 JSX 语法，比模板更接近编译器。

4. @vue/cli搭建项目

自定义选项：

Use class-style component syntax? (Y/n) 是否使用class风格的组件语法（即，安装vue-class-component和vue-property-decorator。[vue-class-component](https://github.com/vuejs/vue-class-component)是一个支持es6 class风格来开发组件的vue官方库，并使vue组件可以使用继承、混入等特性。[vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)是基于vue-class-component扩展的一个非官方库, 主要功能就和名字一样，为vue的属性提供[装饰器](https://es6.ruanyifeng.com/#docs/decorator)写法，让代码更加美观、直白。）

Sass/SCSS (with dart-sass)和Sass/SCSS (with node-sass)：node-sass是自动编译实时的，dart-sass需要保存后才会生效。在大型项目中 使用`dart-sass`比 `node-sass`（`js`本机`C`库的包装）要慢于很多。

5. 指令

- v-html：输出真正的HTML
- v-once：一次性地插值
- v-bind：<a v-bind:href="url"></a>将该元素的href属性与表达式url的值绑定。支持动态参数，如<a v-bind:[attributeName]='url'></a>（动态参数为null时可用于移除绑定）。v-bind可以省略
- v-on：用于监听DOM事件。支持动态参数，如<a v-on:[eventName]='doSomething'></a>（动态参数为null时可用于移除绑定）。v-on:可缩写为@。有时也需要访问原始的 DOM 事件，可以用特殊变量 `$event` 把它传入方法。

6. 事件修饰符

   .stop

   .prevent

   .capture    添加事件监听器时使用事件捕获模式，即内部元素触发的事件先在此处理，然后才交由内部元素进行处理

   .self    只当在 event.target 是当前元素自身时触发处理函数，即事件不是从内部元素触发的

   .once    事件将只会触发一次，`.once` 修饰符还能被用到自定义的[组件事件](https://cn.vuejs.org/v2/guide/components-custom-events.html)上

   .passive    `.passive` 会告诉浏览器你*不*想阻止事件的默认行为，`.passive` 修饰符尤其能够提升移动端的性能，不要把 `.passive` 和 `.prevent` 一起使用，因为 `.prevent` 将会被忽略

   ```vue
   <form v-on:submit.prevent="onSubmit"></form>
   .prevent修饰符告诉v-on指令对于触发的事件调用event.preventDefault()
   
   .stop修饰符告诉v-on指令对于触发的事件调用event.stopPropagation()
   
   修饰符可以串联，顺序很重要
   <a v-on:click.stop.prevent="doThat"></a>
   v-on:click.prevent.self 会阻止所有的点击，而 v-on:click.self.prevent 只会阻止对元素自身的点击
   
   <div v-on:scroll.passive="onScroll">...</div>
   滚动事件的默认行为 (即滚动行为) 将会立即触发，而不会等待 `onScroll` 完成，这其中包含 `event.preventDefault()` 的情况
   ```

7. 按键修饰符

   - `.enter`
   - `.tab`
   - `.delete` (捕获“删除”和“退格”键)
   - `.esc`
   - `.space`
   - `.up`
   - `.down`
   - `.left`
   - `.right`
   - `.page-down`
   - `.ctrl`
   - `.alt`
   - `.shift`
   - `.meta`
   - `.exact` 修饰符允许你控制由精确的系统修饰符组合触发的事件
   - `.left` 鼠标按钮修饰符
   - `.right` 鼠标按钮修饰符
   - `.middle` 鼠标按钮修饰符

8. 当一个 ViewModel 被销毁时，所有的事件处理器都会自动被删除。你无须担心如何清理它们。

9. 计算属性

   ```js
   computed: {
       // 计算属性的 getter
       reversedMessage: function () {
         // `this` 指向 vm 实例
         return this.message.split('').reverse().join('')
       }
   }
   /*
   只要message还没有发生改变，多次访问reversedMessage计算属性会立即返回之前的计算结果，而不必再次执行函数
   */
   
   // 下面的计算属性不会更新，因为Date.now()不是响应式依赖
   computed: {
     now: function () {
       return Date.now()
     }
   }
   ```

   计算属性是基于响应式依赖进行缓存的，只在相关响应式依赖发生改变时才会重新求值。

   计算属性也可以设置setter

   ```js
   computed: {
     fullName: {
       // getter
       get: function () {
         return this.firstName + ' ' + this.lastName
       },
       // setter
       set: function (newValue) {
         var names = newValue.split(' ')
         this.firstName = names[0]
         this.lastName = names[names.length - 1]
       }
     }
   }
   ```

10. 侦听器

    当需要在数据变化时执行异步或开销较大的操作时，使用侦听器最有用。

    ```vue
    <div id="watch-example">
      <p>
        Ask a yes/no question:
        <input v-model="question">
      </p>
      <p>{{ answer }}</p>
    </div>
    
    <script>
    var watchExampleVM = new Vue({
      el: '#watch-example',
      data: {
        question: '',
        answer: 'I cannot give you an answer until you ask a question!'
      },
      watch: {
        // 如果 `question` 发生改变，这个函数就会运行
        question: function (newQuestion, oldQuestion) {
          this.answer = 'Waiting for you to stop typing...'
          this.debouncedGetAnswer()
        }
      },
      created: function () {
        this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
      },
      methods: {
        getAnswer: function () {
          if (this.question.indexOf('?') === -1) {
            this.answer = 'Questions usually contain a question mark. ;-)'
            return
          }
          this.answer = 'Thinking...'
          var vm = this
          axios.get('https://yesno.wtf/api')
            .then(function (response) {
              vm.answer = _.capitalize(response.data.answer)
            })
            .catch(function (error) {
              vm.answer = 'Error! Could not reach the API. ' + error
            })
        }
      }
    })
    </script>
    ```

9. v-bind:class

   ```vue
   <div v-bind:class="{active: isActive}"></div>
   <div
        class="static"
        v-bind:class="{active: isActive, 'text-danger': hasError}"
   ></div>
   <div v-bind:class="[{active: isActive}, errorClass]"></div>
   ```

   当在一个自定义组件上使用class属性时，这些class将被添加到该组件的根元素上面，这个元素上已经存在的class不会被覆盖。

   ```vue
   Vue.component('my-component', {
   	template: '<p class="foo bar">Hi</p>'
   })
   <my-component v-bind:class="{active: isActive}"></my-component>
   // 渲染为：
   <p class="foo bar active"></p>
   ```

10. v-bind:style

    ```html
    <div v-bind:style="{color: activeColor, fontSize: fontSize+'px'}"></div>
    <div v-bind:style="[baseStyles, overridingStyles]"></div>
    ```

11. v-if v-else v-else-if

    v-else元素必须紧跟在带v-if或者v-else-if的元素的后面，否则将不会被识别。

    v-else-if也必须紧跟在带v-if或者v-else-if的元素之后。

    不推荐同时使用v-if和v-for

    当v-if和v-for一起使用时，v-for有更高的优先级，这意味着v-if将分别重复运行于每个v-for循环中

12. 用key管理可复用的元素

    Vue会尽可能高效地渲染元素，通常会复用已有元素而不是从头开始渲染。

    ```vue
    <template v-if="loginType === 'username'">
    	<label>Username</label>
    	<input placeholder="Enter your username" />
    </template>
    <template v-else>
    	<label>Email</label>
    	<input placeholder="Enter your email address" />
    </template>
    ```

    在上面的代码中切换loginType，将不会清除input中的内容，因为两个模板使用了相同的元素，<input>不会被替换掉，仅会替换它的placeholder。

    想要元素不被复用，就在元素上添加key属性。

13. v-show

    带有v-show的元素始终会被渲染并保留在DOM中。v-show只是简单地切换元素的css属性display

    与v-if的区别：

    - v-if可以用在<template>上，v-show不可以
    - v-if，会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建
    - v-if是惰性的：如果在初始渲染时条件为假，则什么也不做，直到条件第一次变为真时才会开始渲染条件块
    - v-show，不管渲染条件是真是假，元素总会被渲染，只是简单地基于css进行切换
    - v-if有更高的切换开销，v-show有更高的初始渲染开销。如果需要非常频繁地切换则使用v-show较好；如果在运行时条件很少改变则使用v-if较好

14. v-for

    ```vue
    <!--数组-->
    <ul id="example-2">
      <li v-for="(item, index) in items">
        {{ parentMessage }} - {{ index }} - {{ item.message }}
      </li>
    </ul>
    <!--对象-->
    <div v-for="(value, name, index) in object">
      {{ index }}. {{ name }}: {{ value }}
    </div>
    ```

    - 可以用of替代in
    - 在遍历对象时，会按Object.keys()的结果遍历，但是不能保证它的结果在不同的JavaScript引擎中都一致
    - 当Vue正在更新  使用v-for渲染的  元素列表时，默认使用“就地更新”的策略。如果数据项的顺序被改变，Vue将不会移动DOM元素来匹配数据项的顺序，而是就地更新每个元素，并确保它们在每个索引位置正确渲染
    - v-for和key结合使用，以便Vue能跟踪每个节点的身份，从而重用和重新排序现有元素
    - Vue将被侦听的数组的变更方法（push() pop() shift() unshift() splice() sort() reverse()）进行了包裹，会触发视图更新
    - 对于非变更方法，用一个含有相同元素的数组去替换原来的数组是非常高效的操作，比如items = items.filter(...)
    - v-for可以用于<template>元素
    - 当在组件上使用v-for时，key是必须的

15. 任何数据都不会被自动传递到组件里，因为组件有自己独立的作用域

18. is属性（针对有严格限制的HTML元素解析DOM模板的变通方法）

    ```vue
    // 有些 HTML 元素，诸如 <ul>、<ol>、<table> 和 <select>，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，诸如 <li>、<tr> 和 <option>，只能出现在其它某些特定的元素内部。
    // 如果想在table元素中使用自定义组件，会被作为无效的内容提升到外部，并导致最终渲染结果出错，即不能直接如下使用：
    <table>
        <blog-post-row></blog-post-row>
    </table>
    // 可以使用is属性进行变通
    <table>
        <tr is="blog-post-row"></tr>
    </table>
    ```

19. 表单输入绑定（v-model）

    - v-model指令用于在表单<input>、<textarea>、<select>元素上创建双向数据绑定

    - v-model会忽略所有表单元素的value、checked、selected属性的初始值而总是将Vue实例的数据作为数据来源

    - v-model不会在输入法组合文字过程中得到更新

    - 对于input(type='text')，v-model内部会利用input(type='text')的value属性和input事件

    - 对于input(type='checkbox')和input(type='radio')，v-model内部会利用checked属性和change事件

    - 对于textarea，v-model内部会利用value属性和input事件

    - 对于select，v-model内部会利用被选中的option的value属性，以及利用select的change事件。多选时绑定到一个数组

    - 值绑定

      ```vue
      // v-model绑定的值通常是静态字符串(对于复选框也可以是布尔值)
      <!-- 当选中时，`picked` 为字符串 "a" -->
      <input type="radio" v-model="picked" value="a">
      
      <!-- `toggle` 为 true 或 false -->
      <input type="checkbox" v-model="toggle">
      
      <!-- 当选中第一个选项时，`selected` 为字符串 "abc" -->
      <select v-model="selected">
        <option value="abc">ABC</option>
      </select>
      
      // 可以用v-bind实现v-model绑定的值是Vue实例的一个动态property，这个property可以不仅仅是字符串或布尔值
      <select v-model="selected">
          <!-- 内联对象字面量 -->
        <option v-bind:value="{ number: 123 }">123</option>
      </select>
      // 当选中时
      typeof vm.selected // => 'object'
      vm.selected.number // => 123
      ```

    - 修饰符---.lazy：在默认情况下，v-model在每次input事件触发后将输入框的值与数据进行同步（除了输入法组合文字时）。使用.lazy修饰符可以转为在change事件之后进行同步
    - 修饰符---.number：该修饰符会自动将用户的输入值转为数值类型，如果输入值无法被parseFloat()解析，则会返回原始的输入值
    - 修饰符---.trim：该修饰符会自动过滤输入值的首尾空白字符

20. 组件基础

    - 组件是可复用的Vue实例，所以组件与new Vue接收相同的选项，如data、computed、watch、methods以及生命周期钩子等。仅有的例外是el，它是根实例特有的选项。
    - 组件可以被复用，每用一次组件，就会有一个它的新实例被创建。
    - 一个组件的data必须是一个函数，这样它的每个实例可以维护一份被返回对象的独立的拷贝，不会影响到该组件的其他实例
    - 组件的props选项用于维护该组件接收的prop列表，可以用v-bind:来动态传递prop
    - 父组件可以通过v-on监听子组件实例的任意事件；子组件可以通过调用内建的$emit方法并传入事件名称来触发一个事件

21. 在组件上使用v-model（v-model也可以自定义，参见26）

    ```vue
    <input v-model='searchText' />
    // 等价于
    <input
        v-bind:value='searchText'
        v-on:input='searchText = $event.target.value'
    />
    
    // 在组件上使用v-model
    // 注册组件
    Vue.component('custom-input', {
        props: ['value'],
        template: `
            <input v-bind:value='value' v-on:input='$emit('input', $event.target.value)'/>
        `
    })
    // 使用v-model
    <custom-input v-model='searchText'></custom-input>
    
    // 等价于
    <custom-input
        v-bind:value='searchText'
        v-on:input='searchText = $event'
    ></custom-input>
    ```

22. 自定义组件标签体内写内容------用slot

    ```vue
    Vue.component('alert-box', {
        template: `
            <div>
              <strong>Error</strong>
              <slot></slot>
            </div>
        `
    })
    <alert-box>Something bad happeded.</alert-box>
    ```

23. 动态组件

    ```vue
    <component v-bind:is='currentTabComponent'></component>
    
    // 组件会在currentTabComponent改变时改变
    // currentTabComponent可以是已注册组件的名字，或者是一个组件的选项对象
    ```

24. 基础组件的自动化全局注册

    ```js
    // 可能你的许多组件只是包裹了一个输入框或按钮之类的元素，是相对通用的。我们有时候会把它们称为基础组件，它们会在各个组件中被频繁的用到。不想频繁import基础组件的话，可以全局注册。
    // 注意，全局注册的行为必须在Vue根实例(通过new Vue)创建之前发生。
    
    // 在main.js中全局注册基础组件
    import Vue from 'vue'
    import upperFirst from 'lodash/upperFirst'
    import camelCase from 'lodash/camelCase'
    
    const requireComponent = require.context(
      // 其组件目录的相对路径
      './components',
      // 是否查询其子目录
      false,
      // 匹配基础组件文件名的正则表达式
      /Base[A-Z]\w+\.(vue|js)$/
    )
    
    requireComponent.keys().forEach(fileName => {
      // 获取组件配置
      const componentConfig = requireComponent(fileName)
    
      // 获取组件的 PascalCase 命名
      const componentName = upperFirst(
        camelCase(
          // 获取和目录深度无关的文件名
          fileName
            .split('/')
            .pop()
            .replace(/\.\w+$/, '')
        )
      )
    
      // 全局注册组件
      Vue.component(
        componentName,
        // 如果这个组件选项是通过 `export default` 导出的，
        // 那么就会优先使用 `.default`，
        // 否则回退到使用模块的根。
        componentConfig.default || componentConfig
      )
    })
    ```

25. Prop

    - Prop类型可以是数组或对象

      ```js
      props: ['title', 'commentIds']
      props: {
          title: String,
          commentIds: Array,
          author: Object,
          callback: Function,
          contactsPromise: Promise
      }
      ```

    - Prop可以静态传递或动态传递(v-bind:)。传入一个数字或一个布尔值或一个数组或一个对象都需要动态传递，否则会被当作字符串。

      ```vue
      <blog-post v-bind:likes="42"></blog-post>
      <blog-post v-bind:is-published="false"></blog-post>
      <blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>
      <blog-post
        v-bind:author="{
          name: 'Veronica',
          company: 'Veridian Dynamics'
        }"
      ></blog-post>
      ```

    - 若要将一个对象的所有property都作为prop传入，可以使用v-bind代替v-bind:prop-name

      ```vue
      post: {
          id: 1,
          title: 'test'
      }
      <blog-post v-bind='post'></blog-post>
      
      // 等价于
      <blog-post v-bind:id='post.id' v-bind:title='post.title'></blog-post>
      ```

    - 单向数据流。每次父级组件发生变更时，子组件中所有的prop都将会刷新为最新的值。不应该在一个子组件内部改变prop，否则会影响到父组件的状态。

      ```js
      // 有两种常见的试图变更一个prop的情形：
      // 1.这个prop用来传递一个初始值，这个子组件希望将其作为一个本地的数据来使用。在这种情况下，最好定义一个data property并将这个prop用作其初始值。
      props: ['initCount'],
      data: function() {
        return {
          count: this.initCount
        }
      }
      // 2.这个prop以一种原始的值传入且需要进行转换。在这种情况下，最好使用这个prop的值来定义一个计算属性
      props: ['size'],
      computed: {
        normalizedSize: function() {
          return this.size.trim().toLowerCase()
        }
      }
      ```

    - Prop验证

      ```js
      Vue.component('my-component', {
        props: {
          // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
          propA: Number,
          // 多个可能的类型
          propB: [String, Number],
          // 必填的字符串
          propC: {
            type: String,
            required: true
          },
          // 带有默认值的数字
          propD: {
            type: Number,
            default: 100
          },
          // 带有默认值的对象
          propE: {
            type: Object,
            // 对象或数组默认值必须从一个工厂函数获取
            default: function () {
              return { message: 'hello' }
            }
          },
          // 自定义验证函数
          propF: {
            validator: function (value) {
              // 这个值必须匹配下列字符串中的一个
              return ['success', 'warning', 'danger'].indexOf(value) !== -1
            }
          }
        }
      })
      
      // type可以是String Number Boolean Array Object Date Function Symbol，还可以是一个自定义的构造函数
      // 注意：prop会在一个组件实例创建之前进行验证，所以实例的property（如data、computed等）在default或validator函数中是不可用的
      ```

    - 组件可以接受任意的attribute（包括非prop的attribute），这些attribute会被添加到这个组件的根元素上。对于绝大多数attribute来说，从外部提供给组件的值会替换掉组件内部设置好的值。而class和style这两个attribute会把传入的值和内部设置的值合并起来。

    - inheritAttrs和$attrs

      ```js
      // 当一个组件没有声明任何prop，$attrs会包含所有父作用域的attribute名和attribute值（class和style除外）；子组件在props中声明过的属性，不会在$attrs对象中（可console.log(this.$attrs);查看）
      // inheritAttrs默认为true，true的意思是将父组件中除了子组件props声明外的属性添加到子组件的根节点上
      // 注意：即使inheritAttrs设为false，仍然可以通过$attrs获取到props之外的attribute名和attribute值
      ```

      ```vue
      <template>
       <div>
        <p>我是父组件</p>
        <test name="tom" age="12" class="child" style="color: red" />
       </div>
      </template>
      <script>
      export default {
       components: {
        test: {
         template: `<p>我是子组件</p>`,
         props: ["name"],
         inheritAttrs: true, // 默认为 true
         created() {
          console.log(this.$attrs); // {age: "12"}
         }
        }
       }
      };
      </script>
      ```

      可以看到父组件传给子组件传了 name、age 两个属性，其中 name 属性在子组件 props 中声明，以上代码浏览器解析后

      ```vue
      <div>
       <p data-v-469af010>我是父组件</p>
       <p data-v-469af010 age="12" class="child" style="color: red;">我是子组件</p>
      </div>
      ```

      可以看到没有在子组件 props 中声明的 age 属性被写到了标签里，如果你不希望这样，则可以把子组件中的 inheritAttrs 设为 false，解析后的结果如下

      ```vue
      <div>
       <p data-v-469af010>我是父组件</p>
       <p data-v-469af010 class="child" style="color: red;">我是子组件</p>
      </div>
      ```

      可以看到标签中的属性消失了，同时可以看到 class、style 属性不受影响

      

26. 自定义事件

    - **推荐事件名使用kebab-case形式**

    - 自定义v-model。一个组件上的v-model默认会利用名为value的prop和名为input的事件

      ```vue
      Vue.component('base-checkbox', {
          model: {
              prop: 'checked',
              event: 'change'
          },
          props: { checked: Boolean },
          template: `
              <input
                  type='checkbox'
                  v-bind:checked='checked'
                  v-on:change="$emit('change', $event.target.checked)"
              />
          `
      })
      <base-checkbox v-model='testModel'></base-checkbox>
      // testModel的值将会传入这个名为checked的prop，当<base-checkbox>触发change事件并附带一个新值时，testModel会被更新
      ```

    - 将原生事件绑定到组件。使用.native修饰符可以在一个组件的根元素上直接监听一个原生事件。但自定义组件的根元素不一定有这个事件(比如，自定义组件的根元素是label标签，而给这个组件监听了focus事件v-on:focus.native)，这时就需要用到$listeners

    - $listeners是一个对象，包含了作用在这个组件上的所有监听器，例如：

      ```js
      {
          focus: function(event) { /* ... */ },
          input: function(value) { /* ... */ }
      }
      ```

      可以使用v-on="$listeners"将所有的事件监听器指向这个组件的某个特定的子元素

      ```vue
      Vue.component('base-input', {
          inheritAttrs: false,
          props: ['label', 'value'],
          computed: {
              inputListeners: function() {
                  var vm = this
                  return Object.assign({}, this.$listeners, {
                      input: function(event) {
                          vm.$emit('input', event.target.value)
                      }
                  })
              }
          },
          template: `
              <label>
                {{ label }}
                <input
                   v-bind="$attrs"
                   v-bind:value="value"
                   v-on="inputListeners"
                />
              </label>
          `
      })
      ```

    - .sync修饰符。带有.sync修饰符的v-bind不能和表达式一起使用，是无效的。

      ```vue
      // text-document组件内部
      this.$emit('update:title', newTitle);
      
      // 父组件中使用text-document组件
      <text-document v-bind:title.sync="doc.title"></text-document>
      
      // 上面使用.sync等价于
      <text-document
          v-bind:title="doc.title"
          v-on:update:title="doc.title = $event"
      ></text-document>
      
      // 把doc对象中的每个property都作为一个独立的prop传递，然后各自添加用于更新的v-on监听器
      <text-document v-bind.sync="doc"></text-document>
      // 注意：v-bind.sync只能和变量结合使用。和字面量对象结合使用是无效的，如v-bind.sync="{ title: 'test' }"
      ```

27. 插槽

    - 对应插槽处可以包含任何模版代码，包括HTML及其他组件

    - 父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的

    - 插槽可以设置默认内容

      ```vue
      // submit-button组件
      <button type='submit'>
        <slot>Submit</slot>
      </button>
      ```

    - 具名插槽

      ```vue
      // <base-layout>组件
      <div class="container">
        <header>
          <!-- 我们希望把页头放这里 -->
          <slot name="header"></slot>
        </header>
        <main>
          <!-- 我们希望把主要内容放这里 -->
          <slot></slot> <!-- 不指定name,则name为"default" -->
        </main>
        <footer>
          <!-- 我们希望把页脚放这里 -->
          <slot name="footer"></slot>
        </footer>
      </div>
        
      // 使用<base-layout>组件
      <base-layout>
        <template v-slot:header>
          <h1>Here might be a page title</h1>
        </template>
      
        <p>A paragraph for the main content.</p>
        <p>And another one.</p>
      
        <template v-slot:footer>
          <p>Here's some contact info</p>
        </template>
      </base-layout>
      // 注意上面写法：任何没有被包裹在带有v-slot的<template>中的内容都会被视为name为default的插槽的内容。或者更明确的写法：
      <base-layout>
        <template v-slot:header> <!-- v-slot:header可以缩写为#header -->
          <h1>Here might be a page title</h1>
        </template>
      
        <template v-slot:default> <!-- v-slot:default可以缩写为#default -->
          <p>A paragraph for the main content.</p>
          <p>And another one.</p>
        </template>
      
        <template v-slot:footer> <!-- v-slot:footer可以缩写为#footer -->
          <p>Here's some contact info</p>
        </template>
      </base-layout>
        
      // 渲染结果
      <div class="container">
        <header>
          <h1>Here might be a page title</h1>
        </header>
        <main>
          <p>A paragraph for the main content.</p>
          <p>And another one.</p>
        </main>
        <footer>
          <p>Here's some contact info</p>
        </footer>
      </div>
      ```

    - 作用域插槽------让子组件的对象在父组件中使用。

      ```vue
      // current-user组件
      <span>
        <slot v-bind:user="user">
          {{ user.lastName }}
        </slot>
      </span>
      // 父组件
      <current-user>
        <template v-slot:default="slotProps"> <!-- v-slot:default可以缩写为#default -->
          {{ slotProps.user.firstName }}
        </template>
      </current-user>
      
      // 1. 绑定在 <slot> 元素上的 attribute 被称为插槽 prop。
      // 2. 上述slotProps是一个包含所有插槽prop的对象，"slotProps"这个名字可以是任意。
      // 3. 如上，当只有默认插槽时，可以简写：
      <current-user v-slot="slotProps">
        {{ slotProps.user.firstName }}
      </current-user>
      // 4. 插槽prop可以使用解构语法
      <current-user v-slot="{ user }">
        {{ user.firstName }}
      </current-user>
      <current-user v-slot="{ user: person }">
        {{ person.firstName }}
      </current-user>
      <current-user v-slot="{ user = {firstName: 'Guest'} }">
        {{ user.firstName }}
      </current-user>
      ```

    - 动态插槽名

      ```vue
      <base-layout>
        <template v-slot:[dynamicSlotName]>
          ...
        </template>
      </base-layout>
      ```

28. 异步组件。可以和webpack的code-splitting功能结合使用。

    ```js
    const AsyncComponent = () => ({
      // 需要加载的组件 (应该是一个 `Promise` 对象)
      component: import('./MyComponent.vue'),
      // 异步组件加载时使用的组件
      loading: LoadingComponent,
      // 加载失败时使用的组件
      error: ErrorComponent,
      // 展示加载时组件的延时时间。默认值是 200 (毫秒)
      delay: 200,
      // 如果提供了超时时间且组件加载也超时了，
      // 则使用加载失败时使用的组件。默认值是：`Infinity`
      timeout: 3000
    })
    ```

29. 处理边界情况(不建议使用)

    - 访问根实例  this.$root

    - 访问父级组件实例  this.$parent

    - 访问子组件实例或子元素  ref  this.$refs。当ref和v-for一起使用时，ref将会是一个包含了对应数据源的子组件的数组。$refs只会在组件渲染完成后生效，并且不是响应式的，应避免在模板或计算属性中访问$refs。

    - 依赖注入。provide和inject。provide指定提供给后代组件的数据/方法（注意，该数据/方法是非响应式的），在任何后代组件里都可以用inject接收。

    - 事件侦听器。通过$on(eventName, eventHandler)侦听一个事件；通过$once(eventName, eventHandler)一次性侦听一个事件；通过$off(eventName, eventHandler)停止侦听一个事件

    - 递归组件。组件是可以在自己的模板中调用自身的。

    - 组件之间的循环引用，比如组件A里使用组件B，组件B里使用组件A。可以在注册组件时使用webpack的异步import，或者在beforeCreate里注册

      ```js
      components: {
        TreeFolderContents: () => import('./tree-folder-contents.vue')
      }
      // 或者
      beforeCreate: function () {
        this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue').default
      }
      ```

    - 内联模板 inline-template。把标签体内容当作模板。

      ```vue
      <my-component inline-template>
        <div>
          <p>These are compiled as the component's own template.</p>
          <p>Not parent's transclusion content.</p>
        </div>
      </my-component>
      ```

    - 另一个定义模板的方式 text/x-template

      ```vue
      <script type="text/x-template" id="hello-world-template">
        <p>Hello hello hello</p>
      </script>
      Vue.component('hello-world', {
        template: '#hello-world-template'
      })
      ```

    - 手动强制更新：$forceUpdate

    - 通过v-once创建低开销的静态组件。如果一个组件包含了大量的静态内容，可以在根元素上使用v-once，让这些内容只计算一次然后缓存起来

      ```vue
      Vue.component('terms-of-service', {
        template: `
          <div v-once>
            <h1>Terms of Service</h1>
            ... a lot of static content ...
          </div>
        `
      })
      ```

30. 过渡动画

    - 单元素/组件的过渡 transition

      ```vue
      <div id="demo">
        <button v-on:click="show = !show">
          Toggle
        </button>
        <transition name="fade">
          <p v-if="show">hello</p>
        </transition>
      </div>
      new Vue({
        el: '#demo',
        data: {
          show: true
        }
      })
      <style>
      .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
      }
      .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
        opacity: 0;
      }
      </style>
      
      /*
      当插入或删除包含在transition组件中的元素时，Vue将会做以下处理：
      1.自动嗅探目标元素是否应用了css过渡或动画，如果是，则在恰当的时机添加/删除css类名
      2.如果过渡组件提供了JavaScript钩子函数，这些钩子函数将在恰当的时机被调用
      3.如果没有找到JavaScript钩子并且也没有检测到css过渡/动画，DOM操作（插入/删除）在下一帧中立即执行。（注意：此指浏览器逐帧动画机制，和Vue的nextTick概念不同）
      */
      ```

    - 过渡的类名

    - 

    - 

31. 混入mixin

    - 一个混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项。

      ```js
      // 1.数据对象在内部会进行递归合并，并在发生冲突时以组件数据优先
      var mixin = {
        data: function () {
          return {
            message: 'hello',
            foo: 'abc'
          }
        }
      }
      
      new Vue({
        mixins: [mixin],
        data: function () {
          return {
            message: 'goodbye',
            bar: 'def'
          }
        },
        created: function () {
          console.log(this.$data)
          // => { message: "goodbye", foo: "abc", bar: "def" }
        }
      })
      
      // 2.同名钩子函数将合并为一个数组，都将被调用，混入对象的钩子将在组件自身钩子之前调用
      var mixin = {
        created: function () {
          console.log('混入对象的钩子被调用')
        }
      }
      
      new Vue({
        mixins: [mixin],
        created: function () {
          console.log('组件钩子被调用')
        }
      })
      
      // => "混入对象的钩子被调用"
      // => "组件钩子被调用"
      
      // 3.值为对象的选项，例如methods、components、directives，将被合并为同一个对象。两个对象键名冲突时，取组件对象的键值对
      var mixin = {
        methods: {
          foo: function () {
            console.log('foo')
          },
          conflicting: function () {
            console.log('from mixin')
          }
        }
      }
      
      var vm = new Vue({
        mixins: [mixin],
        methods: {
          bar: function () {
            console.log('bar')
          },
          conflicting: function () {
            console.log('from self')
          }
        }
      })
      
      vm.foo() // => "foo"
      vm.bar() // => "bar"
      vm.conflicting() // => "from self"
      
      // Vue.extend()也使用上述的策略进行合并
      ```

    - 全局混入。混入也可以进行全局注册，一旦使用全局混入，它将影响每一个之后创建的Vue实例

32. 自定义指令---对普通DOM元素进行底层操作

    ```js
    // 注册一个全局自定义指令v-focus
    Vue.directive('focus', {
        // 当被绑定的元素插入到DOM中时
        inserted: function(el) {
            // 聚焦元素
            el.focus()
        }
    })
    // 注册局部指令v-focus
    directives: {
      focus: {
        inserted: function(el) {
          el.focus()
        }
      }
    }
    
    // 示例
    Vue.directive('demo', {
      bind: function (el, binding, vnode) {
        var s = JSON.stringify
        el.innerHTML =
          'name: '       + s(binding.name) + '<br>' +
          'value: '      + s(binding.value) + '<br>' +
          'expression: ' + s(binding.expression) + '<br>' +
          'argument: '   + s(binding.arg) + '<br>' +
          'modifiers: '  + s(binding.modifiers) + '<br>' +
          'vnode keys: ' + Object.keys(vnode).join(', ')
      }
    })
    
    new Vue({
      el: '#hook-arguments-example',
      data: {
        message: 'hello!'
      }
    })
    /*
    name: "demo"
    value: "hello!"
    expression: "message"
    argument: "foo"
    modifiers: {"a":true,"b":true}
    vnode keys: tag, data, children, text, elm, ns, context, fnContext, fnOptions, fnScopeId, key, componentOptions, componentInstance, parent, raw, isStatic, isRootInsert, isComment, isCloned, isOnce, asyncFactory, asyncMeta, isAsyncPlaceholder
    */
    ```

    - 自定义指令钩子函数---inserted：被绑定元素插入父节点时调用（仅保证父节点存在，但不一定已被插入文档中）

    - 自定义指令钩子函数---bind：只调用一次，指令第一次绑定到元素时调用，可用于一次性的初始化设置

    - 自定义指令钩子函数---update：所在组件的VNode更新时调用，但是可能发生在其子VNode更新之前。指令的值可能发生了改变，也可能没有，但是可以通过比较更新前后的值来忽略不必要的模板更新

    - 自定义指令钩子函数---componentUpdated：指令所在组件的VNode及其子VNode全部更新后调用

    - 自定义指令钩子函数---unbind：只调用一次，指令与元素解绑时调用

    - 自定义指令钩子函数的参数---el：指令所绑定的元素，可以用来直接操作DOM

    - 自定义指令钩子函数的参数---binding：一个对象，包含以下property：

      - name：指令名，不包括v-前缀
      - value：指令的绑定值。例如v-my-directive="1+1"中，value为2。value可以是一个JavaScript对象字面量，比如v-demo="{ color: 'white', text: 'hello!' }"
      - oldValue：指令绑定的前一个值，仅在update和componentUpdated钩子中可用。无论值是否改变都可用
      - expression：字符串形式的指令表达式。例如v-my-directive="1+1"中，expression为"1+1"
      - arg：传给指令的参数，可选。例如v-my-directive:foo中，arg为"foo"
      - modifiers：一个包含修饰符的对象。例如v-my-directive.foo.bar中，修饰符对象为{foo: true, bar: true}

    - 自定义指令钩子函数的参数---vnode：Vue编译生成的虚拟节点

    - 自定义指令钩子函数的参数---oldVnode：上一个虚拟节点，仅在update和componentUpdated钩子中可用。

    - 传给指令的参数(arg)可以是动态的，v-my-directive:[argument]="value"，参数argument可以根据组件实例数据进行更新

      ```vue
      <div id="dynamicexample">
        <h3>Scroll down inside this section ↓</h3>
        <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>
      </div>
      
      Vue.directive('pin', {
        bind: function (el, binding, vnode) {
          el.style.position = 'fixed'
          var s = (binding.arg == 'left' ? 'left' : 'top')
          el.style[s] = binding.value + 'px'
        }
      })
      
      new Vue({
        el: '#dynamicexample',
        data: function () {
          return {
            direction: 'left'
          }
        }
      })
      ```

33. 渲染函数

    - 虚拟节点(VNode)：节点的描述信息

    - 虚拟DOM：由Vue组件树建立起来的整个VNode树

      ```js
      // @returns {VNode}
      createElement(
        // {String | Object | Function}
        // 一个 HTML 标签名、组件选项对象，或者
        // resolve 了上述任何一种的一个 async 函数。必填项。
        'div',
      
        // {Object}
        // 一个与模板中 attribute 对应的数据对象。可选。
        {
          // 与 `v-bind:class` 的 API 相同，
          // 接受一个字符串、对象或字符串和对象组成的数组
          'class': {
            foo: true,
            bar: false
          },
          // 与 `v-bind:style` 的 API 相同，
          // 接受一个字符串、对象，或对象组成的数组
          style: {
            color: 'red',
            fontSize: '14px'
          },
          // 普通的 HTML attribute
          attrs: {
            id: 'foo'
          },
          // 组件 prop
          props: {
            myProp: 'bar'
          },
          // DOM property
          domProps: {
            innerHTML: 'baz'
          },
          // 事件监听器在 `on` 内，
          // 但不再支持如 `v-on:keyup.enter` 这样的修饰器。
          // 需要在处理函数中手动检查 keyCode。
          on: {
            click: this.clickHandler
          },
          // 仅用于组件，用于监听原生事件，而不是组件内部使用
          // `vm.$emit` 触发的事件。
          nativeOn: {
            click: this.nativeClickHandler
          },
          // 自定义指令。注意，你无法对 `binding` 中的 `oldValue`
          // 赋值，因为 Vue 已经自动为你进行了同步。
          directives: [
            {
              name: 'my-custom-directive',
              value: '2',
              expression: '1 + 1',
              arg: 'foo',
              modifiers: {
                bar: true
              }
            }
          ],
          // 作用域插槽的格式为
          // { name: props => VNode | Array<VNode> }
          scopedSlots: {
            default: props => createElement('span', props.text)
          },
          // 如果组件是其它组件的子组件，需为插槽指定名称
          slot: 'name-of-slot',
          // 其它特殊顶层 property
          key: 'myKey',
          ref: 'myRef',
          // 如果你在渲染函数中给多个元素都应用了相同的 ref 名，
          // 那么 `$refs.myRef` 会变成一个数组。
          refInFor: true
        },
      
        // {String | Array}
        // 子级虚拟节点 (VNodes)，由 `createElement()` 构建而成，
        // 也可以使用字符串来生成“文本虚拟节点”。可选。
        [
          '先写一些文字',
          createElement('h1', '一则头条'),
          createElement(MyComponent, {
            props: {
              someProp: 'foobar'
            }
          })
        ]
      )
      ```

    - v-if和v-for

      ```vue
      <ul v-if="items.length">
        <li v-for="item in items">{{ item.name }}</li>
      </ul>
      <p v-else>No items found.</p>
      
      // 使用渲染函数
      props: ['items'],
      render: function (createElement) {
        if (this.items.length) {
          return createElement('ul', this.items.map(function (item) {
            return createElement('li', item.name)
          }))
        } else {
          return createElement('p', 'No items found.')
        }
      }
      ```

    - v-model

      ```js
      props: ['value'],
      render: function (createElement) {
        var self = this
        return createElement('input', {
          domProps: {
            value: self.value
          },
          on: {
            input: function (event) {
              self.$emit('input', event.target.value)
            }
          }
        })
      }
      ```

    - 事件修饰符

      对于 `.passive`、`.capture` 和 `.once` 这些事件修饰符，Vue 提供了相应的前缀可以用于 `on`：

      | 修饰符                             | 前缀 |
      | ---------------------------------- | ---- |
      | `.passive`                         | `&`  |
      | `.capture`                         | `!`  |
      | `.once`                            | `~`  |
      | `.capture.once` 或 `.once.capture` | `~!` |

      例如：

      ```js
      on: {
        '!click': this.doThisInCapturingMode,
        '~keyup': this.doThisOnce,
        '~!mouseover': this.doThisOnceInCapturingMode
      }
      ```

      | 修饰符                                      | 处理函数中的等价操作                                         |
      | ------------------------------------------- | ------------------------------------------------------------ |
      | `.stop`                                     | `event.stopPropagation()`                                    |
      | `.prevent`                                  | `event.preventDefault()`                                     |
      | `.self`                                     | `if (event.target !== event.currentTarget) return`           |
      | 按键： `.enter`, `.13`                      | `if (event.keyCode !== 13) return` (对于别的按键修饰符来说，可将 `13` 改为[另一个按键码](http://keycode.info/)) |
      | 修饰键： `.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return` (将 `ctrlKey` 分别修改为 `altKey`、`shiftKey` 或者 `metaKey`) |

      ```js
      on: {
        keyup: function (event) {
          // 如果触发事件的元素不是事件绑定的元素
          // 则返回
          if (event.target !== event.currentTarget) return
          // 如果按下去的不是 enter 键或者
          // 没有同时按下 shift 键
          // 则返回
          if (!event.shiftKey || event.keyCode !== 13) return
          // 阻止 事件冒泡
          event.stopPropagation()
          // 阻止该元素默认的 keyup 事件
          event.preventDefault()
          // ...
        }
      }
      ```

    - 插槽

      可以通过this.$slots访问静态插槽的内容，每个插槽都是一个VNode数组。

      ```js
      render: function (createElement) {
        // `<div><slot></slot></div>`
        return createElement('div', this.$slots.default)
      }
      ```

      也可以通过this.$scopedSlots访问作用域插槽，每个作用域插槽都是一个返回若干VNode的函数。

      ```js
      props: ['message'],
      render: function (createElement) {
        // `<div><slot :text="message"></slot></div>`
        return createElement('div', [
          this.$scopedSlots.default({
            text: this.message
          })
        ])
      }
      ```

      如果要用渲染函数向子组件中传递作用域插槽，可以利用VNode数据对象中的scopedSlots字段。

      ```js
      render: function (createElement) {
        // `<div><child v-slot="props"><span>{{ props.text }}</span></child></div>`
        return createElement('div', [
          createElement('child', {
            // 在数据对象中传递 `scopedSlots`
            // 格式为 { name: props => VNode | Array<VNode> }
            scopedSlots: {
              default: function (props) {
                return createElement('span', props.text)
              }
            }
          })
        ])
      }
      ```

34. JSX

    ```js
    import AnchoredHeading from './AnchoredHeading.vue'
    
    new Vue({
      el: '#demo',
      render: function (h) {
        return (
          <AnchoredHeading level={1}>
            <span>Hello</span> world!
          </AnchoredHeading>
        )
      }
    })
    /*
    将h作为createElement的别名是Vue生态系统中的一个通用惯例，实际上也是JSX所要求的。从Vue的Babel插件3.4.0版本开始，会在以ES2015语法声明的含有JSX的任何方法和getter中（不是函数或箭头函数中）自动注入const h = this.$createElement，可以去掉(h)参数。
    */
    ```

35. 函数式组件---无状态(没有响应式数据)、无实例(没有this上下文)、渲染开销低

    ```vue
    // 全局注册的函数式组件（functional）
    Vue.component('my-component', {
      functional: true,
      // Props 是可选的
      props: {
        // ...
      },
      // 为了弥补缺少的实例，提供第二个参数作为上下文
      render: function (createElement, context) {
        // ...
      }
    })
    
    // 基于模板的函数式组件（functional）
    <template functional>
    </template
    ```

    context是一个对象，包含以下property：

    - props：提供所有的prop对象

    - children：VNode子节点的数组

    - slots：一个函数，返回了包含所有插槽的对象

    - scopedSlots：一个暴露传入的作用域插槽的对象。也以函数形式暴露普通插槽

    - data：传递给组件的整个数据对象

    - parent：对父组件的引用

    - listeners：一个包含了所有父组件为当前组件注册的事件监听器的对象

    - injections：如果使用了inject选项，则该对象包含了应当被注入的property

      ```vue
      <my-functional-component>
        <p v-slot:foo>
          first
        </p>
        <p>second</p>
      </my-functional-component>
      
      // 对于这个组件，children 会给你两个段落标签，而 slots().default 只会传递第二个匿名段落标签，slots().foo 会传递第一个具名段落标签。
      ```

    函数式组件向子元素或子组件传递attribute和事件

    ```vue
    Vue.component('my-functional-button', {
      functional: true,
      render: function (createElement, context) {
        // 完全透传任何 attribute、事件监听器、子节点等。
        return createElement('button', context.data, context.children)
      }
    })
    
    // 基于模板的函数式组件
    // 使用data.attrs传递任何HTML attribute
    // 使用listeners传递任何事件监听器
    <template functional>
      <button
        class="btn btn-primary"
        v-bind="data.attrs"
        v-on="listeners"
      >
        <slot/>
      </button>
    </template>
    ```

36. 插件---插件通常用来为Vue添加全局功能

    - 插件的功能一般有以下几种：
      - 添加全局方法或者property。如：vue-custom-element
      - 添加全局资源：指令/过滤器/过渡等。如：vue-touch
      - 通过全局混入来添加一些组件选项。如：vue-router
      - 添加Vue实例方法，通过把它们添加到Vue.$prototype上实现
      - 一个库，提供自己的API，同时提供上面提到的一个或多个功能。如：vue-router

    - 使用插件。在new Vue()之前通过全局方法Vue.use()使用插件。即使多次调用Vue.use()来注册相同插件，该插件也只会注册一次。

    - 开发插件。Vue.js的插件应该暴露一个install方法，该方法的第一个参数是Vue构造器，第二个参数是一个可选的选项对象

      ```js
      MyPlugin.install = function (Vue, options) {
        // 1. 添加全局方法或 property
        Vue.myGlobalMethod = function () {
          // 逻辑...
        }
      
        // 2. 添加全局资源
        Vue.directive('my-directive', {
          bind (el, binding, vnode, oldVnode) {
            // 逻辑...
          }
          ...
        })
      
        // 3. 注入组件选项
        Vue.mixin({
          created: function () {
            // 逻辑...
          }
          ...
        })
      
        // 4. 添加实例方法
        Vue.prototype.$myMethod = function (methodOptions) {
          // 逻辑...
        }
      }
      ```

37. 过滤器

    - 过滤器可以用在两个地方：双花括号插值和v-bind表达式

    ```vue
    <!-- 在双花括号中 -->
    {{ message | capitalize }}
    
    <!-- 在 `v-bind` 中 -->
    <div v-bind:id="rawId | formatId"></div>
    ```

    - 当全局过滤器和局部过滤器重名时，会采用局部过滤器

    ```js
    // 局部定义过滤器
    filters: {
      capitalize: function (value) {
        if (!value) return ''
        value = value.toString()
        return value.charAt(0).toUpperCase() + value.slice(1)
      }
    }
    // 全局定义过滤器
    Vue.filter('capitalize', function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    })
    
    new Vue({
      // ...
    })
    ```

    - 过滤器可以接收参数

    ```js
    {{ message | filterA('arg1', arg2) }}
    
    // 过滤器filterA接收三个参数：第一个参数是message的值，第二个参数是'arg1'，第三个参数是arg2的值
    ```

    - 过滤器可以串联

      ```vue
      {{ message | filterA | filterB }}
      ```

38. 关注点分离不等于文件类型分离

39. 测试

    - 单元测试。将独立单元的代码进行隔离测试，在构建新特性或重构已有代码的同时保持应用的功能和稳定。
      - Jest。一个专注于简易性的JavaScript测试框架。一个独特的功能是可以为测试生成快照(snapshot)，以提供另一种验证应用单元的方法。[Vue CLI 官方插件 - Jest](https://cli.vuejs.org/core-plugins/unit-jest.html)
      - Mocha。一个专注于灵活性的 JavaScript 测试框架。因为其灵活性，它允许你选择不同的库来满足诸如侦听 (如 Sinon) 和断言 (如  Chai) 等其它常见的功能。另一个 Mocha 独特的功能是它不止可以在 Node.js 里运行测试，还可以在浏览器里运行测试。[Vue CLI 官方插件 - Mocha](https://cli.vuejs.org/core-plugins/unit-mocha.html)
    - 组件测试
      - Vue Testing Library(@testing-library/vue)。一组专注于测试组件而不依赖实现细节的工具。它的指导原则是，与软件使用方式相似的测试越多，它们提供的可信度越高。
      - Vue Test Utils。Vue Testing Library是 Vue Test Utils 的抽象。Vue Test Utils为用户提供了对 Vue 特定 API 的访问。
    - 端到端（E2E，end-to-end）测试。单元测试和组件测试在部署到生产环境时提供应用整体覆盖的能力是有限的。端到端测试验证应用中的所有层，不仅包括前端代码还包括所有相关的后端服务和基础设施。
      - Cypress.io。[Vue CLI 官方插件 - Cypress](https://cli.vuejs.org/core-plugins/e2e-cypress.html)
      - Nightwatch.js。[Vue CLI 官方插件 - Nightwatch](https://cli.vuejs.org/core-plugins/e2e-nightwatch.html)
      - Puppeteer
      - TestCafe

40. TypeScript支持

    - [Vue CLI](https://cli.vuejs.org) 提供了内建的 TypeScript 工具支持。

    - 基于类的Vue组件。[vue-class-component](https://github.com/vuejs/vue-class-component) vue-property-decorator

    - 标注返回值。render的返回值类型是VNode

      ```js
      import Vue, { VNode } from 'vue'
      
      const Component = Vue.extend({
        data () {
          return {
            msg: 'Hello'
          }
        },
        methods: {
          // 需要标注有 `this` 参与运算的返回值类型
          greet (): string {
            return this.msg + ' world'
          }
        },
        computed: {
          // 需要标注
          greeting(): string {
            return this.greet() + '!'
          }
        },
        // `createElement` 是可推导的，但是 `render` 需要返回值类型
        render (createElement): VNode {
          return createElement('div', this.greeting)
        }
      })
      ```

    - 标注Prop

      ```js
      import Vue, { PropType } from 'vue'
      
      interface ComplexMessage { 
        title: string,
        okMessage: string,
        cancelMessage: string
      }
      const Component = Vue.extend({
        props: {
          name: String,
          success: { type: String },
          callback: { 
            type: Function as PropType<() => void>
          },
          message: {
            type: Object as PropType<ComplexMessage>,
            required: true,
            validator (message: ComplexMessage) {
              return !!message.title;
            }
          }
        }
      })
      ```

41. 生产环境部署

    - 当使用webpack或Browserify类似的构建工具时，Vue源码会根据process.env.NODE_ENV决定是否启用生产环境模式，默认为开发环境模式。

    - 在webpack4+中，使用mode: 'production'启用生产环境模式

    - 在Rollup中，使用[@rollup/plugin-replace](https://github.com/rollup/plugins/tree/master/packages/replace)

      ```js
      const replace = require('@rollup/plugin-replace')
      rollup({
        // ...
        plugins: [
          replace({
            'process.env.NODE_ENV': JSON.stringify( 'production' )
          })
        ]
      }).then(...)
      ```

    - 当使用DOM内模板或JavaScript内的字符串模板时，模板会在运行时被编译为渲染函数

    - 提取组件的CSS。当使用单文件组件时，组件内的CSS会以<style> 标签的方式，通过JavaScript动态注入。这有一些小小的运行时开销，如果你使用服务端渲染，这会导致一段“无样式内容闪烁 (fouc)”。将所有组件的 CSS 提取到同一个文件可以避免这个问题，也会让 CSS 更好地进行压缩和缓存。[webpack + vue-loader](https://vue-loader.vuejs.org/zh-cn/configurations/extract-css.html) (`vue-cli` 的 webpack 模板已经预先配置好)；[Rollup + rollup-plugin-vue](https://vuejs.github.io/rollup-plugin-vue/#/en/2.3/?id=custom-handler)

    - 跟踪运行时错误。如果在组件渲染时出现运行错误，错误将会被传递至全局Vue.config.errorHandler配置函数中(如果已设置)。利用这个钩子函数来配合错误跟踪服务是个不错的主意。比如 [Sentry](https://sentry.io)，它为 Vue 提供了[官方集成](https://sentry.io/for/vue/)。

42. 服务端渲染

    - 构建Vue服务端渲染应用，参考[ssr.vuejs.org](https://ssr.vuejs.org/zh/)
    - Nuxt.js。[Nuxt.js](https://nuxtjs.org/)是一个基于Vue生态的更高层的框架，为开发服务端渲染的 Vue 应用提供了极其便利的开发体验。甚至可以用它来做为静态站生成器。
    - Quasar Framework SSR + PWA。[Quasar Framework](https://quasar.dev) 可以通过其一流的构建系统、合理的配置和开发者扩展性生成 (可选地和 PWA 互通的) SSR  应用，让你的想法的设计和构建变得轻而易举。你可以在服务端挑选执行超过上百款遵循“Material Design  2.0”的组件，并在浏览器端可用。你甚至可以管理网站的 `<meta>` 标签。Quasar 是一个基于 Node.js 和 webpack 的开发环境，它可以通过一套代码完成 SPA、PWA、SSR、Electron、Capacitor 和 Cordova 应用的快速开发。

43. 安全

    - 不论使用模板还是渲染函数，内容都会被自动转义，避免脚本注入。该转义通过诸如textContent的浏览器原生API完成
    - 动态attribute绑定也会自动被转义，避免通过闭合`title` attribute 而注入新的任意 HTML。该转义通过诸如 `setAttribute` 的浏览器原生的 API 完成。

44. 深入响应式原理

    ![vue响应式](https://cn.vuejs.org/images/data.png)

    - 当把一个普通的JavaScript对象传入Vue实例作为data选项，Vue将遍历此对象所有的property，并使用Object.defineProperty把这些property全部转为getter/setter。Object.defineProperty是ES5中一个无法shim（将一个新的API引入到一个旧的环境中，而且仅靠旧环境中已有的手段实现）的特性，这也就是Vue不支持IE8及以下版本浏览器的原因。

    - 这些getter/setter在内部让Vue能够追踪依赖，在property被访问和修改时通知变更。

    - 每个组件实例都对应一个watcher实例，它会在组件渲染的过程中把“接触”过的数据property记录为依赖。之后当依赖项的setter触发时，会通知watcher，从而使它关联的组件重新渲染。

    - Vue不能检测数组和对象的变化。

      - 对于对象。Vue无法检测property的添加或移除。由于 Vue 会在初始化实例时对 property 执行 getter/setter 转化，所以 property 必须在 `data` 对象上存在才能让 Vue 将它转换为响应式的。对于已经创建的实例，Vue 不允许动态添加根级别的响应式 property。但是，可以使用 `Vue.set(object, propertyName, value)` 方法向嵌套对象添加响应式 property。`vm.$set` 实例方法是全局 `Vue.set` 方法的别名。为已有对象赋值多个新 property时，可以使用this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })

      - 对于数组。以下情况，vue不能检测数组的变动：1.修改数组的长度；2.利用索引改变数组的某一项

        ```js
        var vm = new Vue({
          data: {
            items: ['a', 'b', 'c']
          }
        })
        vm.items[1] = 'x' // 不是响应性的
        vm.items.length = 2 // 不是响应性的
        
        // 以下可以在响应式系统内触发更新
        
        //改变数组某一项
        // Vue.set
        Vue.set(vm.items, indexOfItem, newValue)
        vm.$set(vm.items, indexOfItem, newValue)
        // Array.prototype.splice
        vm.items.splice(indexOfItem, 1, newValue)
        
        // 改变数组长度
        vm.items.splice(newLength)
        ```

    - 异步更新队列

      - Vue在更新DOM时是异步执行的。

      - 只要侦听到数据变化，Vue将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。

      - 如果同一个watcher被多次触发，只会被推入到队列中一次。

      - 这种在缓冲时去除重复数据对于避免不必要的计算和DOM操作是非常重要的。

      - 在下一个的事件循环“tick”中，Vue刷新队列并执行实际(已去重的)工作。

      - Vue在内部对异步队列使用原生的`Promise.then`、`MutationObserver` 和 `setImmediate`，如果执行环境不支持，则会采用 `setTimeout(fn, 0)` 代替。

      - 例如，当你设置 `vm.someData = 'new value'`，该组件不会立即重新渲染。当刷新队列时，组件会在下一个事件循环“tick”中更新。

      - 为了在数据变化之后等待 Vue 完成更新 DOM，可以在数据变化之后立即使用 `Vue.nextTick(callback)`。这样回调函数将在 DOM 更新完成后被调用。

        ```js
        Vue.component('example', {
          template: '<span>{{ message }}</span>',
          data: function () {
            return {
              message: '未更新'
            }
          },
          methods: {
            updateMessage: function () {
              this.message = '已更新'
              console.log(this.$el.textContent) // => '未更新'
              this.$nextTick(function () {
                console.log(this.$el.textContent) // => '已更新'
              })
            }
            // 或者使用async/await，因为$nextTick() 返回一个 Promise 对象
            updateMessage: async function () {
              this.message = '已更新'
              console.log(this.$el.textContent) // => '未更新'
              await this.$nextTick()
              console.log(this.$el.textContent) // => '已更新'
            }
          }
        })
        ```

45. 对比React

    - React 和 Vue 有许多相似之处，它们都有：
      - 使用 Virtual DOM
      - 提供了响应式 (Reactive) 和组件化 (Composable) 的视图组件。
      - 将注意力集中保持在核心库，而将其他功能如路由和全局状态管理交给相关的库。

    - 优化
      - 在React应用中，当某个组件的状态发生变化时，会以该组件为根，重新渲染整个组件子树。如要避免不必要的子组件的充渲染，需要在所有可能的地方使用PureComponent，或使用shouldComponentUpdate方法。同时你可能会需要使用不可变的数据结构来使得你的组件更容易被优化。使用 `PureComponent` 和 `shouldComponentUpdate` 时，需要保证该组件的整个子树的渲染输出都是由该组件的 props 所决定的。如果不符合这个情况，那么此类优化就会导致难以察觉的渲染结果不一致。
      - 在Vue应用中，组件的依赖是在渲染过程中自动追踪的，所以系统能精确知道哪个组件确实需要被重渲染。

    - JSX vs Templates
      - 使用JSX的渲染函数的优势：可以使用完整的编程语言 JavaScript 功能来构建你的视图页面。比如可以使用临时变量、JS 自带的流程控制、以及直接引用当前 JS 作用域中的值等等；开发工具对 JSX 的支持相比于现有可用的其他 Vue 模板还是比较先进的 (比如，linting、类型检查、编辑器的自动完成)。

    - 组件作用域内的css

      CSS作用域在React中是通过CSS-in-JS 的方案实现的 (比如 [styled-components](https://github.com/styled-components/styled-components) 和 [emotion](https://github.com/emotion-js/emotion))。

      许多主流的 CSS-in-JS 库也都支持 Vue (比如 [styled-components-vue](https://github.com/styled-components/vue-styled-components) 和 [vue-emotion](https://github.com/egoist/vue-emotion))。这里 React 和 Vue 主要的区别是，Vue 设置样式的默认方法是[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)里类似 `style` 的标签，scoped会自动添加一个唯一的 attribute (比如 `data-v-21e5b78`) 为组件内 CSS 指定作用域，编译的时候 `.list-container:hover` 会被编译成类似 .list-container[data-v-21e5b78]:hover

      Vue 的单文件组件里的样式设置是非常灵活的。通过 [vue-loader](https://github.com/vuejs/vue-loader)，你可以使用任意预处理器、后处理器，甚至深度集成 [CSS Modules](https://vue-loader.vuejs.org/en/features/css-modules.html)——全部都在 `<style>` 标签内。

    - 原生渲染

      React Native 能使你用相同的组件模型编写有本地渲染能力的 APP (iOS 和 Android)，能同时跨多平台开发。

      Weex （Weex 是阿里巴巴发起的跨平台用户界面开发框架）允许你使用 Vue 语法开发不仅仅可以运行在浏览器端，还能被用于开发 iOS 和 Android 上的原生应用的组件。另一个选择是 [NativeScript-Vue](https://nativescript-vue.org/)，一个用 Vue.js 构建完全原生应用的 [NativeScript](https://www.nativescript.org/) 插件。

    - [MobX](https://cn.mobx.js.org/)

46. Vuex

    Redux 事实上无法感知视图层，所以它能够轻松的通过一些[简单绑定](https://classic.yarnpkg.com/en/packages?q=redux vue&p=1)和 Vue 一起使用。Vuex 区别在于它是一个专门为 Vue 应用所设计。这使得它能够更好地和 Vue 进行整合，同时提供简洁的 API 和改善过的开发体验。

    Vuex采用集中式存储管理应用的所有组件的状态，以相应的规则保证状态以一种可预测的方式发生变化。

    ![vuex](https://vuex.vuejs.org/vuex.png)

    mapState函数返回的是一个对象

47. Vue CLI

48. Vue Loader

49. Vue Router