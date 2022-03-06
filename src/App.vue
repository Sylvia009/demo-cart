<template>
  <div class="app-container">
    <!-- Header头部区域 -->
    <Header title="购物车实例"></Header>
    
    <!-- 循环渲染每一个商品的信息 -->
    <!-- 编码规范：先指令，后绑定，最后事件绑定 -->
    <Goods 
    v-for="item in list" 
    :key="item.id"
    :id="item.id" 
    :title="item.goods_name" 
    :pic="item.goods_img" 
    :price="item.goods_price"
    :state="item.goods_state"
    :count="item.goods_count"
    @state-change="getNewState"
    ></Goods>

    <!-- Footer区域 -->
    <Footer :isfull="fullState" :amt="amount" :all="total" @full-change="getFullState"></Footer>
  </div>
</template>

<script>
//导入axios请求库
import axios from 'axios'
//导入需要的组件
import Header from '@/components/Header/Header.vue'
import Goods from '@/components/Goods/Goods.vue'
import Footer from '@/components/Footer/Footer.vue'

import bus from '@/components/eventBus.js'

export default {
  data() {
    return {
      //用来存储购物车的列表数据，默认为空数组
      list: []
    }
  },
  
  //计算属性；全选
  computed: {
    //动态计算出全选的状态是true还是false
    fullState() {
     return this.list.every(item => item.goods_state)
    },
    //已勾选商品的总价格
    amount() {
      //先filter过滤
      //再reduce累加
      return this.list
      .filter(item => item.goods_state)
      .reduce((total, item) => (total += item.goods_price * item.goods_count),0)
    },
    //已勾选商品的总价格
    total() {
      //先过滤filter掉商品的状态：是否被选中（item.goods_state=true简写了=true,因为默认是true）
      //接下来累加reduce商品的数量，（t,item）t代表total.省略了{return},最后的0指的是reduce初始值是0，从0开始累加
      return this.list.filter(item => item.goods_state).reduce((t,item) => (t += item.goods_count),0)
    }
  },
  created() {
    //调用请求数据的方法
    this.initCartList()

    bus.$on('share', (val) => {
      this.list.some(item => {
        if(item.id === val.id) {
          item.goods_count = val.value
          return true
        }
      })
    })
  },
  methods: {

    //封装请求列表数据的方法
   async initCartList() {
     //调用axios的get方法，请求列表数据
     const { data:res } = await axios.get('https://www.escook.cn/api/cart')
     if (res.status === 200) {
       this.list = res.list
     }
    },
    //接受子组件传递过来的数据
    //e的格式为{id,value}
    getNewState(e) {
      console.log(this)
      this.list.some(item => {
        if(item.id === e.id) {
          item.goods_state = e.value
          //终止后续的循环
          return true
        }
      })
    },
    //接收子组件传递过来的数据
    getFullState(val) {
      console.log('已经传递过来啦')
      this.list.forEach(item => item.goods_state = val)
    }
  },
  components: {
    Header,
    Goods,
    Footer
  }
}
</script>

<style lang="less" scoped>
.app-container {
  padding-top: 45px;
  padding-bottom: 50px;
}
</style>
