<template>
	<div>
		<Row>
		  <Col span="6">
		        <Bread></Bread>
		  </Col>
		</Row>
	
		<Row class="margin-10">
		  <Col span="12">
		    <div class="searchBox addBox">
		      <DatePicker type="daterange" placement="bottom-start" placeholder="选择日期" style="width: 200px" v-model="date"></DatePicker>
		      <Input v-model="searchStr" icon="ios-search" placeholder="公司名称/负责人手机号" style="width: 200px"></Input>
		      <Button type="primary" @click="search">查询</Button>
		    </div>
		  </Col>
		  <Col span="2" offset="8">
		    <div class="addBox">
		      <Button type="info" icon="plus" ><router-link tag="span" to="/bill/index/add">新增常用清单</router-link></Button>
		    </div>
		  </Col>
		</Row>
		
		<div class="layout-content-main">
		 <Table stripe border highlight-row :columns="columns1" :data="data1" size="small"></Table>
		 <div style="margin: 10px;overflow: hidden">
		     <div style="float: right;">
		         <Page :total="total" :current="current" @on-change="changePage"></Page>
		     </div>
		 </div>
		</div>


	</div>
</template>

<script>
import axios from 'axios'
import {URL} from '@/api/config.js'
export default {
  data () {
    return {
      newUser:{
        company:'',
        name:'',
        phone:'',
        address:'',
        credit:0,
        area:''
      },
      addModal:false,
      date:'',
      total:0,
      current:1,
      spin:false,
      searchStr:'',
      data1: [],
      columns1: [
                 {
                     title: '编号',
                     key: 'id'
                 },
                 {
                     title: '清单名称',
                     key: 'name'
                 },
                 {
                     title: '清单类型',
                     key: 'manager',
                 },
                 {
                     title: '金额',
                     key: 'phoneNumber',
                 },
                 {
                     title: '清单详情',
                     key: 'area',
                 },
                 {
                     title: '操作',
                     key: 'action',
                     width: 100,
                     align: 'center',
                     render: (h, params) => {
                         return h('div', [
                             h('Button', {
                                 props: {
                                     type: 'success',
                                     size: 'small'
                                 },
                                 style: {
                                     marginRight: '5px'
                                 },
                                 on: {
                                     click: () => {
                                         
                                     }
                                 }
                             }, '使用'),
                             h('Button', {
                                 props: {
                                     type: 'error',
                                     size: 'small'
                                 },
                                 style: {
                                     marginRight: '5px'
                                 },
                                 on: {
                                     click: () => {
                                         
                                     }
                                 }
                             }, '删除'),
                         ]);
                     }
                 }
             ],
    }
  },
  mounted(){
    
      this.total = 1;
      this.current = 1;
      // this.data1 = [
      //   {no:'123',company:'帮陪科技',name:'小王',phone:'12322321123',area:'江干',credit:1000,consume:1000,unfinishAccount:500,unfinishBill:2,date:'2017-10-11',status:1}
      // ];
  },
  methods:{
    changePage(n){
      this.spin = true;
      axios.get(URL+'order/paginate?page='+n)
        .then(function(res){
          var data = res.data;
          this.total = data.total;
          this.current = data.current_page;
          this.data1 = data.data;
          this.spin = false;
      }.bind(this)).catch(function(error){
        console.log('error');
      });
    },
    search(){
      // this.$http.get('http://localhost/tp5/public/index.php/api/v1/task_search/'+this.searchStr)
      //   .then(function(res){
      //     var data = res.data;
      //     this.total = data.total;
      //     this.current = data.current_page;
      //     this.data1 = data.data;
      //     this.$Message.success('搜索成功');
      //   },function(res){
      //     this.$Message.error('搜索请求失败');
      //   });
      console.log('search');
    },
    show(index) {
        this.$Modal.info({
            title: '详细信息',
            content: `ID：${this.data1[index].remarks}<br>`
        })
    },
    add(){

    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>


</style>