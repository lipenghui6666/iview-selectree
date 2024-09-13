<template>
  <Select
    v-model="title"
    multiple
    :disabled="disabled"
    clearable
    class="select_tree"
    :id="'select_tree'+ selectKey"
    ref="SelectBox"
    @on-change="scopedSelect"
    :max-tag-count="maxtagcount"
  >
    <Option :value="ltem.id" class="hidden_option" v-for="ltem in option" :key="ltem.title"> {{ ltem.title }}</Option>
    <template v-if="filterable">
      <Input
        v-model="filterValue"
        placeholder="请搜索"
        icon="ios-search"
        class="filter_input"
        clearable
        @on-change="handleChangeInput"
      />
    </template>
    <Tree :data="successTrees"  multiple  class="tree_class" :id="'tree_id' + selectKey" :render="renderContent" :key="externalChangevalue"></Tree>
  </Select>
</template>

<script>
import { cloneDeep, debounce } from "lodash";
export default {
  "name": "select-tree-L",
  "model": {
      "prop": "value",
      "event": "update-value"
  },
  "props": {
      "disabled": { "type": Boolean, "default": false },
      //是否开启搜索功能
      "filterable": {
          "type": Boolean,
          "default": false
      },
      //树形数据
      "treeData": {
          "type": Array,
          "default": () => {
              return [];
          }
      },
      //双向绑定值
      "value": {
          "type": Array,
          "default": []
      },
      "maxtagcount": {
          "type": Number,
          "default": Number.MAX_VALUE
      }
  },
  data () {
      return {
          "externalChangevalue": new Date().getTime() + Math.random(), //外部改变key ,更新视图 节约性能
          "title": [], //select显示值
          "option": [], //option值
          "filterValue": null,
          "successTrees": [], //树形组件数据
          "treeAlldata": [],
          "checkAll": [],
          "checkeddadatree": [], //选中树数据
          "selectKey": new Date().getTime() + Math.random()
      };
  },
  mounted () {
      const self = this;
      this.treeAlldata = cloneDeep(this.treeData);
      this.successTrees = cloneDeep(this.treeData);

      let select_tree = document.getElementById("select_tree" + self.selectKey);
      let id = "cleartreedata" + this.selectKey;
      select_tree.insertAdjacentHTML("beforeend", 
          `<div id= ${id} class='cleartreedata'>
      <svg t="1723625709720" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4351" width="20" height="20"><path d="M980.151 313.82c-26.653-60.184-62.764-113.49-108.332-161.639-45.569-46.428-101.455-82.539-161.64-109.192C647.417 16.336 582.073 1.72 512.43 1.72c-67.922 0-134.985 14.617-197.749 41.27-60.185 26.653-113.491 62.764-161.639 108.333C105.753 197.75 69.642 253.636 42.99 313.82 16.336 376.584 1.72 441.928 1.72 511.57c0 67.923 14.617 134.986 41.27 197.75 26.653 60.185 62.764 113.491 108.333 161.639 45.568 45.568 101.454 82.539 161.639 108.332 62.764 26.654 128.107 41.27 197.75 41.27 67.922 0 134.985-14.616 197.75-41.27 60.184-26.653 113.49-62.764 161.638-108.332 45.569-45.569 82.54-101.454 108.333-161.64 26.653-62.763 41.27-128.107 41.27-197.749 1.719-67.063-12.898-134.986-39.55-197.75zM738.552 770.364L514.15 545.961 294.045 765.206l-33.531-33.532L480.618 511.57 255.355 287.167l33.532-33.531L513.29 478.039l220.964-220.104 33.531 33.531L547.681 511.57l224.403 224.403-33.532 34.391z" p-id="4352" fill="#707070"></path></svg>
      </div>`);
      let cleartreedata = document.getElementById(id);
      cleartreedata.onclick = ()=>{
          self.Clearchecked();
      };
  },
  "watch": {
      treeData (newVal){
          this.treeAlldata = cloneDeep(newVal);
          this.successTrees = cloneDeep(newVal);
      }
  },
  "computed": {},
  "methods": {
      renderContent (h, { root, node, data }) {
          return h("span", {
              "style": {
                  "display": "inline-block",
                  "width": "100%",
                  "pointerEvents": "none"
              }
          }, [
              h("span", [
                  h("Checkbox", {
                      "props": {
                          "value": data.selected
                      },
                      "on": {
                          "on-change": () => this.handleSelectTree(data)
                      },
                      "style": {
                          "marginRight": "2px",
                          "marginLeft": "8px",
                          "pointerEvents": "auto"
                      }
                  }),
                  h("span",{
                      "style": {
                          "pointerEvents": "none"
                      }
                  },data.title)
              ])
          ]);
      },
      "scopedSelect": debounce(function (e){
          if (e.length != this.checkAll.length){
              if (e.length == 0) {
                  this.checkAll = [];
                  this.option = [];
                  this.treeAlldata = cloneDeep(this.treeData);
                  this.successTrees = cloneDeep(this.treeData);
                  this.$emit("update-value", this.title);
                  this.$emit("select-tree", this.option);
                  this.QxTree(this.successTrees);
                  return;
              }
              this.checkAll = e;
              let arr = this.option;
              this.option = e.map(ltem=>{
                  let index = arr.findIndex(item=>item.id == ltem);
                  return {"title": arr[index].title,"id": arr[index].id};
              });
              this.QxTree(this.successTrees);
              this.$emit("update-value", this.title);
              this.$emit("select-tree", this.option);
          }
      },300),
      //递归清楚已选
      QxTree (originGens){
          for (let ltem of originGens) {
              let isid = ltem.id && this.checkAll.some(id=>id == ltem.id);
              let isChildren = ltem.children && ltem.children.length > 0;
              ltem.selected = isid ?  true : false;
              if (isChildren){
                  this.QxTree(ltem.children);
              } 
          }
          this.externalChangevalue = new Date().getTime() + Math.random(); 
      },

      handleSelectTree (now){
          let index = this.checkAll.findIndex(ltem=>ltem == now.id);
          if (index<0){
              this.checkAll.length >=0 && this.checkAll.push(now.id);
              this.title.push(now.id);
              this.option.push({"title": now.title,"id": now.id});
          } else {
              this.checkAll.splice(index,1);
              this.title.splice(index,1);
              this.option.splice(index,1);
          }
          this.$emit("update-value", this.title);
          this.$emit("select-tree", this.option);
      },

      Clearchecked (){
          if (this.title.length > 0){
              this.title = [];
              this.option = [];
              this.checkAll = [];
              this.treeAlldata = [];
              this.successTrees = []; 
              this.treeAlldata = cloneDeep(this.treeData);
              this.successTrees = cloneDeep(this.treeData);
              this.QxTree(this.successTrees);
              this.$emit("update-value", []);
              this.$emit("select-tree", []);
          }
      },
      //搜索框值改变
      "handleChangeInput": debounce(function () {
          let treedom = document.getElementById("tree_id"+ this.selectKey);
          let str = `<div id="tree_loading">
          <div style="color:#2d8cf0">加载中...</div>
          </div>`;
          treedom.insertAdjacentHTML("beforeend", str);
          let originGens = {
              "data": this.treeAlldata,
              "parent": "",
              "current": this.value,
              "key": this.selectKey,
              "value": true
          };
          if (!this.filterValue) {
              this.$nextTick(()=>{
                  this.successTrees = this.treeAlldata;
                  this.QxTree(this.successTrees);
              });
              setTimeout (()=>{
                  let element = document.getElementById("tree_loading");
                  element && element.parentNode.removeChild(element);
              },300);
          } else {
              this.successTrees = this.recursiveTitle(originGens);
              setTimeout (()=>{
                  let element = document.getElementById("tree_loading");
                  element && element.parentNode.removeChild(element);
              },300);
          }
      }, 250),

      //递归找名字
      recursiveTitle (
          trees = { "data": [], "parent": "", "current": {}, "key": "ruleId", "value": false }
      ) {
          let findNames = [];
          trees.data.forEach(item => {
              let treeItem = {};
              let treeItemChildern = [];
              let isTitle = item.title && item.title.includes(this.filterValue);
              let isChildren = item.children && item.children.length > 0;
              let treeItemChildernLength = 0;

              if (isTitle) {
                  treeItem = {
                      ...item,
                      "selected": this.checkAll.some(ltem=>ltem == item.id)
                  };
              }
              if (trees.parent) {
                  trees.parent.expand = true;
              }
              if (isChildren) {
                  let itemChildren = {
                      ...trees,
                      "data": item.children
                  };
                  isTitle
                      ? (itemChildren.parent = treeItem)
                      : (itemChildren.parent = "");
                  treeItemChildern = this.recursiveTitle(itemChildren);
                  treeItemChildernLength = treeItemChildern.length;
              }
              if (isTitle && treeItemChildernLength > 0) {
                  treeItem.children = treeItemChildern;
                  return findNames.push(treeItem);
              }

              if (isTitle && treeItemChildernLength === 0) {
                  return findNames.push(treeItem);
              }

              if (!isTitle && treeItemChildernLength > 0) {
                  return (findNames = findNames.concat(treeItemChildern));
              }
          });

          return findNames;
      }
  }
};
</script>

<style lang="less" scoped>
.select_tree {
  position: relative;
  ::v-deep .ivu-select-dropdown {
    padding: 5px;
    width: 300px !important;
    max-height: 400px !important;
    height: 260px !important;
    overflow: hidden;
  }
}

.hidden_option {
  display: none;
}
/deep/.ivu-tree-empty {
  text-align: center;
  height: 30px;
  line-height: 30px;
}
/deep/.tree_class{
  overflow: auto;
  height: 220px;
}
/deep/.cleartreedata{
  position:absolute;
  width:30px;
  height:30px;
  top:0px;
  bottom:0px;
  right:30px;
  margin: auto;
  font-size:16px;
  z-index: 100;
  opacity: 0;
  pointer-events: all;
  display: flex;
  align-items: center;
  justify-content: center;
}
/deep/.cleartreedata:hover{
  opacity: 1;
}
/deep/ #tree_loading {
  height:200px;
  background-color:rgba(255,255,255,0.9);
  position:absolute;
  top:0;
  width:300px;
  z-index:100
}
/deep/ .ivu-tree-title{
  pointer-events: none;
}
</style>