<template>
  <div>
    <div class="mask" v-show="isShowSelect" @click="isShowSelect = !isShowSelect"></div>
    <el-popover placement="bottom-start" :width="width" trigger="click" v-model="isShowSelect">
      <el-tree
        class="common-tree"
        :style="style"
        ref="tree"
        :data="data"
        :props="defaultProps"
        :show-checkbox="multiple"
        :node-key="nodeKey"
        :default-expand-all="expandAll"
        :check-strictly="checkStrictly"
        :expand-on-click-node="false"
        :check-on-click-node="multiple"
        :highlight-current="true"
        @node-click="handleNodeClick"
        @check-change="handleCheckChange"
      ></el-tree>
      <el-select
        slot="reference"
        ref="select"
        :size="size"
        v-model="selectedData"
        :multiple="multiple"
        :clearable="clearable"
        :collapse-tags="true"
        :placeholder="placeholder"
        :isGoodsTree="isGoodsTree"
        @click.native="isShowSelect = !isShowSelect"
        @remove-tag="removeSelectedNodes"
        @clear="removeSelectedNode"
        @change="changeSelectedNodes"
        class="tree-select"
      >
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
    </el-popover>
  </div>
</template>
 
<script>
export default {
  name: "tree-select",
  props: {
    // 树结构数据
    data: {
      type: Array,
      default() {
        return [];
      }
    },
    defaultProps: {
      type: Object,
      default() {
        return {};
      }
    },
    // 配置是否可多选
    multiple: {
      type: Boolean,
      default() {
        return false;
      }
    },
    // 配置默认提示信息
    placeholder: {
      type: String,
      default() {
        return "请选择商户";
      }
    },
    // 判断是否是商品树，商品树单选不允许选择父节点
    isGoodsTree: {
      type: Boolean,
      default() {
        return false;
      }
    },
    // 配置是否可清空选择
    clearable: {
      type: Boolean,
      default() {
        return false;
      }
    },
    // 配置多选时是否将选中值按文字的形式展示
    collapseTags: {
      type: Boolean,
      default() {
        return false;
      }
    },
    nodeKey: {
      type: String,
      default() {
        return "id";
      }
    },
    defaultCheckedKeys: {
      type: Array,
      default() {
        return [];
      }
    },
    // 显示复选框情况下，是否严格遵循父子不互相关联
    checkStrictly: {
      type: Boolean,
      default() {
        return false;
      }
    },
    // 是否默认展开所有节点
    expandAll: {
      type: Boolean,
      default() {
        return false;
      }
    },
    // 默认选中的节点key数组
    checkedKeys: {
      type: Array,
      default() {
        return [];
      }
    },
    size: {
      type: String,
      default() {
        return "normal";
      }
    },
    width: {
      type: Number,
      default() {
        return 220;
      }
    },
    height: {
      type: Number,
      default() {
        return 300;
      }
    }
  },
  data() {
    return {
      isShowSelect: false, // 是否显示树状选择器
      options: [],
      selectedData: [], // 选中的节点
      selectedDataName: [],
      style:
        "width:" +
        this.width +
        "px;" +
        "height:" +
        this.height +
        "px;" +
        "min-width:220px",
      checkedIds: [],
      checkedData: []
    };
  },
  mounted() {
    this.initCheckedData();
  },
  methods: {
    // 单选时点击tree节点，设置select选项
    setSelectOption(node) {
      let tmpMap = {};
      tmpMap.value = node.key;
      tmpMap.label = node.label;
      this.options = [];
      this.options.push(tmpMap);
      this.selectedData = node.key;
    },
    // 单选，选中传进来的节点
    checkSelectedNode(checkedKeys) {
      var item = checkedKeys[0];
      this.$refs.tree.setCurrentKey(item);
      var node = this.$refs.tree.getNode(item);
      this.setSelectOption(node);
    },
    // 多选，勾选上传进来的节点
    checkSelectedNodes(checkedKeys) {
      //this.selectedData = checkedKeys;
      this.$refs.tree.setCheckedKeys(checkedKeys);
    },
    // 单选，清空选中
    clearSelectedNode() {
      this.selectedData = "";
      this.$refs.tree.setCurrentKey(null);
    },
    // 多选，清空所有勾选
    clearSelectedNodes() {
      var checkedKeys = this.$refs.tree.getCheckedKeys(); // 所有被选中的节点的 key 所组成的数组数据
      for (let i = 0; i < checkedKeys.length; i++) {
        this.$refs.tree.setChecked(checkedKeys[i], false);
      }
    },
    initCheckedData() {
      if (this.multiple) {
        // 多选
        if (this.checkedKeys.length > 0) {
          this.checkSelectedNodes(this.checkedKeys);
        } else {
          this.clearSelectedNodes();
        }
      } else {
        // 单选
        if (this.checkedKeys.length > 0) {
          this.checkSelectedNode(this.checkedKeys);
        } else {
          this.clearSelectedNode();
        }
      }
    },
    // 单选，节点被点击时的回调,返回被点击的节点数据
    handleNodeClick(data, node) {
      if (!this.multiple) {
        // 商品树单选时判断选择的是否是商品
        if (this.isGoodsTree) {
          if (node.key[0] === "a") {
            this.setSelectOption(node);
            this.isShowSelect = !this.isShowSelect;
            this.$emit("change", this.selectedData);
          }
        } else {
          this.setSelectOption(node);
          this.isShowSelect = !this.isShowSelect;
          this.$emit("change", this.selectedData);
        }
      }
    },
    // 多选，节点勾选状态发生变化时的回调
    handleCheckChange() {
      var checkedKeys = this.$refs.tree.getCheckedKeys(); // 所有被选中的节点的 key 所组成的数组数据
      this.options = checkedKeys.map(item => {
        var node = this.$refs.tree.getNode(item); // 所有被选中的节点对应的node
        let tmpMap = {};
        tmpMap.value = node.key;
        tmpMap.label = node.label;
        return tmpMap;
      });
      this.selectedData = this.options.map(item => {
        return item.value;
      });
      this.selectedDataName = [];
      this.selectedDataName = this.options.map(item => {
        return item.label;
      });
      //传给父组件id和text
      this.$emit("change", this.selectedData, this.selectedDataName);
    },
    // 多选,删除任一select选项的回调
    removeSelectedNodes(val) {
      this.$refs.tree.setChecked(val, false);
      var node = this.$refs.tree.getNode(val);
      if (!this.checkStrictly && node.childNodes.length > 0) {
        this.treeToList(node).map(item => {
          if (item.childNodes.length <= 0) {
            this.$refs.tree.setChecked(item, false);
          }
        });
        this.handleCheckChange();
      }
      this.$emit("change", this.selectedData, this.selectedDataName);
    },
    treeToList(tree) {
      var queen = [];
      var out = [];
      queen = queen.concat(tree);
      while (queen.length) {
        var first = queen.shift();
        if (first.childNodes) {
          queen = queen.concat(first.childNodes);
        }
        out.push(first);
      }
      return out;
    },
    // 单选,清空select输入框的回调
    removeSelectedNode() {
      this.clearSelectedNode();
      this.$emit("change", this.selectedData, this.selectedDataName);
    },
    // 选中的select选项改变的回调
    changeSelectedNodes(selectedData, selectedDataName) {
      // 多选,清空select输入框时，清除树勾选
      if (this.multiple && selectedData.length <= 0) {
        this.clearSelectedNodes();
      }
      this.$emit("change", this.selectedData, this.selectedDataName);
    }
  },
  watch: {
    isShowSelect(val) {
      // 隐藏select自带的下拉框
      this.$refs.select.blur();
    },
    checkedKeys(val) {
      if (!val) return;
      this.checkedKeys = val;
      this.initCheckedData();
    }
  }
};
</script>
 
<style lang="scss">
.mask {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  left: 0;
  opacity: 0;
  z-index: 11;
}
.common-tree {
  overflow: auto;
  .el-tree-node > .el-tree-node__children {
    overflow: visible;
  }
  label {
    margin-bottom: 0;
    margin-right: 5px !important;
  }
}
.tree-select {
  z-index: 111;
  width: 200px;
  .el-select__tags > span {
    width: 200px;
    span:nth-child(2) {
      // display: inline-block;
      // overflow: hidden;
      display: none;
    }
    span:first-child {
      .el-select__tags-text {
        display: inline-block;
        width: 120px;
        text-overflow: ellipsis;
        overflow: hidden;
      }
      i {
        top: -6px;
      }
    }
  }
}
</style>