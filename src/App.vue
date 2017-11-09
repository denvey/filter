<template>
  <div id="app">
    <Filters v-if="filterData.length > 0" :filters="filterData" :query="selectParams" :change="change" />
  </div>
</template>

<script>
import Filters from './components/filters'

export default {
  name: 'app',
  data () {
    return {
      filterData: [],
      selectParams: '1-842,5-3,5-6,6-1000' // 2-1161
    }
  },
  components: {
    Filters
  },
  mounted () {
    fetch('https://www.easy-mock.com/mock/58ff3f8a5e43ae5dbea60053/qqshsh/shopping/brand/shop/filter')
      .then((res) => {
        return res.json()
      }).then((data) => {
        this.filterData = data.msg.filterDos;
        console.log(data)
      })
  },
  methods: {
    change (data) {
      console.log(data)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
  * {
    margin: 0;
    padding: 0;
  }
</style>
