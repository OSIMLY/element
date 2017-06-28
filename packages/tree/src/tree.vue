<template>
  <!--组件根元素，设定当前节点高亮属性-->
  <div class="el-tree" :class="{ 'el-tree--highlight-current': highlightCurrent }">
    <!--引入树节点组件-->
    <el-tree-node
      v-for="child in root.childNodes"
      :node="child"
      :props="props"
      :key="getNodeKey(child)"
      :render-content="renderContent"
      @node-expand="handleNodeExpand">
    </el-tree-node>
    <!--用于显示空树占位符-->
    <div class="el-tree__empty-block" v-if="!root.childNodes || root.childNodes.length === 0">
      <span class="el-tree__empty-text">{{ emptyText }}</span>
    </div>
  </div>
</template>

<script>
  // 引入状态管理模块
  import TreeStore from './model/tree-store';
  // 引入语言模块
  import {t} from 'element-ui/src/locale';
  // 引入全局事件管理模块
  import emitter from 'element-ui/src/mixins/emitter';
  // 默认导出对象
  export default {
    // 组件名称
    name: 'ElTree',
    // 混入全局事件管理模块
    mixins: [emitter],
    // 引入树节点子组件
    components: {
      // 引入并赋值，区别于 import 方式
      ElTreeNode: require('./tree-node.vue')
    },
    // 定义页面数据
    data() {
      return {
        // 数据仓库
        store: null,
        // 根节点
        root: null,
        // 当前节点
        currentNode: null
      };
    },
    // 定义属性
    props: {
      // 数据源
      data: {
        type: Array
      },
      // 空占位符文本
      emptyText: {
        type: String,
        default() {
          // 返回渲染结果
          return t('el.tree.emptyText');
        }
      },
      // 节点的键名
      nodeKey: String,
      // 含复选框时是否父子解除关联
      checkStrictly: Boolean,
      // 默认展开子节点
      defaultExpandAll: Boolean,
      // 点击时展开子节点
      expandOnClickNode: {
        type: Boolean,
        default: true
      },
      // 展开子节点时是否展开父节点
      autoExpandParent: {
        type: Boolean,
        default: true
      },
      // 通过键名设置默认勾选的节点
      defaultCheckedKeys: Array,
      // 通过键名设置默认展开的节点
      defaultExpandedKeys: Array,
      // 自定义节点渲染方式
      renderContent: Function,
      // 显示选择框
      showCheckbox: {
        type: Boolean,
        default: false
      },
      // 定义基础数据结构
      props: {
        default() {
          return {
            children: 'children',
            label: 'label',
            icon: 'icon'
          };
        }
      },
      // 延迟加载数据
      lazy: {
        type: Boolean,
        default: false
      },
      // 高亮当前节点
      highlightCurrent: Boolean,
      // 通过键名设置当前节点
      currentNodeKey: [String, Number],
      // 自定义加载子节点数据的方式
      load: Function,
      // 遍历筛选节点的方法
      filterNodeMethod: Function,
      // 手风琴模式
      accordion: Boolean,
      // 子节点缩进距离
      indent: {
        type: Number,
        default: 16
      }
    },
    // 计算属性
    computed: {
      // 子节点数据
      children: {
        set(value) {
          this.data = value;
        },
        get() {
          return this.data;
        }
      }
    },
    // 监测数据
    watch: {
      // 默认勾选节点的键名
      defaultCheckedKeys(newVal) {
        this.store.defaultCheckedKeys = newVal;
        this.store.setDefaultCheckedKey(newVal);
      },
      // 默认展开节点的键名
      defaultExpandedKeys(newVal) {
        this.store.defaultExpandedKeys = newVal;
        this.store.setDefaultExpandedKeys(newVal);
      },
      // 当前节点的键名
      currentNodeKey(newVal) {
        this.store.setCurrentNodeKey(newVal);
        this.store.currentNodeKey = newVal;
      },
      // 数据源
      data(newVal) {
        this.store.setData(newVal);
      }
    },
    // 组件方法
    methods: {
      // 根据筛选方法执行筛选
      filter(value) {
        if (!this.filterNodeMethod) throw new Error('[Tree] filterNodeMethod is required when filter');
        this.store.filter(value);
      },
      // 根据节点键名获取键值
      getNodeKey(node, index) {
        const nodeKey = this.nodeKey;
        if (nodeKey && node) {
          return node.data[nodeKey];
        }
        return index;
      },
      // 获取勾选的节点
      getCheckedNodes(leafOnly) {
        return this.store.getCheckedNodes(leafOnly);
      },
      // 获取勾选节点的键值
      getCheckedKeys(leafOnly) {
        return this.store.getCheckedKeys(leafOnly);
      },
      // 设置选中的节点
      setCheckedNodes(nodes, leafOnly) {
        if (!this.nodeKey) throw new Error('[Tree] nodeKey is required in setCheckedNodes');
        this.store.setCheckedNodes(nodes, leafOnly);
      },
      // 设置选中节点的键值
      setCheckedKeys(keys, leafOnly) {
        if (!this.nodeKey) throw new Error('[Tree] nodeKey is required in setCheckedNodes');
        this.store.setCheckedKeys(keys, leafOnly);
      },
      // 设置选中的状态
      setChecked(data, checked, deep) {
        this.store.setChecked(data, checked, deep);
      },
      // 触发节点展开事件
      handleNodeExpand(nodeData, node, instance) {
        this.broadcast('ElTreeNode', 'tree-node-expand', node);
        this.$emit('node-expand', nodeData, node, instance);
      }
    },

    created() {
      this.isTree = true;
      // 设置仓库初始值
      this.store = new TreeStore({
        // 节点键名
        key: this.nodeKey,
        // 数据源
        data: this.data,
        // 延迟加载
        lazy: this.lazy,
        // 节点数据模板
        props: this.props,
        // 异步加载方法
        load: this.load,
        // 当前节点的键值
        currentNodeKey: this.currentNodeKey,
        // 是否父子节点关联
        checkStrictly: this.checkStrictly,
        // 默认勾选节点的键值
        defaultCheckedKeys: this.defaultCheckedKeys,
        // 默认展开节点的键值
        defaultExpandedKeys: this.defaultExpandedKeys,
        // 自动展开父节点
        autoExpandParent: this.autoExpandParent,
        // 自动全部展开
        defaultExpandAll: this.defaultExpandAll,
        // 筛选节点的方法
        filterNodeMethod: this.filterNodeMethod
      });
      // 设定根节点
      this.root = this.store.root;
    }
  };
</script>
