##### 1、关于el-selector组件

###### 动态获取options

场景：很多时候el-selector的options（选项)需要使用从后台获取的数据，当进行一些其它能够影响到数据的交互（增删改）时，对应的options使用的数据也应该进行更新，如果有多个交互都会影响到selector的下拉数据，那么在每个交互事件触发的回调方法中去进行selector数据的更新会变得非常繁琐，这时更好的做法是让用户去点击下拉框触发下拉菜单时实时对后台发起请求获取最新的数据



很容易我们可以想到通过触发focus（聚焦下拉框选中）来进行请求发起

```vue
<template>
	<el-selector @focus="updateSelectorOptions"></el-selector>
</template>
<script lang="js">
    export default {
        name:"demo",
        methods:{
            // 更新下拉框选项
            updateSelectorOptions(){
                
            }
        }
    }
</script>

```

理想情况下这应该足够满足我们的开发需求了，但这里存在一点小问题，我们会发现当聚焦下拉框后如我们所愿触发了请求更新到了数据，接着我们选中了其中一个选项，这时发现选择完后下拉框并没有失焦，那么再去点击下拉框想要更新数据发现不会再次触发聚焦事件了。显然这不是我们需要的结果



结论：我们可以在selector上挂载click事件来代替focus，只要点击到下拉框，必定触发更新，需要注意的是需要加上.native事件修饰符

>  .native修饰符在组件的根元素绑定一个原生事件

```vue
<template>
	<el-selector @click.native="updateSelectorOptions"></el-selector>
</template>
<script lang="js">
    export default {
        name:"demo",
        methods:{
            // 更新下拉框选项
            updateSelectorOptions(){
                
            }
        }
    }
</script>

```

为什么这里需要用.native修饰符?

当在父组件上直接用v-on绑定事件，这时绑定的是Vue的自定义事件，在Vue的自定义事件运行机制中，事件的回调以及触发都需要由开发者自己设置

假设有一个子组件如下:

```vue
<template>
  <button>按钮</button>
</template>

<script>
export default {
    name: "child-component"
}
</script>
```

它的父组件中使用到该子组件

```vue
<template>
  <child-component @click="handleClick"></child-component>
</template>

<script>
import ChildComponent from "@/components/ChildComponent.vue"
export default {
    name: "parent-component",
    components:{
        ChildComponent
    },
    methods:{
        handleClick(){
            console.log("触发点击")
        }
    }
}
</script>
```

这个时候去点击按钮，不会出现我们预想的打印”触发点击“，因为实际上这个click事件是一个Vue自定义事件，我们并没有触发它

如果我们想要触发这个自定义事件
```vue
<template>
  <button @click="$emit('click')">按钮</button>
</template>

<script>
export default {
    name: "child-component"
}
</script>
```

如果子组件如下
```vue
<template>
  <button @click="onClick">按钮</button>
</template>

<script>
export default {
    name: "child-component",
    props:{
        onClick:{
            
        }
    }
}
</script>
```

这时点击按钮发现竟然打印出了"触发点击"，这是为什么？明明我们只是在子组件上的原生button元素绑定了一个click事件，并且回调函数是一个从父组件传递下来的props

**原因**：props绑定在原生元素的click事件上，Vue识别到这是一个原生button，于是这个click事件就是原生的click事件而不是Vue自定义事件，onClick并当成了一个函数，并且点击时触发了事件的冒泡机制，于是父元素的自定义click事件也被执行。

事实上，我们**正确的做法**应当是直接在自定义组件上给@click增加一个.native修饰符来告诉Vue我们想要一个真正的原生click事件而不是Vue自定义事件，这时会在自定义组件的根元素，也就是button元素上绑定一个原生的click事件，像这样

```vue
<template>
  <child-component @click.native="handleClick"></child-component>
</template>

<script>
import ChildComponent from "@/components/ChildComponent.vue"
export default {
    name: "parent-component",
    components:{
        ChildComponent
    },
    methods:{
        handleClick(){
            console.log("触发点击")
        }
    }
}
</script>
```

###### .native修饰符

如果绑定了一个dom中不存

在的原生事件并且加上了.native修饰符，这时就算使用$emit在子组件中去触发该事件也是行不通的，因为加了.native修饰符就已经把这个事件绑定给了根元素上了，根本就不会通过events，也就不会被触发回调了

父组件：

```vue
<template>
  <child-component @ssffff.native="handleClick"></child-component>
</template>

<script>
import ChildComponent from "@/components/ChildComponent.vue"
export default {
    name: "parent-component",
    components:{
        ChildComponent
    },
    methods:{
        handleClick(){
            console.log("触发点击")
        }
    }
}
</script>
```
子组件
```vue
<template>
  <button @click="$emit('ssffff')">按钮</button>
</template>

<script>
export default {
    name: "child-component",
}
</script>
```

此时不会触发父组件的回调函数handleclick。

