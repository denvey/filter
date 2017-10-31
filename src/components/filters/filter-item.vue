<template>
  <div :class="['level row',`level-total-${filterItems.length}`, {'level-total-gt2': filterItems.length >= 2}]">
    <ul v-for="(items, index) in filterItems"
        :class="['select-item-wrap',`level-${index}`, `display-${items[0].display}`, 'col-' + colCss(items[0].display, filterItems.length)]"
        v-if="items.length > 0">
      <li v-for="(item, subIndex) in items"
          @click="onSelected(index, subIndex, item)"
          :class="[{cur: computedCur(item, index, subIndex)}]" :key="item.id">
        <div>{{item.name}}</div>
      </li>
    </ul>
    <ul v-else>

    </ul>
  </div>
</template>

<script>
  export default {
    name: 'filter-item',
    props: {
      filter: {
        type: Array
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
      this.filter.map((item, index) => {
        activeIndexArry.push({
          level0: 0,
          level0Id: item.id,
          level0Name: item.name
        });
        this.query.map((queryItem) => {
          if (queryItem[1].name == item.name) {
            for (let i = 1, iLen = queryItem.length; i < iLen; i++) {
              activeIndexArry[index][`level${i - 1}Name`] = queryItem[i].name;
              activeIndexArry[index][`level${i - 1}Id`] = queryItem[i].Id;
            }
          }
        });
      });
      return {
        activeIndex: activeIndexArry,
        selectIndex: 0,
        level: 0,
      }
    },
    computed: {
      filterItems() {
        let tempItem = [];
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
        this.level = i;
        return tempItem;
      }
    },
    mounted() {

    },
    methods: {
      onSelected(ulIndex, index, items) {
        if (ulIndex === this.level || items.items === undefined || items.items.length == 0) {
          this.activeIndex[this.selectIndex].data = items;
          this.filterChange(this.activeData());
        } else if (ulIndex === 0) {
          this.selectIndex = index;
        }
        this.$set(this.activeIndex[this.selectIndex], 'level' + ulIndex + 'Id', items.id);
        this.$set(this.activeIndex[this.selectIndex], 'level' + ulIndex + 'Name', items.name);
        this.$set(this.activeIndex[this.selectIndex], 'level' + ulIndex, index);
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
      computedCur(item, index, subIndex) {
        const id = this.activeIndex[this.selectIndex]['level' + index + 'Id'];
        const name = this.activeIndex[this.selectIndex]['level' + index + 'Name'];
        if (id === item.id || name === item.name) {
          /*if (item.items === undefined || item.items.length === 0) {
            this.activeDatas.push(item);
          }*/
          return true;
        } else if (index < this.level && subIndex === 0 && !id && !name) {
          return true;
        }
        return false;
      },
      colCss(display, length) {
        if (display === 1) {
          return Math.floor(100 / (length - 1));
        } else {
          return Math.floor(100 / length);
        }
      }
    }
  }
</script>
