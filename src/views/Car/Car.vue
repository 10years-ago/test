<template>
  <div class="Car">
    <div class="car-body">
      <el-form :inline="true" :model="selectForm" label-position="top" style="margin-top:50px">
        <el-col :span="3">
          <el-form-item>
            <el-input @input="select()" v-model="selectForm.carNum" placeholder="车牌号" clearable></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="3">
          <el-form-item>
            <el-input @input="select()" v-model="selectForm.name" placeholder="使用人" clearable></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="3">
          <el-form-item>
            <el-input @input="select()" v-model="selectForm.frameNum" placeholder="车架号" clearable></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="4">
          <el-form-item>
            <el-date-picker 
            type="date" 
            placeholder="注册日期" 
            v-model="selectForm.registerDate" 
            style="width: 100%;"
            @input="select()"
            clearable
            >
            </el-date-picker>
          </el-form-item>
        </el-col>
        <el-col :span="4">
          <el-form-item>
            <el-date-picker 
            type="date" 
            placeholder="年审日期" 
            v-model="selectForm.issueDate" 
            style="width: 100%;"
            @input="select()"
            clearable
            >
            </el-date-picker>
          </el-form-item>
        </el-col>
        <el-col :span="4">
          <el-form-item>
            <el-input @input="select()" v-model="selectForm.carState" clearable placeholder="车辆状态"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="3">
          <el-button type="primary" style="margin-left:30px" @click="carAdd()">车辆添加</el-button>
        </el-col>
      </el-form>
      <select-table 
        :tableData="tableData" 
        :loading="loading"
        @isRegisterstatus="isRegisterstatus" 
        @tableUpdate="tableUpdate" 
        @tableDel="tableDel"/>
      <el-pagination
        class="pagination"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange" :current-page="currentPage"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    <el-dialog title="车辆添加" :visible.sync="dialogFormVisible" :close-on-click-modal='false'>
      <el-upload
        class="avatar-uploader"
        action="fakeaction"
        ref="upload"
        :show-file-list="false"
        :http-request="picload"
        drag
        multiple
        v-loading="carLoading"
        element-loading-text="拼命识别中"
        element-loading-spinner="el-icon-loading"
        >
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将行驶证文件拖到此处，或<em>点击上传</em><p>只能上传jpg/png文件，且不超过4MB</p></div>
      </el-upload>
      <el-form :model="formData" :rules="rules" v-if="exgClear" ref="formData" label-position="left" label-width="100px">
        <el-form-item label="车牌号" prop="carNum">
          <el-input v-model="formData.carNum" placeholder="车牌号" clearable></el-input>
        </el-form-item>
        <el-form-item label="使用人" prop="name">
          <el-input v-model="formData.name" placeholder="使用人" clearable></el-input>
        </el-form-item>
        <el-form-item label="车架号" prop="frameNum">
          <el-input v-model="formData.frameNum" placeholder="车架号" clearable></el-input>
        </el-form-item>
        <el-form-item label="注册日期" prop="registerDate">
          <el-date-picker 
          type="date" 
          placeholder="选择日期" 
          v-model="formData.registerDate" 
          style="width: 100%;"
          clearable
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="年审日期" prop="annualReview">
          <el-date-picker 
          type="date" 
          placeholder="选择日期" 
          v-model="formData.annualReview" 
          style="width: 100%;"
          clearable
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="是否年审" prop="registerstatus">
           <el-select v-model="formData.registerstatus" placeholder="一年内是否年审" style="width: 100%;">
            <el-option label="是" value="yes"></el-option>
            <el-option label="否" value="no"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="车辆使用情况">
          <el-input v-model="formData.carState" clearable></el-input>
        </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="change('formData')">提 交</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import SelectTable from './components/CarTable'
