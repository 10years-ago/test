<template>
  <div class="home-page">
    <charts 
    :dateList="dateList" 
    v-if="dataOk" 
    :years="years"
    :nowYear="nowYear"
    @chartChange="chartChange"/>
    <main-data :mainData="mainData"/>
  </div>
</template>

<script>
import charts from './ChildComponents/charts'
import MainData from './ChildComponents/MainData'
export default {
  name: 'HomePage',
  components:{MainData,charts},
  data(){
    return {
      years:[],//select框的所有年份
      mainData:{},//今日报名，剩余时间，table内的数据
      dateList:[],//图表内的数据
      dataOk:false,//用v-if判定，数据加载完过后再加载charts
      nowYear:Number,//现在图表的年份 or 选择过后的年份
    }
  },
  methods:{
    chartChange(value){//点击年份时间改变
      this.dataPost(value)
    },
    dataPost(value){//请求主页的所有数据
    //现在图表的年份 or 选择过后的年份，得转为number，不然会报错
    this.nowYear = value || Number(window.localStorage.nowTime.split(" ")[0].split('-')[0])
    let data = {//传入学校和当前所选择年份
      school:this.$store.state.school,
      year:this.nowYear,
    }
    this.axios.post(`${process.env.VUE_APP_fank}/home`,data,{timeout:10000})
    .then(res => {
      if(res.data.days < 0){
        this.$alert(`此账号已过期`,'温馨提示',{
          confirmButtonText:'确定',
          callback: action => {
            localStorage.clear()
            this.$router.push({path:'/Login'})
        }
      })
      }
      //今日添加
      let today = res.data.today
      //图表内的日期
      this.years = res.data.yearList
      this.dateList = res.data.list
      //公告
      let notice = res.data.textList
      //放入对象传给主页下面的内容
      this.mainData = {
        lastDay:res.data.nowTime,
        days:res.data.days,
        today,
        notice
      }
      this.dataOk = true
    })
    .catch(err => {
        this.$alert(`网络错误，请检查网络或联系供应商。错误代码：${err}`,'温馨提示',{
          confirmButtonText:'确定',
          callback: action => {
        }
      })
    })
    }
  },
  created(){
    this.dataPost()
  }
}
</script>
<style scoped lang="scss">

</style>