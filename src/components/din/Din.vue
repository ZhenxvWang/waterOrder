<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>Index</el-breadcrumb-item>
      <el-breadcrumb-item>Order management</el-breadcrumb-item>
      <el-breadcrumb-item>Order list</el-breadcrumb-item>
    </el-breadcrumb>
     <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="5">
          <el-input placeholder="Please enter your name" v-model="queryInfo.query" clearable></el-input>
        </el-col>
        <el-col :span="3">
          <el-button type="primary" icon="el-icon-search" @click="search">Search</el-button>
        </el-col>
      </el-row>
      <!-- 订单列表区 -->
      <el-table  v-loading="loading" :data="orders" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="Order meals" align="center">
          <template slot-scope="scope">
            {{scope.row.wait}}
          </template>
        </el-table-column>
         <el-table-column label="Buyer" align="center">
            <template slot-scope="scope">
            {{scope.row.user.nike}}
          </template>
         </el-table-column>
          <el-table-column label="Buyer telephone number" align="center">
            <template slot-scope="scope">
            {{scope.row.user.phone}}
          </template>
         </el-table-column>
         <el-table-column label="Order status" align="center">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.wstate==0" type="warning">Equipment eat</el-tag>
              <el-tag v-if="scope.row.wstate==1" type="danger">To take food</el-tag>
              <el-tag v-if="scope.row.wstate==2" type="success">Has been completed</el-tag>
          </template>
         </el-table-column>
         <el-table-column label="Buy time" align="center">
            <template slot-scope="scope">
            {{scope.row.otime | timeflow}}
          </template>
         </el-table-column>
        <el-table-column label="Operation" align="center">
          <template slot-scope="scope">
            <el-button
              @click="toEdit(scope.row.wait,scope.row.wstate)"
              type="primary"
              icon="el-icon-edit"
              circle
            ></el-button>
            <el-button type="warning" @click="toOrderDetail(scope.row.wait)" icon="el-icon-tickets" circle ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pageIndex"
        :page-sizes="[5, 10]"
        :page-size="queryInfo.pageSize"
        layout="total, sizes, prev, pager, next"
        :total="total"
      >
      </el-pagination>
    </el-card> 
    <el-dialog title="Modify reservation information" :visible.sync="editOrderShow" width="80%" center @close="editOrderShow=false" >
      <!-- 内容主题区域 -->
      <el-table :data="ors" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="The name of the food" align="center">
          <template slot-scope="scope">
            {{scope.row.card.food.fname}}
          </template>
        </el-table-column>
         <el-table-column label="Buy copies" align="center">
            <template slot-scope="scope">
            {{scope.row.card.num}}
          </template>
         </el-table-column>
         <el-table-column label="Order status" align="center">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.wstate==0" type="warning">Equipment eat</el-tag>
              <el-tag v-if="scope.row.wstate==1" type="danger">To take food</el-tag>
              <el-tag v-if="scope.row.wstate==2" type="success">Has been completed</el-tag>
          </template>
         </el-table-column>
         <el-table-column label="Buy time" align="center">
            <template slot-scope="scope">
            {{scope.row.otime | timeflow}}
          </template>
         </el-table-column>
          <el-table-column label="operation" align="center">
          <template slot-scope="scope">
            <el-select @change="changeItem(scope.row)" v-model="scope.row.wstate">
              <el-option
                v-for="item in states"
                :key="item.ostate"
                :label="item.value"
                :value="item.ostate">
              </el-option>
            </el-select>
          </template>
        </el-table-column>
      </el-table>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-radio-group @change="changes()" v-model="radio">
          <el-radio :label="0">Equipment eat,</el-radio>
          <el-radio :label="1">To take food</el-radio>
          <el-radio :label="2">Has been completed</el-radio>
        </el-radio-group>
      </span>
    </el-dialog>


     <el-dialog title="Overall order information" :visible.sync="orderDetailShow" width="80%" center @close="orderDetailShow=false" >
      <!-- 内容主题区域 -->
      <el-table :data="ors" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="The name of the food" align="center">
          <template slot-scope="scope">
            {{scope.row.card.food.fname}}
          </template>
        </el-table-column>
        <el-table-column label="Food price" align="center">
          <template slot-scope="scope">
            {{scope.row.card.food.fprice.toFixed(2)}}
          </template>
        </el-table-column>
         <el-table-column label="Buy copies" align="center">
            <template slot-scope="scope">
            {{scope.row.card.num}}
          </template>
         </el-table-column>
          <el-table-column label="Total price" align="center">
          <template slot-scope="scope">
            {{scope.row.card.tprice.toFixed(2)}}
          </template>
        </el-table-column>
         <el-table-column label="Order status" align="center">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.wstate==0" type="warning">Equipment eat</el-tag>
              <el-tag v-if="scope.row.wstate==1" type="danger">To take food</el-tag>
              <el-tag v-if="scope.row.wstate==2" type="success">Has been completed</el-tag>
          </template>
         </el-table-column>
         <el-table-column label="Name of Buyer" align="center">
            <template slot-scope="scope">
            {{scope.row.user.nike}}
          </template>
         </el-table-column>
         <el-table-column label="Buyer telephone number" align="center">
            <template slot-scope="scope">
            {{scope.row.user.phone}}
          </template>
         </el-table-column>
         <el-table-column label="Buy time" align="center">
            <template slot-scope="scope">
            {{scope.row.otime | timeflow}}
          </template>
         </el-table-column>
      </el-table>
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
    return {
      loading:false,
      orders:[],
      ors:[],
      total:0,
      radio: 0,
      orderWait:'',
      editOrderShow:false,
      orderDetailShow:false,
      checkedState:'',
      editOrderInfo:{
        ostate:'',
      },
      states:[
        {ostate:'0',value:'Equipment eat'},
        {ostate:'1',value:'To take food'},
        {ostate:'2',value:'Has been completed'}
      ],
      queryInfo:{
        query:'',
        pageIndex:1,
        pageSize:5
      }
    }
  },
  methods: {
    //获取所有订单信息
    getOrders(){
      axios.post('/api/order/admin/getOrdersPa',this.queryInfo).then(res=>{
        this.orders = res.data.list;
        this.total = res.data.count;
      })
    },
    handleSizeChange(val){
      this.queryInfo.pageSize = val;
      this.getOrders()
    },
    handleCurrentChange(val){
      this.queryInfo.pageIndex = val;
      this.getOrders()
    },
    search(){
      this.getOrders()
    },
    toEdit(wait,wstate){
      axios.get('/api/order/admin/getOrders?wait='+wait).then(res=>{
        this.orderWait = wait;
        this.radio = parseInt(wstate);
        this.ors = res.data;
        this.editOrderShow = true;
      })
      
    },
    changeItem(or){
      axios.get('/api/order/admin/updateOrder?oid='+or.oid+'&wstate='+or.wstate).then(res=>{
        if(res.data==1){
          this.getOrders();
        }else{
          this.$message.error('Failed to modify, please try again!');
        }
      })
    },
    changes(){
      console.log(this.orderWait,this.radio)
      axios.get('/api/order/admin/updateOrders?wait='+this.orderWait+'&wstate='+this.radio).then(res=>{
        if(res.data>0){
          this.getOrders();
          this.editOrderShow = false;
        }else{
          this.$message.error('Failed to modify, please try again!');
        }
      })
    },
    toOrderDetail(wait){
       axios.get('/api/order/admin/getOrders?wait='+wait).then(res=>{
        this.ors = res.data;
        this.orderDetailShow = true;
      })
    }
  },
  mounted(){
    this.getOrders();
  },
}
</script>
<style scoped>

</style>