//正则
import { isCarId } from 'assets/js/regexp'
//这里面放着表格的导出与翻页方法
import {tableMixin} from 'assets/js/tableMixin.js'
export default {
  name: 'Car',
  components:{SelectTable},
  mixins:[tableMixin],
  data(){
    return {
      selectForm: {
        carNum: '',
        name: '',
        frameNum:'',
        registerDate:'',
        issueDate:'',
        carState:'',
        school:this.$store.state.school
      },
      formData: {
        id:'',
        carNum: '',
        name: '',
        frameNum:'',
        registerDate:'',
        annualReview:'',
        carState:'',
        registerstatus:'',
        school:this.$store.state.school
      },
      dialogFormVisible:false,
      tableData:[],
      rules:{
        carNum:[
          {required:true,message:'请输入车牌号'},
          {validator:isCarId,message:'请输入正确车牌号'}
        ],
        name:{required:true,message:'请输入使用人'},
        frameNum:{required:true,message:'请输入车架号'},
        registerDate:{required:true,message:'请选择输入日期'},
        annualReview: {required:true,message:'请选择年审日期'},
        registerstatus:{required:true,message:'填写今年是否已经年审'}
      },
      loading:false,
      exgClear:true,//是否显示form表单，用来掩盖关闭窗口后rules的验证问题
      carLoading:false
    }
  },
  watch:{
    dialogFormVisible(val){//关闭dialog窗口清空里面数据
      if(!val){
        this.exgClear = false
        for(let key in this.formData){
        key !== 'school'? this.formData[key] = '' : ''
        }
        setTimeout(()=>{
          this.exgClear = true
        },500)
      }
    }
  },
  methods:{
    change(val){//添加和修改
      this.$refs[val].validate((valid) => {
        if (valid) {//判断form里面数据是否正确
          //添加
          if(this.formData.id == ''){//看是否有id判断当前是修改还是添加
            this.dateChange(this.formData)
            this.axios.post(`${process.env.VUE_APP_fank}/carInsert`,this.formData,{timeout:10000})
            .then(res => {
              if(res.data.status == 'ok'){
                this.$alert('录入成功','温馨提示',{
                  confirmButtonText:'确定',
                    callback: action => {
                    this.dialogFormVisible = false
                    this.select()
                  }
                })
              }else if(res.data.status == 'over'){
                this.$alert('账号已到期，如想继续使用，请联系供应商续费。','温馨提示',{
                  confirmButtonText:'确定',
                    callback: action => {
                  }
                })
              }else{
                this.$alert('录入失败，车牌号或车架号不可重复','温馨提示',{
                  confirmButtonText:'确定',
                    callback: action => {
                  }
                })
              }
            })
            .catch(err => {
              this.$alert(`网络错误，请检查网络或联系供应商。错误代码：${err}`,'温馨提示',{
                confirmButtonText:'确定',
                callback: action => {
                }
              })
            })
          }else{
            //修改
            this.dateChange(this.formData)
            this.axios.post(`${process.env.VUE_APP_fank}/carUpdate`,this.formData,{timeout:10000})
            .then(res => {
              if(res.data.status == 'ok'){
                this.$alert('修改成功','温馨提示',{
                  confirmButtonText:'确定',
                    callback: action => {
                      this.dialogFormVisible = false
                  }
                })
                //修改后页面及时改变，而不用再请求一次服务器--start
                let tableDataLength = this.tableData.length
                for(let i = 0; i < tableDataLength; i++){
                  if(this.tableData[i].id == this.formData.id){
                    this.$set(this.tableData, i, JSON.parse(JSON.stringify(this.formData)))
                    return 
                  }
                }
                //--end
              }else if(res.data.status == 'over'){
                this.$alert('账号已到期，如想继续使用，请联系供应商续费。','温馨提示',{
                  confirmButtonText:'确定',
                    callback: action => {
                  }
                })
              }else{
                this.$alert('修改失败，车牌号或车架号已存在','温馨提示',{
                  confirmButtonText:'确定',
                    callback: action => {
                  }
                })
              }
            })
            .catch(err => {
              this.$alert(`网络错误，请检查网络或联系供应商。错误代码：${err}`,'温馨提示',{
                confirmButtonText:'确定',
                callback: action => {
                }
              })
            })
          }
          }else{
            this.$alert('重要选项不能为空，且选项的条件要匹配','温馨提示',{
              confirmButtonText:'确定',
                callback: action => {
              }
            })
          }
        })
    },
    dateChange(val){//日期转换
      //判断注册日期是否为空，不为空转yyyy-MM-dd，为空就变为''
      if(val.registerDate !== null && String(val.registerDate).length > 15){
        var y = val.registerDate.getFullYear(); 
        var m = val.registerDate.getMonth() + 1; 
        m = m < 10 ? '0' + m : m; 
        var d = val.registerDate.getDate(); 
        d = d < 10 ? ('0' + d) : d; 
        val.registerDate = y + '-' + m + '-' + d
      }else if(String(val.registerDate).length == 10){
        
      }else{
        val.registerDate = ''
      }
      //同理和上，这是判断年审日期
      if(val.issueDate !== null  && String(val.issueDate).length > 15){
        var y = val.issueDate.getFullYear(); 
        var m = val.issueDate.getMonth() + 1; 
        m = m < 10 ? '0' + m : m; 
        var d = val.issueDate.getDate(); 
        d = d < 10 ? ('0' + d) : d; 
        val.issueDate = y + '-' + m + '-' + d
      }else if(String(val.issueDate).length == 10){
        
      }else{
        val.issueDate = ''
      }
    },
    carAdd(){//打开车辆添加的窗口
      for(let key in this.formData){
        key !== 'school'? this.formData[key] = '' : ''
      }
      this.dialogFormVisible = true
    },
    tableUpdate(val){//打开车辆修改窗口，里面和车辆添加一样
      this.dialogFormVisible = true
      this.formData = JSON.parse(JSON.stringify(val)) 
    },
    tableDel(val){//删除数据
      let deleteId = {
        id:val
      }
      this.axios.post(`${process.env.VUE_APP_fank}/carDelete`,deleteId,{
        timeout:10000
      })
      .then(res => {
        if(res.data.status == 'ok'){
          this.$alert('删除成功','温馨提示',{
            confirmButtonText:'确定',
              callback: action => {
                this.FormVisibleAdd = false
            }
          })
          let tableDataLength = this.tableData.length
          for(let i = 0; i < tableDataLength; i++){
            if(this.tableData[i].id == val){
              this.$delete(this.tableData, i)
              return 
            }
          }
        }else if(res.data.status == 'over'){
          this.$alert('账号已到期，如想继续使用，请联系供应商续费。','温馨提示',{
            confirmButtonText:'确定',
              callback: action => {
            }
          })
        }else{
          this.$alert('删除失败','温馨提示',{
            confirmButtonText:'确定',
              callback: action => {
            }
          })
        }
      })
      .catch(err => {
        this.$alert(`网络错误，请检查网络或联系供应商。错误代码：${err}`,'温馨提示',{
        confirmButtonText:'确定',
          callback: action => {
        }
        })
      })
    },
    isRegisterstatus(val){ //确认年审
      let data = {
        id:val.id,
        issueDate:val.issueDate,
        school:this.$store.state.school
      }
      console.log(val)
      this.axios.post(`${process.env.VUE_APP_fank}/confirm`,data,{
        timeout:10000
      })
      .then(res => {
        if(res.data.status == 'ok'){
          this.$alert('确认成功','温馨提示',{
            confirmButtonText:'确定',
              callback: action => {
                this.dataPost()
            }
          })
        }else if(res.data.status == 'over'){
          this.$alert('账号已到期，如想继续使用，请联系供应商续费。','温馨提示',{
            confirmButtonText:'确定',
              callback: action => {
            }
          })
        }else{
        this.$alert('确认失败','温馨提示',{
          confirmButtonText:'确定',
            callback: action => {
          }
        })
        }
      })
      .catch(err => {
        this.$alert(`网络错误，请检查网络或联系供应商。错误代码：${err}`,'温馨提示',{
        confirmButtonText:'确定',
          callback: action => {
        }
        })
      })
    },
    select(){//input框查询
      this.currentPage = 1
      this.dataPost()
    },
    dataPost(){//查询的函数
      this.loading = true
      this.$debounce(() =>{//每个查询都有防抖
      this.dateChange(this.selectForm)//日期转换,至于为什么要这么转，问后台小哥
      //浅拷贝selectForm，然后把页数(page)，每页条数(pageSize)一起放入filterData中
      let filterData = Object.assign({},this.selectForm)
      filterData['page'] = this.currentPage
      filterData['pageSize'] = this.pageSize
      this.axios.post(`${process.env.VUE_APP_fank}/carSelect`,filterData,{
        timeout: 10000
      })
      .then(res => {
          this.loading = false
          //获取总条数
          this.total = res.data.all
          //获取表单数据
          this.tableData = res.data.list
          if(res.data.status == 'over'){
            this.$alert('账号已到期，如想继续使用，请联系供应商续费。','温馨提示',{
              confirmButtonText:'确定',
                callback: action => {
              }
            })
          }else if(res.data.number > 0){
            this.$notify({
              title: '警告',
              message: `有${res.data.number}辆车准备年审，请留意`,
              type: 'warning',
              offset: 80
            });
          }
      })
      .catch(err => {
        this.loading = false
          this.$alert(`网络错误，请检查网络或联系供应商。错误代码：${err}`,'温馨提示',{
            confirmButtonText:'确定',
              callback: action => {
            }
          })
        })
      },500)//debounce-end
    },
    picload(item){//图片上传后台判断身份证信息后返回数据
      let file = item.file
      //判断格式
      if (file.type == 'image/jpeg' || file.type == 'image/png' || file.type == 'image/JPG') {
      var  isJPG =  true
      } else {
        isJPG =  false
        this.$message.error('上传产品图片只能是 JPG/PNG/JPEG 格式!')
        return
      }

      //判断大小
      const isLt2M = file.size / 1024 / 1024 < 4
      if (!isLt2M) {
        this.$message.error('上传产品图片大小不能超过 4MB!')
        return
      }
      let param = new FormData()
      param.append('img',file)
      param.append('school',this.$store.state.school)
      param.append('perm',this.$store.state.perm)
      let config = {
        headers:{'Content-Type':'multipart/form-data'}
      }
      this.carLoading = true
      this.axios.post(`${process.env.VUE_APP_fank}/carkow`,param,config)
      .then(res =>{
        if(res.data.status == 'over'){
          this.$alert('账号已到期，如想继续使用，请联系供应商续费。','温馨提示',{
            confirmButtonText:'确定',
              callback: action => {
              this.carLoading = false
            }
          })
        }else if(res.data.status == 'picBad'){
           this.$alert('图片异常无法识别,不能为红底或者有反光，请再次重试。','温馨提示',{
            confirmButtonText:'确定',
              callback: action => {
              this.carLoading = false
            }
          })
        }else{
          this.formData.carNum = res.data.carNum
          this.formData.name = res.data.name
          this.formData.frameNum = res.data.frameNum
          this.formData.registerDate = res.data.registerDate
          this.formData.annualReview = res.data.annualReview
          this.carLoading = false
        }
      })
      .catch(err =>{
        this.$alert(`网络错误，请检查网络或联系供应商。错误代码：${err}`,'温馨提示',{
          confirmButtonText:'确定',
          callback: action => {
          }
        })
      })
      // }
    }
  },
  created(){
    this.dataPost()
  }
}
</script>
<style scoped lang="scss">
.car-body{
  width: 1280px;
  margin: 0 auto;
  .el-form-item{
    text-align: center;
  }
  .el-pagination{
    text-align: center;
  }
  ::v-deep .el-dialog{
    margin-top: 10vh!important;
    width: 30%;
    min-width: 500px;
    .el-dialog__body{
      text-align: center;
      padding: 0;
      .el-form{
        padding-top: 30px;
        width: 65%;
        margin: 0 auto;
      }
    }
    .dialog-footer{
      text-align: center;
    }
  }
}
</style>