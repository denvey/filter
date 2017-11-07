<template>
  <div class="filter-wrap">
    <ul class="filter-select-wrap row">
      <li v-for="(item, index) in filterData"
          :class="['select-li', {cur: selectIndex === index}, 'col-' + Math.floor(100/filterData.length)]"
          @click="onExpand(index)">
        <span class="select-title mb-text-ellipsis-1">{{actives[index].name}}</span>
      </li>
    </ul>
    <transition name="slider">
      <div v-show="expanded" class="filter-modal-overly" @click.self="onCloseExpand">
        <div class="select-multi-wrap">
          <template v-if="filterSub.length > 0" v-for="(item, index) in filterData">
            <FilterItemTile v-if="item.display == 3" v-show="index == selectIndex" :filter="item" :query="queryData"
                            :filterChange="filterChange"></FilterItemTile>
            <FilterItem v-else v-show="index == selectIndex" :filter="item.items" :query="queryData"
                        :filterChange="filterChange"/>
          </template>
        </div>
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
          query: this.filters[i].items ? this.filters[i].items[0] : {}
        };
        if (query) {
          query.forEach((item) => {
            const arrTemp = item.split('-');
            if (this.filters[i].type == arrTemp[0]) {
              this.filters[i].items.forEach(function (res) {
                if (res.id == arrTemp[1]) {
                  tempData.name = res.name;
                  tempData.query = res;
                }
              })
            }
            this.getPathById(this.filters[i], item, (data) => {
                const ilen = data.length;
                tempData.name = data[ilen - 1].name;
                tempData.query = data[ilen - 1];
                queryData.push(data);
            })
          })
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
      onItemClick(item) {
        if (this.currentFilterId === item.id) {
          return;
        }
        this.currentFilterId = item.id;
        let query = {
          brandid: this.query.brandid,
          query: item.type + '-' + item.id,
          page: 0,
          pagesize: 6
        };
        for (let i = 0, iLen = this.actives.length; i < iLen; i++) {
          if (this.actives[i].type == item.type) {
            this.actives[i].id = item.id;
            this.actives[i].name = item.name;
          } else {
            this.actives[i].id = -1;
            this.actives[i].name = this.filterList[i].name;
          }
        }
      },
      onExpandMore() {
        this.expandMore = !this.expandMore;
      },
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
      filterChange(data) {
        this.actives[this.selectIndex].query = data;
          this.actives[this.selectIndex].name = data.name;
          let tempData = [];
        this.actives.map((item) => {
//          item.map((subItem) => {
            tempData.push(item.query);
//          })
        });
        this.change(tempData);
        this.onCloseExpand();
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
      onSelectLevel1(index) {
        const items = this.filter[index].items;
        this.actives[this.selectIndex].level1Index = index;
        this.actives[this.selectIndex].index = -1;
        this.activeIndex = this.actives[this.selectIndex];
        if (items) {
          this.filterSub = items;
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