##### 2、关于自定义组件使用v-model

场景如下：

现在有一个子组件Children和一个父组件Parent,Children中有一个input元素，需要在父组件中v-model双向绑定这个input元素的值

Parent组件：

```vue
<template>
	<Children v-model="text"></Children>
</template>

<script>
import Children from "@/components/Children.vue"
export default {
    name: "Parent",
    components:{
      Children
    },
	data () {
    return {
      text: 'Hello world'
    }
  }
}
</script>
```

Children组件：

```vue
<template>
	<input type="text" :value="text" input="$emit('input', $event.target.value)">
</template>

<script>
export default {
  props: {
    text: String
  }
}
</script>
```

这样就实现了我们的需求

###### v-model原理

v-model实际上是v-on和v-bind的组合语法糖，所以实际上父组件等同于：
```vue
<template>
	<Children :text="text" @input="text=$event"></Children>
</template>

<script>
import Children from "@/components/Children.vue"
export default {
    name: "Parent",
    components:{
      Children
    },
	data () {
    return {
      text: 'Hello world'
    }
  }
}
</script>
```

父组件上的input事件实际上是一个自定义事件，不管叫什么名字都是一样的，真正的input事件在子组件Children原生input元素中的@input，当触发input事件(输入框中内容变更)，输入框将值通过events传递给Children，Children执行自定义事件回调将父组件Parent的text变量赋值为最新的输入框值，Children组件上绑定的text是一个响应式变量，检测到变更时又将最新的值向下传递给props，从而实现了双向绑定

##### 3、关于el-dialog组件

###### el-dialog插槽写法造成的组件臃肿问题

场景：现在有一个Children组件，该组件中嵌套了一个el-dialog组件，el-dialog组件嵌套了一个el-input组件，el-input输入框的双向绑定变量在Children组件的data中定义，父组件Parent中的data定义了一个visibled变量通过props下发给Children组件中dialog来控制弹窗的显示和隐藏

Parent组件

```vue
<template>
	<Children :visible="visibled" @close-dialog="visibled=$event">
    </Children>
	<el-button @click="visibled=true"></el-button>
</template>

<script>
import Children from "@/components/Children.vue"
export default {
    name: "Parent",
    components:{
      Children
    },
	data () {
    return {
      visibled: false
    }
  }
}
</script>
```

Children组件
```vue
<template>
	<el-dialog :visibled="visibled">
        <el-input v-model="inputValue"></el-input>
        <span slot="footer" class="dialog-footer">
      		<el-button @click="closeDialog">取 消</el-button>
      		<el-button type="primary" @click="closeDialog">确 定</el-button>
    	</span>
    </el-dialog>
</template>

<script>
export default {
    name: "Parent",
    props:{
        visibled:{
            type:Boolean,
            default:false
        }
    },
	data () {
    	return {
      		inputValue: ""
    	}
  	},
    methods:{
        closeDialog(){
            this.$emit('close-dialog',false)
        }
    }
}
</script>
```

显然，当点击Parent组件中的button按钮时，Children组件中的dialog弹窗显示，当点击弹窗中的取消或确定时，Children组件通过自定义事件将Parent组件用来控制弹窗显隐的visibled变量设置为false并触发更新，visibled变量通过v-bind下发给Children组件，此时Children中的Dialog被隐藏。



###### 解决el-dialog插槽写法臃肿问题后造成的bug

这满足了我们想将el-dialog弹窗中一大堆嵌套元素以及元素上绑定的一些data和methods都单独放在了一个自定义组件中，不容易造成Parent组件的代码臃肿的需求，但也带来了其它问题，我们只控制了弹窗的显示隐藏，el-dialog组件内部通过下发的visibled属性只是做了一个v-show的控制，那么隐藏后Children的整个实例依旧存在，所有弹窗内部的元素所绑定的事件和值的不会消失，就像上面dialog中嵌套的input一样，我们的本意是点击取消后，清空input的绑定值，当用户重新点开弹窗，看到的是一个被初始化过的弹窗而不是保留历史数据。

做法一、在取消、确定这两个事件的回调函数中去将Children组件中需要初始化的变量重置掉

很显然，这会增加很多我们的开发量，因此不考虑这种做法

做法二、在Children组件上绑定一个v-if，用来控制Children实例的销毁和创建

Parent组件

```vue
<template>
	<Children :visible="visibled" @close-dialog="visibled=$event" v-if="visibled">
    </Children>
	<el-button @click="visibled=true"></el-button>
</template>

<script>
import Children from "@/components/Children.vue"
export default {
    name: "Parent",
    components:{
      Children
    },
	data () {
    return {
      visibled: false
    }
  }
}
</script>
```

