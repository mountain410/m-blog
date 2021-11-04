---
layout:     post
title:      "一些不为人知的Element UI踩坑指南"
date:       2021-11-3 20:30:00
author:     "Mountain"
header-img: "img/post-bg-2015.jpg"
tags:
    - 组件库
---


#### 1、form 下面只有一个 input 时回车键刷新页面

原因是触发了表单默认的提交行为，给el-form 加上@submit.native.prevent就行了。

```html
<el-form inline @submit.native.prevent>
  <el-form-item label="订单号">
    <el-input
      v-model="query.orderNo"
      :placeholder="输入订单号查询"
      clearable
      @keyup.enter.native="enterInput"
    />
  </el-form-item>
</el-form>
复制代码
```

#### 2、表格固定列最后一行显示不全

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/933c262bbab94d07a303123111e6acbf~tplv-k3u1fbpfcp-watermark.awebp)

这种情况有时在宽度刚好处于临界值状态时会出现。因为固定列是独立于表格body动态计算高度的，出现了固定列高度小于表格高度所以造成最后一行被遮挡。

```css
// 设置全局
.el-table__fixed-right {
  height: 100% !important;
}
复制代码
```

#### 3、气泡确认框文档里的confirm事件不生效

ElementUI 2.14.0 版本事件名称改为 confirm 和 cancel，如果你的版本低于 2.14.0，记得使用onConfirm、onCancel。

```html
// 将confirm改为onConfirm
<el-popover @onConfirm="">
</el-popover>
复制代码
```

#### 4、输入框用正则限制但绑定值未更新

有这么一段代码：

```html
<el-input 
  v-model="form.retailMinOrder" 
  placeholder="请输入" 
  @keyup.native="form.retailMinOrder=form.retailMinOrder.replace(/[^\d.]/g,'')"
/>
复制代码
```

这样做虽然输入框的显示是正确的，但绑定的值是没有更新的，将 onkeyup 改为 oninput 即可。

#### 5、去除type="number"输入框聚焦时的上下箭头

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4e7170d6f974b2a997505d28b4a57f9~tplv-k3u1fbpfcp-watermark.awebp)

```css
/* 设置全局 */
.clear-number-input.el-input::-webkit-outer-spin-button,
.clear-number-input.el-input::-webkit-inner-spin-button {
  margin: 0;
  -webkit-appearance: none !important;
} 
.clear-number-input.el-input input[type="number"]::-webkit-outer-spin-button,
.clear-number-input.el-input input[type="number"]::-webkit-inner-spin-button {
  margin: 0;
  -webkit-appearance: none !important;
}
.clear-number-input.el-input {
  -moz-appearance: textfield;
} 
.clear-number-input.el-input input[type="number"] {
  -moz-appearance: textfield;
}
复制代码
<el-input type="number" class="clear-number-input" />
复制代码
```

#### 6、只校验表单其中一个字段

在一些用户注册场景中，提交整个表单前有时候我们会做一些单独字段的校验，例如发送手机验证码，发送时我们只需要校验手机号码这个字段，可以这样做：

```javascript
this.$refs['form'].validateField('mobile', valid => {
  if (valid) {
    // 发送验证码
  }
})
复制代码
```

如果需要多个参数，将参数改为数组形式即可。

#### 7、弹窗重新打开时表单上次的校验信息未清除

有人会在open时在$nextTick里重置表单，而我选择在关闭时进行重置。

```javascript
<el-dialog @close="onClose">
  <el-form ref="form">
  </el-form>
</el-dialog>

// 弹窗关闭时重置表单
onClose() {
  this.$refs['form'].resetFields()
}
复制代码
```

#### 8、表头与内容错位

网上也有其他一些办法，但我记得对我没什么作用，后来我是用下面这个办法：

```css
// 全局设置
.el-table--scrollable-y .el-table__body-wrapper {
 overflow-y: overlay !important;
}
复制代码
```

#### 9、表单多级数据结构校验问题

```html
<el-form :model="form">
  <el-form-item label="部门" prop="dept"></el-form-item>
  <el-form-item label="姓名" prop="user.name"></el-form-item>
</el-form>
复制代码
rules: {
  'user.name': [{ required: true, message: '姓名不能为空', trigger: 'blur' }]
}
复制代码
```

#### 10、表格跨分页多选

看到项目里有小伙伴手动添加代码去处理这个问题，其实根据文档，只需加上row-key和reserve-selection即可。

```html
<el-table row-key="id">
  <el-table-column type="selection" reserve-selection></el-table-column>
</el-table>
复制代码
```

#### 11、根据条件高亮行并去除默认hover颜色

