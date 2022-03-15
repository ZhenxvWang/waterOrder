<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>Index</el-breadcrumb-item>
      <el-breadcrumb-item>Commodity management</el-breadcrumb-item>
      <el-breadcrumb-item>Commodity add</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-button type="primary" @click="toShop">Return to the list of items</el-button>
      <el-form :model="addFood" :rules="addRules" ref="addRules" label-width="150px">
        <el-form-item label="The name of the food" prop="fname">
          <el-input v-model="addFood.fname"></el-input>
        </el-form-item>
        <el-form-item label="Sale price" prop="fprice">
          <el-input type="number" v-model="addFood.fprice"></el-input>
        </el-form-item>
        <el-form-item label="Food description" prop="fdesc">
            <el-input
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 4}"
            placeholder="Please enter the content"
            v-model="addFood.fdesc">
          </el-input>
        </el-form-item>
         <el-form-item label="Food type" prop="tid">
          <el-select v-model="addFood.tid" placeholder="Please select">
            <el-option
              v-for="item in options"
              :key="item.tid"
              :label="item.tname"
              :value="item.tid">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Commodity images">
          <el-upload
            class="upload-demo"
            action="/api/order/admin/uploadImg"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :on-success="successUpload"
            :on-error="errorUpload"
            :file-list="fileList"
            :multiple="false"
            list-type="picture">
            <el-button size="small" type="primary">Click on the upload</el-button>
            <div slot="tip" class="el-upload__tip">Only JPG/PNG files can be uploaded, and the maximum size is 500kb</div>
          </el-upload>
        </el-form-item>
        <el-form-item label="  ">
           <el-button type="primary" @click="onFood">Add food</el-button>
        </el-form-item>
      </el-form>
      
    </el-card>
  </div>
</template>
<script>
import axios from 'axios';
export default {
  data() {
     var valiNumberPass1 = (rule, value, callback) => {//正整数
            if (value === '') {
                callback(new Error('Please enter market price'));
            } else if (isNaN(parseFloat(value))) {
                callback(new Error('Please enter the correct value'));
            } else {
                callback();
            }
     };
    return {
     addFood:{
       fname:'',
       fprice:'',
       tid:'',
       fdesc:'',
       fpicture:''
     },
     fileList:[],
     options:[],
     addRules:{
        fname:[{ required: true, message: 'Please enter the name of the product', trigger: 'blur' }],
        fprice:[{ required: true, validator:valiNumberPass1, trigger: "blur" }],
        fdesc:[{ required: true, message: 'Please enter the quantity of goods', trigger: 'blur'}],
     },


    }
  },
  methods: {
    toShop(){
      this.$router.push('/food');
    },
    //获取所有食品类型
    getFoodType(){
      axios.get('/api/order/admin/getTypes').then(res=>{  
          this.options = res.data;
      })
    },
     handlePreview(file, fileList){
       
     },
     handleRemove(file, fileList){
       axios.get('/api/order/admin/deleteUpload?img='+this.addFood.fpicture).then(res=>{
          this.$message.success('Removal successful!');
       })
     }, 
     successUpload(response, file, fileList){
      if(response.flag){
        this.addFood.fpicture = response.img;
        this.$message.success('Added successfully!');
      }
     },
     errorUpload(err, file, fileList){
        this.$message.console.error('Add failed!');
     },
     onFood(){
       if(this.addFood.fname!=''&&this.addFood.fprice!=null&&this.addFood.fdesc!=''){
         if(this.addFood.tid!=''){
          if(this.addFood.fpicture!=''){
            axios.post('/api/order/admin/addFood',this.addFood).then(res=>{
              if(res.data==1){
                this.$message.success("Added successfully!");
                this.$router.push('/food');
              }else{
                this.$message.error("Add failed!")
              }
            })
          }else{
            this.$message("Please upload pictures of food");
          }
        }else{
          this.$message("Please select a type");
        }
       }else{
         this.$message("Improve basic food information");
       }
     }
  },
  mounted(){
    this.getFoodType();
  }
}
</script>
<style lang="less" scoped>
.el-steps {
  margin: 15px 0;
  padding: 5px;
}
.el-step__title {
  font-size: 12px;
}
.el-tab-pane {
  border-bottom: black;
}
.el-form{
  width: 60%;
  margin-top: 20px;
  // /* border: 1px solid red; */
  // margin: 5px auto;
  // .el-form-item{
  //   margin: 50px 0px;
  // }
}
</style>