<template>
  <div :class="['level row ', `display-${filter.display}-wrap`]" @click.self="closeExpand">
    <div class="filter-tile-content">
      <dl v-for="(items, index) in filterItems"
          :class="['filter-tile',`level-${index}`, `display-${filter.display}`]"
          >
        <dt>{{items.name}}</dt>
        <dd v-for="(item, subIndex) in items.items"
            @click="onSelected(index, subIndex, items.display, item)"
            :class="['col-25', {selected: computedCur(index, subIndex, item)}]" :key="item.id">
          <div class="name">{{item.name}}</div>
        </dd>
      </dl>
    </div>
    <div class="footer">
      <button class="btn" @click="resetClick">重置</button>
      <button class="btn orange" @click="completeClick">完成</button>
    </div>
  </div>
</template>

<script>
    let queryData = [];
  export default {
    props: {
      filter: {
        type: Object
      },
      filterChange: {
        type: Function
      },
      closeExpand: {
        type: Function
      },
      query: {
        default: []
      }
    },
    data() {
      const activeIndexArry = [];
      this.filter.items.map((item, index) => {
        activeIndexArry.push({
          levelId: item.id,
          levelName: item.name,
          active: [],
          data: []
        });
        this.query.map((queryItem) => {
          if (queryItem[1].id == item.id || queryItem[1].name == item.name) {
//            for (let i = 1, iLen = queryItem.length; i < iLen; i++) {
//            activeIndexArry[index].levelName = queryItem[2].name;
//            activeIndexArry[index].levelId = queryItem[2].Id;
            activeIndexArry[index].active.push(queryItem[2].id);
            activeIndexArry[index].data.push(queryItem[2]);
//            }
          }
        });
      });
      return {
        activeIndex: activeIndexArry,
        selectIndex: 0,
        level: 0
      }
    },
    computed: {
      filterItems() {
        return this.filter.items;
      }
    },
    mounted() {

    },
    methods: {
      onSelected(index, subIndex, display, items) {
        let newState = {};
        newState = this.activeIndex[index];
        const position = newState.active.indexOf(items.id);
        if (position !== -1) {
          newState.active.splice(position, 1);
          newState.data.splice(position, 1)
        } else if (display === 5) {
          newState.active.push(items.id);
          newState.data.push(items);
        } else {
          newState.active = [items.id];
          newState.data = [items];

        }
        this.activeIndex[index] = newState;
      },
      activeData() {
        let tempDate = [];
        this.activeIndex.map((item) => {
          if (item.data) {
            tempDate.push(item.data);
          }
        });
        return tempDate;
      },
      computedCur(index, subIndex, item) {
        if (this.activeIndex[index].active.indexOf(subIndex) !== -1 || this.activeIndex[index].active.indexOf(item.id) !== -1) {
          return true;
        }
        return false;
      },
      resetClick() {
        this.activeIndex.map((item, index) => {
          this.activeIndex[index].active = [];
          this.activeIndex[index].data = [];
        })
        this.filterChange([], 'reset');
      },
      completeClick() {
        let tempData = [];
        this.activeIndex.map((item, index) => {
          if (item.data.length > 0) {
            item.data.map((subItem) => {
              tempData.push(subItem);
            })
          } else if (item.data[0]) {
            tempData.push(item.data[0]);
          }
        });
        this.filterChange(tempData);
      }
    }
  }
</script>

<style lang="less">
@import "./base.less";
.level {
  &.display-3-wrap {
    height: 80%;
    background: transparent;
    .filter-tile-content {
      background: #ffffff;
      max-height: 80%;
      .scrollable();
    }
  }
}

.filter-tile {
  overflow: hidden;
  padding: 12px 12px 6px 0;
  margin-left: 6px;
  position: relative;
  .hairline(bottom, #e0e0e0);
  dt {
    font-size: 14px;
    margin-bottom: 6px;
    margin-left: 6px;
  }
  dd {
    display: inline-block;
    width: 80px;
    text-align: center;
    font-size: 13px;
    padding: 6px;
    box-sizing: border-box;
    /*&:nth-child(4n+0) {
      margin-right: 0;
    }*/
    .name {
      position: relative;
      height: 35px;
      line-height: 36px;
      .hairline(all, #bcbcbc, 5px);
    }
    &.selected .name{
      background-color: #fff1ed;
      color: #f63;
      .hairline(all, #f63, 5px);
    }
  }

}
</style>
