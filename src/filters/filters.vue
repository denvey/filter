<template>
  <div class="filter-wrap">
    <ul class="filter-select-wrap row">
      <li v-for="(item, index) in filterData"
          :class="['select-li', {cur: selectIndex === index}, 'col-' + Math.floor(100/filterData.length)]"
          @click="onExpand(index)">
        <span class="select-title text-ellipsis-1">{{actives[index].name}}</span>
      </li>
    </ul>
    <transition name="slider">
      <div v-show="expanded" class="filter-modal-overly" @click.self="onCloseExpand">
        <!--<div class="select-multi-wrap">-->
          <template v-if="filterSub.length > 0" v-for="(item, index) in filterData">
            <FilterItemTile v-if="item.display == 3" v-show="index == selectIndex" :filter="item" :query="actives[index].query"
                            :filterChange="filterChange"></FilterItemTile>
            <FilterItem v-else v-show="index == selectIndex" :filter="item" :query="actives[index].query"
                        :filterChange="filterChange"/>
          </template>
        <!--</div>-->
      </div>
    </transition>
  </div>
</template>

<script>
  import FilterItem from './filter-item.vue';
  import FilterItemTile from './filter-item-tile.vue';

  export default {
    name: 'filters',
    props: {
      filters: {
        type: Array
      },
      query: {
        type: String
      },
      change: {
        type: Function
      }
    },
    data() {
      let data = [];
      let queryData = [];
      let query;
      if (this.query) {
        query = this.query.split(',');
      }
      for (let i = 0, iLen = this.filters.length; i < iLen; i++) {
        let tempData = {
          index: -1,
          level1Index: 0,
          lastIndex: 0,
          type: this.filters[i].type,
          name: this.filters[i].name,
          data: '',
          query: []
        };
        let tempData3 = {
          name: [],
          data: []
        };
        if (query) {
          query.forEach((item) => {
            const arrTemp = item.split('-');
            if (this.filters[i].type == arrTemp[0]) {
              this.filters[i].items.forEach(function (res) {
                if (res.id == arrTemp[1]) {
                  tempData.name = res.name;
                  tempData.data = res;
                }
              })
            }
            this.getPathById(this.filters[i], item, (data) => {
              const ilen = data.length;
              if (this.filters[i].display === 3) {
                console.log(data);
                tempData3.name.push(data[ilen - 1].name);
                tempData3.data.push(data[ilen - 1])
              } else {
                tempData.name = data[ilen - 1].name;
                tempData.data = data[ilen - 1];
              }
              tempData.query.push(data);
            })
          })
        }
        if (this.filters[i].display === 3) {
          if (tempData3.name.length > 0) {
            tempData.name = tempData3.name.join(',');
            tempData.data = tempData3.data;
          }
        }
        let tempObj = {};
        if (this.active && this.active[i]) {
          tempObj = Object.assign({}, tempData, this.active[i]);
        } else {
          tempObj = tempData;
        }
        data.push(tempObj);
      }
      return {
        actives: data,
        selectIndex: -1,
        filter: [],
        expanded: false,
        queryData: queryData,
        filterData: this.filters,
        filterSub: [],
        activeIndex: [],
        activeData: []
      }
    },
    components: {
      FilterItem,
      FilterItemTile
    },
    mounted() {
    },
    methods: {
      onExpand(index) {
        if (this.expanded) {
          if (this.selectIndex == index) {
            this.expanded = !this.expanded;
          }
          this.actives[this.selectIndex].level1Index = this.actives[this.selectIndex].lastIndex;
        } else {
          this.expanded = !this.expanded;
        }
        this.selectIndex = index;
        this.selectData(index);
        if (!this.expanded) {
          this.selectIndex = -1;
        }
      },
      filterChange(data, type) {
        this.actives[this.selectIndex].data = data;
        if (!this.isEmpty(data)) {
          let tempName;
          if (Array.isArray(data)) {
            let name = [];
            data.map((item) => {
              name.push(item.name);
            })
            tempName = name.join(",")
          } else {
            tempName = data.name;
          }
          if (!!tempName) {
            this.actives[this.selectIndex].name = tempName;
          }
        } else {
          this.actives[this.selectIndex].name = this.filters[this.selectIndex].name;
        }
        let tempData = [];
        this.actives.map((item) => {
          if (Array.isArray(item.data)) {
            item.data.map((subItem) => {
              if (subItem) {
                tempData.push(subItem);
              }
            })
          } else if (item.data) {
            tempData.push(item.data);
          }
        });
        if (type !== 'reset') {
          this.change(tempData);
          this.onCloseExpand();
        }
      },
      isEmpty(obj) {
        // 本身为空直接返回true
        if (obj == null) return true;
        // 然后可以根据长度判断，在低版本的ie浏览器中无法这样判断。
        if (obj.length > 0)    return false;
        if (obj.length === 0)  return true;
        const hasOwnProperty = Object.prototype.hasOwnProperty;
        // 最后通过属性长度判断。
        for (var key in obj) {
          if (hasOwnProperty.call(obj, key)) return false;
        }
        return true;
      },
      selectData(index, subIndex) {
        const curIndex = subIndex || 0;
        const data = this.filterData[index].items;
        this.filter = this.filterData[index];
        this.activeIndex = this.actives[index];
        if (data[curIndex].items) {
          this.filterSub = data[curIndex].items;
        } else {
          this.filterSub = this.filter.items;
        }
      },
      onCloseExpand() {
        this.actives[this.selectIndex].level1Index = this.actives[this.selectIndex].lastIndex;
        this.selectIndex = -1;
        this.expanded = false;
      },
      onChange(data) {
        let query = {
          brandid: this.query.brandid,
          query: [],
          page: 0,
          pagesize: 6
        };
        for (let item of this.actives) {
          if (item.type !== undefined && item.id !== undefined && item.id !== -1) {
            query.query.push(item.type + '-' + item.id);
          }
        }
        this.currentFilterId = -1;
        query.query = query.query.join(",");
      },
      onSelected(index, level) {
        let data;
        if (level === 2) {
          data = this.filterSub[index];
        } else {
          data = this.filter[index];
        }
        this.actives[this.selectIndex].name = data.name;
        this.actives[this.selectIndex].id = data.id;
        this.actives[this.selectIndex].type = data.type;
        this.actives[this.selectIndex].index = data.id;
        this.actives[this.selectIndex].lastIndex = this.actives[this.selectIndex].level1Index;
        this.activeIndex = this.actives[this.selectIndex];
        this.expanded = false;
        this.onChange(Object.assign({}, this.actives[this.selectIndex], data));
        if (!this.expanded) {
          this.selectIndex = -1;
        }
      },
      getPathById(data, query, callback) {
        let path = [];
        const queryParam = query.split('-');
        try {
          const queryArray = (data) => {
            path.push({
              name: data.name,
              id: data.id,
              type: data.type
            });
            if (data.type == queryParam[0] && data.id == queryParam[1]) {
              throw ("GOT IT!");
            }
            if (data.items && data.items.length > 0) {
              for (let i = 0, iLen = data.items.length; i < iLen; i++) {
                queryArray(data.items[i]);
              }
              path.pop();
            } else {
              path.pop();
            }
          };
          queryArray(data);
        } catch (e) {
          !!callback && callback(path);
        }
      }
    }
  }
</script>

<style lang="less">
  @import "./filters.less";
</style>
