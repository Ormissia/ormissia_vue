<!--类型页面-->
<template>
  <canvas ref="canvas" style="width: 100%" :width="width" :height="height"></canvas>
</template>
<script>
import WordCloud from 'wordcloud'

export default {
  name: 'Tag',
  data () {
    return {
      // 页面宽度高度
      width: 0,
      height: 0,
      // 用于显示的标签列表
      tagList: [],
      // 用于查询标签列表的参数对象
      queryInfo: {
        // 搜索框的内容
        queryStr: '',
        // 当前页数
        pageNum: 1,
        // 当前每页显示多少条数据
        pageSize: 9999,
        // 向后端发送请求携带的参数，查询未删除的标签，2
        isDeleted: 2
      },
      canvas: document.getElementById('canvas'),
      options: {
        list: [['JAVA', 45], ['Spring', 21]], // 数据list
        wait: 1, // 绘制下一项的时间间隔，单位：毫秒
        gridSize: 5, // 密集程度 数字越小越密集
        weightFactor: 20, // 字体大小=原始大小*weightFactor
        minFontSize: 1, // 最小字号
        maxFontSize: 10, // 最大字号
        fontWeight: 'normal', // 字体粗细
        fontFamily: 'Times, serif', // 字体
        color: 'random-dark', // 字体颜色 'random-dark' 或者 'random-light'
        backgroundColor: 'white', // 背景颜色
        rotationSteps: 2, // 强制使用定义的角度数 在-90°/ 90°范围内将值设置为等于2，表示将仅使用-90、0或90
        // rotateRatio: 1, // 字体倾斜(旋转)概率，1代表总是倾斜(旋转)
        shape: 'diamond'
      }
    }
  },
  mounted () {
    // 调用获取数据的方法
    this.selectTagByPage()
  },
  created () {
    // 页面创建时获取页面高度
    this.sizeHandle()
  },
  methods: {
    // 滚动监听
    sizeHandle () {
      this.height = window.innerHeight// 获取页面高度
      this.width = 2 / 3 * (window.innerWidth + 153)// 获取页面宽度
    },
    // 按照页面分页获取博客列表
    async selectTagByPage () {
      try {
        const { data: res } = await this.$http.post('tag/selectTagByPage', this.$qs.parse(this.queryInfo))
        // 判断列表长度大于0
        if (res.data.dataList.length > 0) {
          this.createOption(res.data.dataList)
        }
      } catch (e) {
        // 保存失败，输出错误提示
        this.$rootMessage({
          showClose: true,
          message: e,
          type: 'error'
        })
      }
    },
    //
    createOption (dataList) {
      // 生成数据list
      const temporaryTagList = []
      for (let i = 0; i < dataList.length; i++) {
        // 如果长度为0，则赋值为1，防止不显示
        const itemList = [dataList[i].tagName, dataList[i].articles.length === 0 ? 1 : dataList[i].articles.length]
        temporaryTagList.push(itemList)
      }

      // 重复添加直至总数超过100条,或者添加次数大于100
      for (let i = 0; i < 100 && this.tagList.length < 100; i++) {
        this.tagList = this.tagList.concat(temporaryTagList)
      }

      // 计算weightFactor的值（控制最大字体为100），字体大小=原始大小*weightFactor
      let maxSize = this.tagList[0][1]
      if (maxSize === 0) {
        maxSize = 2
      }
      let minSize = dataList[dataList.length - 1].articles.length
      if (minSize === 0) {
        minSize = 1
      }
      const fontSize = 100 / maxSize

      // 设置option
      this.options.list = this.tagList// 数据list
      this.options.weightFactor = fontSize
      this.options.maxFontSize = maxSize// 取第一个标签的文章数量
      this.options.minFontSize = minSize// 最小字号,取最后一个标签的文章数量（后端按文章数量排序）

      // 展示
      WordCloud(this.$refs.canvas, this.options)
    }
  }
}
</script>

<style lang="less" scoped>
.card-top {
  margin: 20px 0;
}
</style>
