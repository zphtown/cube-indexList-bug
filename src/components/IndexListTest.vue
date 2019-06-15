<template>
  <div class="view-wrapper">
    <cube-index-list
            ref="indexList"
            :data="list"
            :navbar="false"
            :title="title"
            :options="options"
            @pulling-up="onPullingUp"
            @pulling-down="onPullingDown"
    >
      <cube-index-list-group v-for="(group, index) in list" :key="index" :group="group">
        <cube-index-list-item v-for="(item, index) in group.items" :key="index" :item="item">
          <div class="custom-item">
            <p>{{item.createTime}}</p>
            <p v-if="item.expenditure<0" style="color:red">{{item.expenditure}}</p>
            <p v-else-if="item.income">{{item.income}}</p>
          </div>
        </cube-index-list-item>
      </cube-index-list-group>
    </cube-index-list>
  </div>
</template>

<style lang="stylus" rel="stylesheet/stylus">
  .view-wrapper
    position: fixed
    top: 0
    left: 0
    right: 0
    bottom: 0
    width: 100%
  .custom-item{
    display: flex;
    justify-content:space-between;
    padding: 20px;
  }
</style>

<script>
import axios from 'axios'
import {parseTime} from '../utils'
export default {
  data () {
    return {
      title: '没有按照时间顺序排序，不要在意，只为测试',
      data: [],
      options: {
        pullUpLoad: {
          txt: {
            more: '',
            noMore: '没有更多数据'
          },
          visible: true,
          threshold: 0
        },
        pullDownRefresh: {
          txt: '刷新完成'
        }
      },
      from: 1,
      size: 20,
      list: [],
      noMoreData: false
    }
  },
  filters: {
    parseTime
  },
  mounted () {
    this.getList()
  },
  methods: {
    selectItem (item) {
      // console.log(item.name)
    },
    clickTitle (title) {
      // console.log(title)
    },
    onPullingDown () {
      this.from = 1
      this.getList(1)
    },
    onPullingUp () {
      this.getList()
    },
    getList (isUpdate) {
      axios.get('http://api.zphtown.com/getGoldList.php', {
        params: {
          from: this.from,
          size: this.size
        }
      })
      .then(res => {
        const { list } = res.data
        let arr = []
        if (!isUpdate) {
          arr = [...this.list]
        }
        list.map(v => {
          let time = parseTime(v.createTime, '{y}年{m}月')
          let index = arr.findIndex(v2 => v2.name === time)
          if (index === -1) {
            arr.push({
              name: time,
              items: []
            })
          }
          arr[index > -1 ? index : arr.length - 1].items.push(v)
        })
        this.list = arr
        const len = list.length
        if (!isUpdate) {
          if (len < this.size) {
            if (len === 0) {
              this.$refs.indexList.forceUpdate(false, true)
            } else {
              this.$refs.indexList.forceUpdate(true, true)
            }
          } else {
            this.$refs.indexList.forceUpdate(true, false)
          }
          this.from++
        } else {
          console.log('h')
          this.$refs.indexList.forceUpdate(true)
        }
      })
    }
  }
}
</script>