我们发现，大部分情况下弹窗关闭后Children中所有元素关联的数据被清空，但当弹窗再次打开时，存在弹窗闪动以及Children组件部分生命周期钩子不会重新执行的问题，这是因为Vue的diff算法做了一些优化，当元素被移除/修改时，只更新有变更的内容，实际上这是diff算法所做的性能优化，但不符合我们的需求，有时候会造成我们开发上的一些麻烦

解决方案：给Children组件加上一个唯一的key

```vue
<template>
	<Children :visible="visibled" @close-dialog="visibled=$event" v-if="visibled" key="ocsa214sf124">
    </Children>
	<el-button @click="visibled=true"></el-button>
</template>

<script>
import Children from "@/components/Children.vue"
export default {
    name: "Parent",
    components:{
      Children
    },
	data () {
    return {
      visibled: false
    }
  }
}
</script>
```

原理：参考https://v3.cn.vuejs.org/api/special-attributes.html#key



事实上，我们对于el-dialog有更好的使用方式，我们可以纯粹地用js来调用它而不是依靠模板

###### js调用el-dialog组件（todo）

这应该是更好的一种使用el-dialog的方式，当需要使用到弹窗时，我们通过创建一个新dialog实例的方式，并将嵌套组件以及相关的props全部注入到dialog实例对象中，这样就避免了在模板中写一大堆的dialog嵌套元素以及需要每次用变量控制弹窗显隐的一系列麻烦



##### 4、Vue.$set(todo)

应用场景：现在有一个关于省市区的树状结构数据，我们需要从后台拿到这个数据用于渲染有层级关系的复选框组，并且在用户提交表单后获取到这些复选框中被勾选的数据对象

```vue
<template>
	
</template>
```



##### 5、父子孙组件通信方式

###### 5.1、父子组件通信

这一种情况的通信是最简单的，父组件通过在子组件中绑定v-bind下发到子组件的props，子组件接受props获取到从父组件中的数据

父组件Parent

```vue
<template>
	<Children :parentProps="value" :click-handler="handleClick">
    </Children>
</template>

<script>
import Children from "@/components/Children.vue"
export default {
    name: "Parent",
    components:{
      Children
    },
	data () {
      return {
        value: "我是在父组件中的数据"
      }
    },
    methods:{
        handleClick(){
            console.log("xxxxx")
        }
    }
}
</script>
```

子组件Children

```vue
<template>
	<span @click="clickHandler">{{parentProps}}</span>
</template>

<script>
export default {
    name: "Children",
    props: {
        parentProps:{
            type:String
        },
        clickHandler:{
            type:Function
        }
    }
  }
}
</script>
```



点击"我是在父组件中的数据"打印出"xxxxx"说明父传子成功

这里需要注意的是如果子组件的数据是从父组件中传递下来的，但在子组件会触发一些改变props数据的操作，那么需要在子组件的data中定义一个新的变量，然后再将props的数据赋给data，在子组件中去使用data的值，比较典型的就是子组件中有一个input元素，绑定了一个v-model并且绑定的值是从父组件中得到的

```vue
<template>
	<el-input v-model="parentProps"></el-input>
</template>

<script>
export default {
    name: "Children",
    props: {
        parentProps:{
            type:String
        }
    }
  }
}
</script>
```



此时在输入框中输入任意值，会报Vue警告，这是因为Vue的数据流向是单向的，也就是说只接受从父组件改变数据然后流向子组件，不允许在子组件中去改变从父组件中传递到子组件的数据，这是为了避免出现父组件中多个子组件依赖到了这个数据，当子组件改变从父组件中传递到子组件的数据时，父组件中的多个子组件受其影响并无法定位发生数据改变的子组件的情况

正确的做法如下：
```vue
<template>
	<el-input v-model="value"></el-input>
</template>

<script>
export default {
    name: "Children",
    props: {
        parentProps:{
            type:String
        }
    },
    data(){
        return{
            value:this.parentProps
        }
    }
  }
}
</script>
```

> props是在data之前被创建完成的，因此在data中可以用this取到props的属性

假如我们需要改变父组件的值，需要用自定义事件来完成子传父的通信

子组件

```vue
<template>
	<el-input v-model="value" @input="transValue(value)"></el-input>
</template>

<script>
export default {
    name: "Children",
    props: {
        parentProps:{
            type:String
        }
    },
    data(){
        return{
            value:this.parentProps
        }
    },
    methods:{
        transValue(val){
          this.$emit('getValue',val)
        }
    }
  }
}
</script>
```

父组件Parent

```vue
<template>
	<Children :parentProps="value" @get-value="setValue"/>
</template>

<script>
import Children from "@/components/Children.vue"
export default {
    name: "Parent",
    components:{
      Children
    },
	data () {
      return {
        value: "我是在父组件中的数据"
      }
    },
    methods:{
        setValue(val){
            // 可以在父组件的这个方法中做任何操作，当子组件中的input事件被触发			 时就会执行这个父组件的回调函数
            this.value = val
        }
    }
}
</script>
```

###### 5.2、兄弟组件通信



