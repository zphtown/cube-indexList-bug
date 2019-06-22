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
            more: '上拉加载更多',
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
      isNoMore: false
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
      this.isNoMore = false
      this.from = 1
      this.getList(1)
    },
    onPullingUp () {
      if (this.isNoMore) {
        this.$refs.indexList.forceUpdate()
        return
      }
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
        const len = list.length
        let arr = []
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
        if (isUpdate) {
          if (len) {
            this.list = arr
          } else {
            this.$refs.indexList.forceUpdate()
          }
        } else {
          arr.map(v => {
            let index = this.list.findIndex(v2 => v2.name === v.name)
            if (index > -1) {
              this.list[index].items.push(...(v.items))
            } else {
              this.list.push(v)
            }
          })
          if (len) {
            if (len === this.size) {
              setTimeout(() => {
                this.$refs.indexList.forceUpdate(true)
              }, 30)
            } else {
              console.log('nodata')
              this.isNoMore = true
              setTimeout(() => {
                this.$refs.indexList.forceUpdate(true, true)
              }, 30)
            }
          } else {
            this.$refs.indexList.forceUpdate()
            this.isNoMore = true
          }
          this.from++
        }
      })
    }
  }
}
</script>
