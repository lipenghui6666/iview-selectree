# iview-selectree

基于iview + vue2 进行二次开发的级联树状选择组件，只支持多选，搜索,（注：使用前先引入iview，否则无法使用）

<div>
 <img src="/selectTree/src/assets/123.gif" width="800" height="500" />
<div>


## 引入模块

      <selectTree style="width: 300px;"       
                        v-model="current"    //数组     
                        :treeData="treeData"     
                        filterable        
                        @select-tree="handleSelect"       
                        :maxtagcount="2"          
    ></selectTree>  

 ### 入参  （参考iview tree）

``` bash
 treeData: [                       
         {                        
           id: 1,    //必填      
           title: '中国',     
           selected: false,  //必填          
           children: []             
          }                    
         ]                       
  
```

       
    
