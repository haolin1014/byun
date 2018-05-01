<template>
	<div class="content-box">

    <Row>
      <Col span="6">
            <Bread></Bread>
      </Col>
    </Row>

<!-- 		<Row class="margin-10">
		  <Col span="2" offset="1">
		    <div class="addBox">
		      <Button type="info" icon="plus" @click="addModal=true">添加顶级分类</Button>
		    </div>
		  </Col>
		</Row> -->
		
    <Row class="margin-10">
      <Col span="20" offset="1">
           <Collapse accordion>
               <Panel v-for="(v,i) in data1" :key="v.id" :name="v.name">
                   {{v.name}}  &nbsp;&nbsp;&nbsp;<!-- <Button type="info" size="small" @click="childCreateModal(i)" icon="android-add"></Button>&nbsp;&nbsp;&nbsp;<Button type="warning" size="small" @click="updateCate(v)" icon="edit"></Button> -->
                   <div slot="content">
                       <Row>
                          <Col span="3" v-for="(v1,i1) in v.childCategories" :key="i1">
                            <Row>
                              <Col span="10">
                                <p class="margin-10">{{v1.name}}</p>
                              </Col>
                              <Col span="14">
                                <p class="margin-10">
                                  <!-- <Button type="warning" size="small" @click="updateCate(v1)" icon="edit"></Button> -->
                                  <!-- <Button type="error" size="small" icon="android-delete" @click="deleteCate(v1)"></Button> -->
                                </p>
                              </Col>
                            </Row>
                          </Col>
                       </Row>
                   </div>
               </Panel>
           </Collapse> 
      </Col>
    </Row>
  



		<Modal
		    v-model="addModal"
		    title="添加分类"
		    @on-ok="add"
		    @on-cancel="addModal=false">
		    <p class="margin-10">名称 &nbsp;&nbsp;&nbsp;&nbsp;<Input v-model="signData.name" placeholder="名称" style="width: 300px" ></Input></p>
		    <p class="margin-10">排序 &nbsp;&nbsp;&nbsp;&nbsp;<Input v-model="signData.sort" placeholder="排序" style="width: 300px" ></Input></p>
		</Modal>

    <Modal
        v-model="childModal"
        title="添加子分类"
        @on-ok="addChildCate"
        @on-cancel="childModal=false">
        <p class="margin-10">名称 &nbsp;&nbsp;&nbsp;&nbsp;<Input v-model="childData.name" placeholder="名称" style="width: 300px" ></Input></p>
        <p class="margin-10">排序 &nbsp;&nbsp;&nbsp;&nbsp;<Input v-model="childData.sort" placeholder="排序" style="width: 300px" ></Input></p>
    </Modal>

		<Modal
		    v-model="UpdateModal"
		    title="修改分类"
		    @on-ok="update"
		    @on-cancel="UpdateModal=false">
		    <p class="margin-10">名称 &nbsp;&nbsp;&nbsp;&nbsp;<Input v-model="signUpdate.name" placeholder="名称" style="width: 300px" ></Input></p>
		    <p class="margin-10">排序 &nbsp;&nbsp;&nbsp;&nbsp;<Input v-model="signUpdate.sort" placeholder="排序" style="width: 300px" ></Input></p>
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
      addModal:false,
      UpdateModal:false,
      childModal:false,
      spin:false,
      signUpdate:{
        id:'',
      	name:'',
      	sort:2
      },
      signData:{
        name:'',
        sort:2
      },
      childData:{
        id:'',
        name:'',
        sort:2
      },
      data1: [],
      columns1: [
                 {
                     title: '类别名称',
                     key: 'name'
                 },
                 {
                     title: '商品',
                     key: 'goods',
                     render:(h,params)=>{
                        return h('Button',{
                                props:{
                                  type:'primary'
                                },
                                on:{
                                	click:()=>{
                                		// this.$router.push('/goods/index/sign_goods');
                                	}
                                }
                              },'查看');
                     }
                 },
                 {
                     title: '排序',
                     key: 'sort',
                 },
                 {
                     title: '操作',
                     key: 'action',
                     width: 250,
                     align: 'center',
                     render: (h, params) => {
                         if(params.row.name.indexOf('---')!=0){
                            var child =  h('Button', {
                                    props: {
                                        type: 'success',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    on: {
                                        click: () => {
                                            this.childCreateModal(params.index);
                                        }
                                    }
                                }, '创建子分类');
                         }

                         return h('div', [
                             h('Button', {
                                 props: {
                                     type: 'primary',
                                     size: 'small'
                                 },
                                 style: {
                                     marginRight: '5px'
                                 },
                                 on: {
                                     click: () => {
                                         this.updateCate(params.index);
                                     }
                                 }
                             }, '修改'),
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
                                         console.log('2222');
                                     }
                                 }
                             }, '删除'),
                             child
                         ]);
                     }
                 }
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
      axios.get(URL+'category').then(function(res){
        var data = res.data;
        this.data1 = data.data;
      }.bind(this)).catch(function(error){
        console.log(error);
      });
    },
  	add(){
      var params = new URLSearchParams();
      params.append('name', this.signData.name);
      axios.post(URL+'category',params).then(function(res){
        this.signData.name = '';
        this.dataInit();
        this.$Message.success('添加成功');
      }.bind(this)).catch(function(error){
        this.$Message.error('添加失败');
      }.bind(this));
  	},
    updateCate(v){
      this.UpdateModal = true;
      this.signUpdate.name = v.name;
      this.signUpdate.id = v.id;
    },
    update(){
      axios.put(URL+'category/'+this.signUpdate.id+'?name='+this.signUpdate.name).then(function(res){
        this.signUpdate.name = '';
        this.signUpdate.id = '';
        this.dataInit();
        this.$Message.success('修改成功');
      }.bind(this)).catch(function(error){
        this.$Message.error('修改失败');
      }.bind(this));
    },
    childCreateModal(index){
      this.childData.id = this.data1[index].id;
      this.childModal = true;
    },
    addChildCate(){
      var params = new URLSearchParams();
      params.append('name', this.childData.name);
      axios.post(URL+'category/'+this.childData.id,params).then(function(res){
        this.childData.name = '';
        this.childData.id = '';
        this.dataInit();
        this.$Message.success('添加成功');
      }.bind(this)).catch(function(error){
        this.$Message.error('添加失败');
      }.bind(this));
    },
    deleteCate(v){
      axios.delete(URL+'category/'+v.id).then(function(res){
        if(res.data.errorCode!=200){
          this.$Message.error(res.data.moreInfo);
        }else{
          this.dataInit();
          this.$Message.success('删除成功');
        }
      }.bind(this)).catch(function(error){
        this.$Message.error('删除失败');
      }.bind(this));
    }

  }
}
</script>
<style scoped>

</style>