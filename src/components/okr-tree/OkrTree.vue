<template>
  <div class="org-chart-container">
    <div ref="orgChartRoot" class="org-chart-node-children"
      :class="{
        'vertical': direction === 'vertical',
        'horizontal': direction === 'horizontal',
        'show-collapsable': showCollapsable,
        'one-branch': data.length === 1
      }">
      <OkrTreeNode 
        v-for="child in root.childNodes"
        :node="child"
        :show-collapsable="showCollapsable"
        :label-width="labelWidth"
        :label-height="labelHeight"
        :renderContent="renderContent"
        :label-class-name="labelClassName"
        :selected-key="selectedKey"
        :default-expand-all="defaultExpandAll"
        :node-key="nodeKey"
        :key="getNodeKey(child)"
        :props="props"
      ></OkrTreeNode>
    </div>
  </div>
</template>
<script>
import Vue from 'vue'
import OkrTreeNode from './OkrTreeNode.vue';
import TreeStore from './model/tree-store.js';
import { getNodeKey } from './model/util';
export default {
  name: 'OkrTree',
  components: {
    OkrTreeNode
  },
  provide () {
    return {
      okrEventBus: this.okrEventBus
    }
  },
  props: {
    data: { // 源数据
      required: true,
    },
    // 方向
    direction: {
      type: String,
      default: 'vertical'
    },
    // 子节点是否可折叠
    showCollapsable: {
      type: Boolean,
      default: false
    },
    // 树节点的内容区的渲染 Function
    renderContent: Function,
    // 树节点区域的宽度
    labelWidth: [String, Number],
    // 树节点区域的高度
    labelHeight: [String, Number],
    // 树节点的样式
    labelClassName: [Function, String],
    // 用来控制选择节点的字段名
    selectedKey: String,
    // 是否默认展开所有节点
    defaultExpandAll: {
      type: Boolean,
      default: false
    },
    // 当前选中的节点
    currentNodeKey: [String, Number],
    // 每个树节点用来作为唯一标识的属性，整棵树应该是唯一的
    nodeKey: String,
    defaultExpandedKeys: {
      type: Array
    },
    props: {
      default() {
        return {
          children: 'children',
          label: 'label',
          disabled: 'disabled'
        };
      }
    },
  },
  computed : {
    ondeClass () {
      return {
        findNode: null
      }
    }
  },
  data () {
    return {
      okrEventBus: new Vue(),
      store: null,
      root: null
    }
  },
  created () {
    this.isTree = true;
    this.store = new TreeStore({
      key: this.nodeKey,
      data: this.data,
      props: this.props,
      defaultExpandedKeys: this.defaultExpandedKeys,
      showCollapsable: this.showCollapsable,
      currentNodeKey: this.currentNodeKey,
      defaultExpandAll: this.defaultExpandAll,
      filterNodeMethod: this.filterNodeMethod
    })
    this.root = this.store.root;
    console.log(this.store)
    console.log(this.root)
  },
  watch: {
    defaultExpandedKeys(newVal) {
      this.store.defaultExpandedKeys = newVal;
      this.store.setDefaultExpandedKeys(newVal);
    }
  },
  methods: {
    getNodeKey(node) {
      console.log(getNodeKey(this.nodeKey, node.data))
      return getNodeKey(this.nodeKey, node.data);
    },
    // 内置：根据 key 找到对应的节点
    __findNode (children, nodeId) {
      for (let i = 0; i < children.length; i++) {
        let item = children[i]
        if (nodeId === item.node[this.nodeKey]) {
          return item
        }
        if (item.$children) {
          let findNode = this.__findNode(item.$children, nodeId)
          if (findNode) {
            return findNode
          }
        }
      }
    },
    // 公开，根据 ID 获取对应 node 节点
    getNode (nodeId) {
      return this.__findNode(this.$children, nodeId)
    },
  }
}
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
.org-chart-container{
  display: block;
  width: 100%;
  text-align: center;
}

