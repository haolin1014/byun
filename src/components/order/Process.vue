<template>
  <div class="content-box">
    <Row>
      <Col span="6">
            <Bread></Bread>
      </Col>
    </Row>
    <Row class="margin-10" style="margin-bottom:30px;">
      <Col span="10" offset="2">
       <Alert type="success">
           说明
           <template slot="desc">点击流程项，开启或关闭该流程，<span class="color1"></span> 代表开启  <span class="color2"></span> 代表关闭 </template>
       </Alert>
      </Col>
    </Row>

    <Row class="margin-10">
      <Col span="20" offset="2">
        <span class="block"></span>
        <Avatar class="set" :style="{background: data[0].status==true ? color1:color2,color:data[0].status==true ? font1 : font2}" icon="briefcase" size="large"></Avatar><span class="block"></span>
        <Avatar class="set" :style="{background: data[1].status==true ? color1:color2,color:data[1].status==true ? font1 : font2}" @click.native="changeFlow('EXAMINE_MANAGER',data[1].status,'EXAMINE_ACCOUNTANT')" icon="person" size="large"></Avatar><span class="block"></span>
        <Avatar class="set" :style="{background: data[2].status==true ? color1:color2,color:data[2].status==true ? font1 : font2}" @click.native="changeFlow('EXAMINE_ACCOUNTANT',data[2].status,'EXAMINE_FINANCE')" icon="social-yen" size="large"></Avatar><span class="block"></span>
        <Avatar class="set" :style="{background: data[3].status==true ? color1:color2,color:data[3].status==true ? font1 : font2}" @click.native="changeFlow('EXAMINE_FINANCE',data[3].status,'SUBMITTED')" icon="android-bus" size="large"></Avatar><span class="block"></span>
        <Avatar class="set" :style="{background: data[4].status==true ? color1:color2,color:data[4].status==true ? font1 : font2}" icon="android-done-all" size="large"></Avatar>
        <span class="block"></span>
      </Col>
      <Col span="20" offset="2">
        <span class="des">业务员待审核</span>
        <span class="des">经理待审核</span>
        <span class="des">财务审核</span>
        <span class="des">仓管待审核</span>
        <span class="des">财务确认</span>
      </Col>
    </Row>
    

    <Spin class="demo-spin-col" v-if="spin">
        <Icon type="load-c" size=25 class="demo-spin-icon-load"></Icon>
        <div>Loading</div>
    </Spin>

  </div>
</template>

<script>
import axios from 'axios'
import {URL} from '@/api/config.js'
export default {
  data () {
    return {
      date:'',
      spin:false,
      color1:'rgb(76,184,148)',
      font1:'#fff',
      color2:'#fff',
      font2:'rgb(76,184,148)',
      data:[
        {status:false},
        {status:false},
        {status:false},
        {status:false},
        {status:false},
      ],
    }
  },
  mounted(){
    // 添加响应拦截器
    axios.interceptors.request.use(function (config) {
        this.spin = true;
        return config;
      }.bind(this), function (error) {
        // 对请求错误做些什么
        return Promise.reject(error);
      });
    axios.interceptors.response.use(function (response) {
        this.spin = false;
        if(response.data.errorCode==401){
          this.$Message.error('登录超时,请重新登录');
          setTimeout(()=>{
            this.$router.replace('/login');
          },2000); 
        }
        return response;
      }.bind(this), function (error) {
        // 对响应错误做点什么
        return Promise.reject(error);
      });

    this.dataInit();    

  },
  methods:{
    dataInit(){
      axios.get(URL+'orderflow',{}).then(function(res){
        if(res.data.errorCode==401){
          this.$Message.error('登录超时,请重新登录');
          setTimeout(()=>{
            this.$router.replace('/login');
          },2000); 
        }

        this.data = res.data.data;
        var length = this.data.length;
        if(length<5){
          for(var i = 0 ; i<5-length; i++){
            this.data.push({
              status:false
            });
          }
        }

      }.bind(this)).catch(function(error){
        console.log(error);
      });

    },
    changeFlow(orderStatus,status,nextStatus){
      var params = new URLSearchParams();
      params.append('orderStatus', orderStatus);
      params.append('nextStatus', nextStatus);
      params.append('status', !status);

      axios.post(URL+'orderflow',params).then(function(res){
        if(res.data.errorCode!=200){
          this.$Message.error(res.data.moreInfo);
        }else{
          this.$Message.success('请求成功');
          this.dataInit();
        }
      }.bind(this)).catch(function(error){
        this.$Message.error('请求失败');
      }.bind(this));
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.block{
  display: inline-block;
  width: 100px;
  height: 1px;
  background-color: #ccc;
  margin-top: -1px;
}
.set{
  cursor: pointer;
}
.des{
  display: inline-block;
  width: 100px;
  text-align: center;
  margin:30px 0;
  margin-left: 50px;
  font-weight: bold;
}
.color1{
  display: inline-block;
  width: 10px;
  height: 10px;
  background-color: rgb(76,184,148);
}
.color2{
  display: inline-block;
  width: 10px;
  height: 10px;
  background-color: #fff;
}
</style>
