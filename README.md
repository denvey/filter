# filter

> vue 筛选组件

### 数据格式

```json
[{
   "name": "全部商区",
   "type": 3,
   "display": 3,
   "id": null
 }, {
  "name": "星级价格",
  "type": 3,
  "display": 3,
  "id": null,
  "items": [{
    "type": 0,
    "id": 1,
    "display": 4,
    "name": "星级",
    "count": 0,
    "items": [{
      "type": 5,
      "id": 3,
      "name": "三星级",
      "count": 0,
    }, ..]
  }, ...]
}, {
   "name":"距离排序",
   "type":3,
   "display":0,
   "id":null,
   "items":[
       {
           "type":0,
           "id":1,
           "name":"距离排序",
           "count":0
       },
       {
           "type":0,
           "id":2,
           "name":"智能排序",
           "count":0
       },
       {
           "type":0,
           "id":3,
           "name":"人气排序",
           "count":0
       }
   ]
}]
```

### 参数

```js  
query   String   需要选中的item,类型+id(type-id),如1-1000，3-10001
change  Function 选择完成触发的回调函数 
filters Array    显示的数据 
filters主要使用type、name、id、display、count，其他字段可以自定义
[{
    type: xxx,
    name: xxxx,
    id: xxxx,
    count: xxx,
    display: 1  //  0 默认竖向显示
                    1 横向显示
                    2
                    3  横铺
                    5  多选（目前只支持横铺模式下多选） 
}]
```

