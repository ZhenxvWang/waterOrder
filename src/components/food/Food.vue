<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>Index</el-breadcrumb-item>
      <el-breadcrumb-item>Food management</el-breadcrumb-item>
      <el-breadcrumb-item>List of food</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="30">
        <el-col :span="6">
          <el-input v-model="queryInfo.query" placeholder="Please enter the food name" class="input-serch"></el-input>
        </el-col>
        <el-col :span="3">
            <el-button type="primary" icon="el-icon-search" @click="searchGoods">Search</el-button>
        </el-col>
        <el-col :span="4">
            <el-button type="primary" icon="el-icon-plus" @click="toAdd">Add</el-button>
        </el-col>
        <!-- <el-col :span="16">
          <el-button type="primary" @click="addShop">Add</el-button>
        </el-col> -->
      </el-row>
      <!-- 列表形式 -->
      <el-table v-loading="loading" :data="foods" stripe border style="width: 100%">
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="fname" label="The name of the food" align="center"></el-table-column>
        <el-table-column prop="fprice" label="Sale price" align="center">
           <template slot-scope="scope">
            <div>￥{{scope.row.fprice.toFixed(2) }}</div>
          </template>
        </el-table-column>
        <el-table-column prop="ftime" label="Shelf time" align="center">
           <template slot-scope="scope">
            <div>{{scope.row.ftime | timeflow }}</div>
          </template>
        </el-table-column>
        <el-table-column prop="tid" label="Food type" align="center">
          <template slot-scope="scope">
            <div v-if="scope.row.tid==1">Hot dishes</div>
           <!--  <div v-if="scope.row.tid==2">特色菜品</div> -->
            <div v-if="scope.row.tid==3">Recommendation</div>
           <!-- <div v-if="scope.row.tid==4">香辣菜品</div> -->
          </template>
        </el-table-column>
        <el-table-column label="Operation" align="center">
          <template slot-scope="scope">
            <el-button @click="toEdit(scope.row)" type="primary" icon="el-icon-edit" circle></el-button>
            <el-button @click="foodDelete(scope.row.fid)" type="danger" icon="el-icon-delete" circle ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页底部 -->
       <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pageIndex"
        :page-sizes="[5,10]"
        :page-size="queryInfo.pageSize"
        layout="total, sizes, prev, pager, next"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 修改商品的对话框 -->
    <el-dialog
      title="Modify the goods"
      :visible.sync="editShow"
      width="50%"
      center
      @close="editClose"
    >
      <!-- 内容主题区域 -->
      <el-form :model="editFood" :rules="editRules" ref="editRules" label-width="150px">
        <el-form-item label="The name of the food" prop="fname">
          <el-input v-model="editFood.fname"></el-input>
        </el-form-item>
        <el-form-item label="Sale price" prop="fprice">
          <el-input type="number" v-model="editFood.fprice"></el-input>
        </el-form-item>
        <el-form-item label="Food type" prop="tid">
          <el-select v-model="editFood.tid" placeholder="Please select">
            <el-option
              v-for="item in options"
              :key="item.tid"
              :label="item.tname"
              :value="item.tid">
            </el-option>
          </el-select>
        </el-form-item>
         <el-form-item label="Food description" prop="fdesc">
            <el-input
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 4}"
            placeholder="Please enter the content"
            v-model="editFood.fdesc">
          </el-input>
        </el-form-item>
      </el-form>

      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editClose('editRules')">Cancel</el-button>
        <el-button type="primary" @click="editCommit('editRules')">Confirm the change</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import axios from 'axios'
import Vue from 'vue'