```javascript
<el-table :row-class-name="tableRowClassName">
</el-table>

tableRowClassName({ row }) {
  return row.status === 2 ? 'highlight' : ''
}

// 设置全局
.el-table .highlight {
  background-color: #b6e8fe;
  &:hover > td {
    background-color: initial !important;
  }
  td {
    background-color: initial !important;
  }
}
复制代码
```

#### 12、表单不想显示label但又想显示必填星号怎么办

```html
// label给个空格即可
<el-form>
  <el-table>
    <el-table-column label="名称">
      <template>
        <el-form-item label=" ">
           <el-input placeholder="名称不能为空" />
        </el-form-item>
      </template>
    </el-table-column>
  </el-table>
</el-form>
复制代码
```

#### 13、table 内嵌 input 调用 focus 方法无效

```html
<el-table>
  <el-table-column label="名称">
    <template>
      <el-input ref="inputRef" />
    </template>
  </el-table-column>
</el-table>

// 无效
this.$refs['inputRef'].focus()
this.$refs['inputRef'][0].focus()
this.$refs['inputRef'].$el.children[0].focus()

// 有效
<el-input id="inputRef" />
document.getElementById('inputRef').focus()
复制代码
```

#### 14、表格内容超出省略

看到有小伙伴在代码里自己手动去添加CSS来实现，害，又是一个不看文档的反面例子，其实只要加个show-overflow-tooltip就可以了，还自带tooltip效果，不香吗？

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1210a5c4e2bb41c1bb70acf0bc035111~tplv-k3u1fbpfcp-watermark.awebp)

```html
<el-table-column label="客户名称" prop="customerName" show-overflow-tooltip>
</el-table-column>
复制代码
```

#### 15、el-tree 展开/收起所有节点

```javascript
<el-tree ref="tree"></el-tree>

expandTree(expand = true) {
  const nodes = this.$refs['tree'].store._getAllNodes()
  nodes.forEach(node => {
    node.expanded = expand
  })
}
复制代码
```

#### 16、el-popover 位置偏移问题

事情起因：el-popover 里的内容是动态获取的，所以刚打开时位置正确，此时内容为空，等到获取数据渲染后 el-popover 内容盒子大小发生变化从而造成位置偏移。

解决办法：通过源码知道 el-popover 里有个 updatePopper 方法用于更新位置（文档里没有），所以我们只需在获取数据后重新 updatePopper 就可以了。

```html
<el-popover ref="popover" placement="left" trigger="click">
</el-popover>
复制代码
// 获取数据后
this.$nextTick(() => {
  this.$refs['popover'].updatePopper()
})
复制代码
```

#### 17、el-dialog 的 destroy-on-close 属性设置无效

destroy-on-close 设置为 true 后发现弹窗关闭后 DOM 元素仍在，没有被销毁。

解决办法：在 el-dialog 上添加 v-if。

```html
<el-dialog :visible.sync="visible" v-if="visible" destroy-on-close>
</el-dialog>
复制代码
```

#### 18、el-cascader 选择后需要点击空白处才能关闭

级联选择器在设置为可选任意一级时，选定某个选项时需要手动点击空白处才能关闭。

解决办法：可在 change 事件触发时将其关闭。

```html
<el-cascader
  ref="cascader"
  @change="onChange"
/>
复制代码
onChange() {
  this.$refs['cascader'].dropDownVisible = false
}
复制代码
```

#### 19、使用图片查看器

el-image 组件是自带图片预览功能的，传入 preview-src-list 即可。但有时候我们不用 image 组件但又想预览大图怎么办？例如点击一个按钮时弹出一个图片查看器？

答案是使用 el-image-viewer，文档里并没有这个组件，但通过查看源码知道，该组件正是 el-image 里预览图片所用的。

```javascript
<template>
  <div>
    <el-button @click="open">打开图片预览</el-button>
    <el-image-viewer
      v-if="show"
      :on-close="onClose"
      :url-list="urls"
      :initial-index="initialIndex"
    />
  </div>
</template>

<script>
import ElImageViewer from 'element-ui/packages/image/src/image-viewer'

export default {
  components: {
    ElImageViewer
  },

  data() {
    return {
      show: false,
      urls: ['https://img0.baidu.com/it/u=391928341,1475664833&fm=26&fmt=auto&gp=0.jpg'],
      initialIndex: 0
    }
  },

  methods: {
    open() {
      this.show = true
    },

    onClose() {
      this.show = false
    }
  }
}
</script>
复制代码
```

#### 20、el-table-column添加v-if后数据位置错乱

解决：给 el-table-column 添加key区分

```
<el-table-column v-if="type === 'edit'" key="1"></el-table-column>
<el-table-column v-if="type === 'edit'" key="2"></el-table-column>
复制代码
```

> 未完待续，持续更新。。。



转自：https://juejin.cn/post/6981083988286767117#heading-10