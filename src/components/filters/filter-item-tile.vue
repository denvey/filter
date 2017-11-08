<template>
  <div :class="['row']">
    <dl v-for="(items, index) in filterItems"
        :class="['filter-tile',`level-${index}`, `display-${filter.display}`]"
        >
      <dt>{{items.name}}</dt>
      <dd v-for="(item, subIndex) in items.items"
          @click="onSelected(index, subIndex, items.display, item)"
          :class="['col-25', {selected: computedCur(index, subIndex, item)}]" :key="item.id">
        <div>{{item.name}}</div>
      </dd>
    </dl>
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
          if (queryItem[1].name == item.name) {
            for (let i = 1, iLen = queryItem.length; i < iLen; i++) {
              activeIndexArry[index].levelName = queryItem[i].name;
              activeIndexArry[index].levelId = queryItem[i].Id;
              activeIndexArry[index].active = [queryItem[i].Id];
              activeIndexArry[index].data = [queryItem[i]];
            }
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
        /*let tempItem = [];
        let i = 0;
        tempItem.push(this.filter);
        const handleItems = (data) => {
          if (data && data.length > 0) {
            let curIndex = 0;
            const curId = this.activeIndex[this.selectIndex]['level' + i + 'Id'];
            const curName = this.activeIndex[this.selectIndex]['level' + i + 'Name'];
            if (curId || curName) {
              for (let m = 0, mLen = data.length; m < mLen; m++) {
                if (data[m].id === curId || data[m].name === curName) {
                  curIndex = m;
                }
              }
            }
            const tempData = data[curIndex];
            if (tempData && tempData.items !== undefined) {
              tempItem.push(tempData.items);
              i++;
              handleItems(tempData.items);
            }
          }
        };
        handleItems(this.filter);
        this.level = i;*/
        return this.filter.items;
      }
    },
    mounted() {

    },
    methods: {
      onSelected(index, subIndex, display, items) {
        let newState = {};
        newState = this.activeIndex[index];
        const position = newState.active.indexOf(subIndex);
        if (position !== -1) {
          newState.active.splice(position, 1);
          newState.data.splice(position, 1)
        } else if (display === 5) {
          newState.active = [subIndex];
          newState.data = [items];
        } else {
          newState.active.push(subIndex);
          newState.data.push(items);
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
        if (this.activeIndex[index].active.indexOf(subIndex) !== -1) {
          return true;
        }
        return false;
      },
      resetClick() {
        this.activeIndex.map((item, index) => {
          this.activeIndex[index].active = [];
        })
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
.filter-tile {
  overflow: hidden;
  padding: 12px 12px 12px 0;
  margin-left: 12px;
  position: relative;
  .hairline(bottom, #e0e0e0);
  dt {
    font-size: 14px;
    margin-bottom: 12px;
  }
  dd {
    display: inline-block;
    width: 80px;
    height: 35px;
    line-height: 35px;
    text-align: center;
    position: relative;
    .hairline(all, #bcbcbc, 5px);
    font-size: 13px;
    margin-right: 10px;
    &:nth-child(4n+0) {
      margin-right: 0;
    }
    &.selected {
      background-color: #fff1ed;
      color: #f63;
      .hairline(all, #f63, 5px);
    }
  }

}
</style>
