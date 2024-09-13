<template>
  <div id="app">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
      <selectTree style="width: 300px;"
                        v-model="current"
                        :treeData="treeData"
                        filterable
                        @select-tree="handleSelect"
                        :maxtagcount="2"
    ></selectTree>  
  </div>
</template>

<script>
import selectTree from './components/selectTree.vue'

export default {
  name: 'App',
  components: {
    selectTree
  },
  data(){
    return {
      current: [],
      treeData: [
        {
          id: 1,
          title: '中国',
          selected: false,
          children: [
            {
              id: 11,
              title: '江苏',
              selected: false,
              children: [
                {
                  id: 111,
                  title: '南京',
                  selected: false,
                },
                {
                  id: 112,
                  title: '苏州',
                  selected: false,
                }
              ]
            },
            {
              id: 12,
              title: '河南',
              selected: false,
            }
          ]
        },
        {
          id: 2,
          title: '美国',
          selected: false,
          children: [
            {
              id: 21,
              title: '123',
              selected: false,
            },
            {
              id: 22,
              title: '456',
              selected: false,
            }
          ]
        }
      ]
    }
  },
  methods: {
    handleSelect(val) {
      console.log(val);
    }
  }
}
</script>

<style>
#app {
  display: flex;
  justify-content: center;
  margin-top: 100px;
}
</style>