.vertical .org-chart-node-children {
  position: relative;
  padding-top:20px;
  transition: all 0.5s;
}
.vertical .org-chart-node{
  float: left;
  text-align: center;
  list-style-type: none;
  position: relative;
  padding: 20px 5px 0 5px;
  transition: all .5s;
}
/*使用 ::before 和 ::after 绘制连接器*/
.vertical .org-chart-node::before,
.vertical .org-chart-node::after {
  content: '';
  position: absolute;
  top: 0;
  right: 50%;
  width: 50%;
  border-top: 1px solid #ccc;
  height: 20px;
}
.vertical .org-chart-node::after{
  right: auto;
  left: 50%;
  border-left: 1px solid #ccc;
}
/*我们需要从没有任何兄弟元素的元素中删除左右连接器*/
.vertical.one-branch > .org-chart-node::after,
.vertical.one-branch > .org-chart-node::before{
  display: none;
}
/*从单个子节点的顶部移除空格*/
.vertical.one-branch >.org-chart-node{
  padding-top: 0;
}
/*从第一个子节点移除左连接器，从最后一个子节点移除右连接器*/
.vertical .org-chart-node:first-child::before,
.vertical .org-chart-node:last-child::after{
  border: 0 none;
}
/*将垂直连接器添加回最后的节点*/
.vertical .org-chart-node:last-child::before{
  border-right: 1px solid #ccc;
  border-radius: 0 5px 0 0;
}
.vertical .org-chart-node:only-child:before{
  border-radius: 0 0px 0 0;
  margin-right: -1px;
}
.vertical .org-chart-node:first-child::after{
  border-radius: 5px 0 0 0;
}
.vertical .org-chart-node.is-leaf{
  padding-top: 20px;
  padding-bottom: 20px;
}
.vertical .org-chart-node.is-leaf::before{
  content: '';
  display: block;
  height: 20px;
}
.vertical .org-chart-node.is-leaf .org-chart-node-label::after{
  display: none;
}


/*从父节点添加向下的连接器了*/
.vertical .org-chart-node-children::before{
  content: '';
  position: absolute;
  top:0;
  left: 50%;
  border-left: 1px solid #ccc;
  width: 0;
  height: 20px;
}
.vertical .org-chart-node-label{
  position: relative;
  display: inline-block;
}
.vertical .org-chart-node-label .org-chart-node-label-inner{
  box-shadow: 0 2px 5px 0 rgba(0,0,0,0.16), 0 2px 10px 0 rgba(0,0,0,0.12);
  display: inline-block;
  padding: 10px;
  margin: 0px;
  font-size: 16px;
  word-break: break-all;
}
.vertical .org-chart-node-label .org-chart-node-label-inner:hover{
  background: #eeeeee;
}
.vertical .org-chart-node-label .org-chart-node-btn{
  position: absolute;
  top: 100%;
  left: 50%;
  width: 20px;
  height: 20px;
  z-index: 10;
  margin-left: -11px;
  margin-top: 9px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 50%;
  box-shadow: 0 0 2px rgba(0, 0, 0, .15);
  cursor: pointer;
  transition: all .35s ease;
}
.vertical .org-chart-node-label .org-chart-node-btn:hover{
  background-color: #e7e8e9;
  transform: scale(1.15);
}
.vertical .org-chart-node-label .org-chart-node-btn::before,
.vertical .org-chart-node-label .org-chart-node-btn::after {
  content: '';
  position: absolute;
}
.vertical .org-chart-node-label .org-chart-node-btn::before{
  top: 50%;
  left: 4px;
  right: 4px;
  height: 0;
  border-top: 1px solid #ccc;
}
.vertical .org-chart-node-label .org-chart-node-btn::after{
  top: 4px;
  left: 50%;
  bottom: 4px;
  width: 0;
  border-left: 1px solid #ccc;
}
.vertical .org-chart-node-label .expanded.org-chart-node-btn::after{
  display: none;
}

.vertical .org-chart-node.collapsed .org-chart-node-label{
  position: relative;
}
.vertical .org-chart-node.collapsed .org-chart-node-label::after{
  content: '';
  position: absolute;
  top: 100%;
  left: 0;
  width: 50%;
  height: 20px;
  border-right: 1px solid #ddd;
}



.horizontal .org-chart-node-children{
  position: relative;
  padding-left:20px;
  transition: all 0.5s;
}
.horizontal .org-chart-node{
  display: flex;
  align-items: center;
  position: relative;
  padding: 0px 5px 0 20px;
  transition: all .5s;
}

