<template>
  <div class="content-box">
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
          <Button type="info" icon="plus" @click="addModal=true">添加新用户</Button>
        </div>
      </Col>
    </Row>
    
    <div class="layout-content-main">
     <Table stripe border highlight-row :columns="columns1" :data="data1" @on-selection-change="selectChange"></Table>
     <div style="margin: 10px;overflow: hidden">
         <div style="float: right;">
             <Page :total="total" :current="current" @on-change="changePage"></Page>
         </div>
     </div>
    </div>

    <Modal
        v-model="addModal"
        title="添加新用户"
        @on-ok="add"
        @on-cancel="addModal=false">
 
        <Row>
          <Col span="6">
              <p class="margin-10">公司名称</p>
          </Col>
          <Col span="18">
            <Input v-model="newUser.company" style="width: 300px" ></Input>
          </Col>
        </Row>
        <Row>
          <Col span="6">
              <p class="margin-10">公司负责人</p>
          </Col>
          <Col span="18">
            <Input v-model="newUser.name" style="width: 300px" ></Input>
          </Col>
        </Row>
        <Row>
          <Col span="6">
              <p class="margin-10">公司负责人联系方式</p>
          </Col>
          <Col span="18">
            <Input v-model="newUser.phone" style="width: 300px" ></Input>
          </Col>
        </Row>
        <Row>
          <Col span="6">
              <p class="margin-10">公司地址</p>
          </Col>
          <Col span="18">
            <Input v-model="newUser.adress" style="width: 300px" ></Input>
          </Col>
        </Row>
        <Row>
          <Col span="6">
              <p class="margin-10">公司信用额度</p>
          </Col>
          <Col span="18">
            <Input v-model="newUser.credit" style="width: 300px" ></Input>
          </Col>
        </Row>
        <Row>
          <Col span="6">
              <p class="margin-10">公司地区</p>
          </Col>
          <Col span="18">
            <Input v-model="newUser.area" style="width: 300px" ></Input>
          </Col>
        </Row>
    </Modal>

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
                     key: 'no'
                 },
                 {
                     title: '公司名称',
                     key: 'company'
                 },
                 {
                     title: '负责人姓名',
                     key: 'name',
                 },
                 {
                     title: '负责人手机号',
                     key: 'phone',
                 },
                 {
                     title: '地区',
                     key: 'area',
                 },
                 {
                     title: '信用额度',
                     key: 'credit',
                 },
                 {
                     title: '总消费金额',
                     key: 'consume',
                 },
                 {
                     title: '未结账款',
                     key: 'unfinishAccount',
                 },
                 {
                     title: '未处理清单',
                     key: 'unfinishBill',
                 },
                 {
                     title: '结算日',
                     key: 'date',
                 },
                 {
                     title: '状态',
                     key: 'status',
                     render:(h,params)=>{
                        var type = params.row.status==1 ? 'default' : 'info';
                        var statusName = params.row.status==1 ? '正常' : '冻结';
                        return h('Button',{
                          props:{
                            type:type,
                            size:'small'
                          }
                        },statusName);
                     }
                 },
                 {
                     title: '操作',
                     key: 'action',
                     width: 200,
                     align: 'center',
                     render: (h, params) => {
                         var type = params.row.status==1 ? 'error' : 'info';
                         var statusName = params.row.status==1 ? '冻结' : '恢复正常';
                         return h('div', [
                             h('Button', {
                                 props: {
                                     type: 'warning',
                                     size: 'small'
                                 },
                                 style: {
                                     marginRight: '5px'
                                 },
                                 on: {
                                     click: () => {
                                         this.show(params.index)
                                     }
                                 }
                             }, '修改'),
                             h('Button', {
                                 props: {
                                     type: type,
                                     size: 'small'
                                 },
                                 style: {
                                     marginRight: '5px'
                                 },
                                 on: {
                                     click: () => {
                                      if(params.row.status==2){
                                        params.row.status=1;
                                      }else{
                                        params.row.status=2;
                                      }
                                         
                                     }
                                 }
                             }, statusName),
                         ]);
                     }
                 }
             ],
    }
  },
  mounted(){
    // axios.get(URL+'order/paginate',{
    //   'token':'1111'
    // }).then(function(res){
    //   var data = res.data;
    //   this.total = data.total;
    //   this.current = data.current_page;
    //   this.data1 = data.data;
    // }.bind(this)).catch(function(error){
    //   console.log(error);
    // });
      this.total = 1;
      this.current = 1;
      this.data1 = [
        {no:'123',company:'帮陪科技',name:'小王',phone:'12322321123',area:'江干',credit:1000,consume:1000,unfinishAccount:500,unfinishBill:2,date:'2017-10-11',status:1}
      ];
  },
  methods:{
    selectChange(o){
      var str = '';
      o.forEach(function(v,i){
        str += v.id+',';
      });
      this.del_ids = str.substr(0,str.length-1);
    },
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
    allDeliver(){
      console.log('批量发货');
    },
    show(index) {
        this.$Modal.info({
            title: '详细信息',
            content: `ID：${this.data1[index].id}<br>订单编号：${this.data1[index].order_no}<br>商品名称：${this.data1[index].snap_name}`
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