//过滤器，对时间格式进行转换
Vue.filter('timeflow',date=>{
  if(!date) return ''
  const da = new Date(date);
  let time = da.getFullYear();
  const month = da.getMonth()+1;
  const day = da.getDate();
  //针对月份，天进行拼接处理
  if(month<10)time=time+'-0'+month;
  else time = time+'-'+month;
  if(day<10)time=time+'-0'+day;
  else time = time+'-'+day;
  return time;
})
export default {
  data() {
     var valiNumberPass1 = (rule, value, callback) => {//正整数
            if (value === '') {
                callback(new Error('Please enter selling price price'));
            } else if (isNaN(parseFloat(value))) {
                callback(new Error('Please enter the correct value'));
            } else {
                callback();
            }
     };
    return {
      loading:true,
      searchValue:'',
      foods:[],
      options:[],
      editFood:{
        fid:'',
        fname:'',
        fprice:'',
        fdesc:'',
        tid:'',
      },
      editShow:false,
      queryInfo:{
        query:'',
        pageIndex:1,
        pageSize:5
      },
      total:0,
      editRules:{
        fname:[{ required: true, message: 'Please enter the food name', trigger: 'blur' }],
        fprice:[{ required: true, validator:valiNumberPass1, trigger: "blur" }],
        fdesc:[{ required: true, message: 'Please enter food description', trigger: 'blur'}],
      }
      
    }
  },
  methods: {
    searchGoods(){
      this.getFoods();
    },
    toEdit(val){
      //拷贝
      this.editFood.fid = val.fid;
      this.editFood.fname = val.fname;
      this.editFood.fprice = val.fprice;
      this.editFood.fdesc = val.fdesc;
      this.editFood.tid = val.tid;
      //请求获取食品类型
      this.getFoodType();
      this.editShow = true;
    },
    editClose(){
      this.editShow = false;
      //重置规则
      // this.$refs[formName].resetFields(); 
     //会报错
     //原因是：mouted加载数据以后，隐藏的弹出框并没有编译渲染进dom里面
     //。所以click弹出的时候$refs并没有获取到dom元素导致 ‘resetFields’ of undefined
      this.$nextTick(()=>{
        this.$refs.editRules.resetFields();
      });
    },
    toAdd(){
      this.$router.push('/addfood');
    },
    checkGname(){
      this.editFood.fname
    },
    editCommit(formName){
      this.$refs[formName].validate(valid => {
        if(valid){
          //请求
          axios.post('/api/order/admin/updateFoodBase',this.editFood).then(res=>{
            if(res.data==1){
              this.$message.success('Modified successfully!');
              this.getFoods();
            }else{
              this.$message.error('Modification failed!');
            }
            this.editShow = false;
          })
        }else{
          return false;
        }
      })

      
    },
    handleSizeChange(val){
      this.queryInfo.pageSize = val;
      this.getFoods();
    },
    handleCurrentChange(val){
      this.queryInfo.pageIndex = val;
      this.getFoods();
    },
    //根据id删除食品
   async foodDelete(fid){
      const re = await this.$confirm('This operation will permanently delete the food information. Do you want to continue?', 'prompt', {
          confirmButtonText: 'Determine',
          cancelButtonText: 'Cancel',
          type: 'warning'
      }).catch(err=>err)
      if(re !='confirm'){
        this.$message.info('Cancelled deletion!')
      }else{
        axios.get("api/order/admin/deleteFood?fid="+fid).then(res=>{
          if(res.data==1){
            this.$message.success('Deleted successfully!')
            this.getFoods();
          }else{
            this.$message.error('Delete failed!')
          }
        })
      }
    },
    //获取所有食品类型
    getFoodType(){
      axios.get('/api/order/admin/getTypes').then(res=>{  
          this.options = res.data;
      })
    },
    //查询食品信息
    getFoods(){
      axios.post("api/order/admin/getFoods",this.queryInfo).then(res=>{
            this.loading = false;
            this.foods = res.data.list;
            this.total = res.data.count;
      })
    },
  },
  mounted() {
    //获取商品数据
    this.getFoods();
  },
}
</script>
<style scoped>
/* .input-serch{
  width: 130px;
} */
</style>