/*使用 ::before 和 ::after 绘制连接器*/
.horizontal .org-chart-node::before,
.horizontal .org-chart-node::after{
  content: '';
  position: absolute;
  border-left: 1px solid #ccc;
  top:0;
  left: 0;
  width: 20px;
  height: 50%;
}
.horizontal .org-chart-node::after{
  top: 50%;
  border-top: 1px solid #ccc;
}

/*我们需要从没有任何兄弟元素的元素中删除左右连接器*/
.horizontal.one-branch > .org-chart-node::after,
.horizontal.one-branch > .org-chart-node::before{
  display: none;
}
/*从单个子节点的顶部移除空格*/
.horizontal.one-branch >  .org-chart-node{
  padding-left: 0;
}

/*从第一个子节点移除左连接器，从最后一个子节点移除右连接器*/
.horizontal .org-chart-node:first-child::before,
.horizontal .org-chart-node:last-child::after{
  border: 0 none;
}
/*将垂直连接器添加回最后的节点*/
.horizontal .org-chart-node:last-child::before{
  border-bottom: 1px solid #ccc;
  border-radius: 0 0px 0 5px;
}
.horizontal .org-chart-node:only-child::before{
  border-radius: 0 0px 0 0px;
}

.horizontal .org-chart-node:first-child::after{
  border-radius: 5px 0px 0 0;
}
.horizontal .org-chart-node.is-leaf{
  position: relative;
  padding-left: 20px;
  padding-right: 20px;
}
.horizontal .org-chart-node.is-leaf::before{
  content: '';
  display: block;
}
.horizontal .org-chart-node.is-leaf .org-chart-node-label::after{
  display: none;
}


/*从父节点添加向下的连接器了*/
.horizontal .org-chart-node-children::before{
  content: '';
  position: absolute;
  left:0;
  top: 50%;
  border-top: 1px solid #ccc;
  width: 20px;
  height: 0;
}
.horizontal .org-chart-node-label{
  position: relative;
  display: inline-block;
}

.horizontal .org-chart-node-label .org-chart-node-label-inner{
  box-shadow: 0 2px 5px 0 rgba(0,0,0,0.16), 0 2px 10px 0 rgba(0,0,0,0.12);
  display: inline-block;
  padding: 10px;
  margin: 10px 0;
  font-size: 16px;
  word-break: break-all;
}

.horizontal .org-chart-node-label .org-chart-node-label-inner:hover{
  background: #eeeeee;
}
.horizontal .org-chart-node-label .org-chart-node-btn{
  position: absolute;
  left: 100%;
  top: 50%;
  width: 20px;
  height: 20px;
  z-index: 10;
  margin-top: -11px;
  margin-left: 9px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 50%;
  box-shadow: 0 0 2px rgba(0, 0, 0, .15);
  cursor: pointer;
  transition: all .35s ease;
}

.horizontal .org-chart-node-label .org-chart-node-btn:hover{
  background-color: #e7e8e9;
  transform: scale(1.15);
}
.horizontal .org-chart-node-label .org-chart-node-btn::before,
.horizontal .org-chart-node-label .org-chart-node-btn::after {
  content: '';
  position: absolute;
}

.horizontal .org-chart-node-label .org-chart-node-btn::before{
  top: 50%;
  left: 4px;
  right: 3px;
  border-top: 1px solid #ccc;
  height: 0;
  transform: translateY(-50%);
}
.horizontal .org-chart-node-label .org-chart-node-btn::after{
  top: 4px;
  left: 50%;
  bottom: 4px;
  width: 0;
  border-left: 1px solid #ccc;
}
.horizontal .org-chart-node-label .expanded.org-chart-node-btn::after{
  display: none;
}

.horizontal .org-chart-node.collapsed .org-chart-node-label{
  position: relative;
}
.horizontal .org-chart-node.collapsed .org-chart-node-label::after{
  content: '';
  border-bottom: 1px solid #ccc;
  position: absolute;
  top:0;
  left: 100%;
  height: 50%;
  width: 20px;
}

</style>