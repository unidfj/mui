<template>
  <div class="page">
    <m-header title="购物车" :canback="Boolean(1)"></m-header>
    <section class="body">
      <ul class="list">
          <li v-for="(v,i) in list" :key="i">
            <img :src="v.image" alt="">
            <div>
              <p>{{v.goods_name}}</p>
              <p>{{v.goods_sku.goods_attr||'默认规格'}}</p>
              <p>￥{{v.goods_price}}</p>
            </div>
            
            <p class="bot">
              <i class="iconfont iconjianhao" @click="sub(v.goods_id,v.goods_sku_id,i,v.goods_price)"></i>
              <span>{{v.total_num}}</span>
              <i class="iconfont iconjiahao" @click="add(v.goods_id,v.goods_sku_id,i,v.goods_price)"></i>
            </p>
          </li>
      </ul>
      <div class="bot">
        <p>
          合计：
          {{order_pay_price}}
        </p>
        <button @click="$router.push({name:'pOrder',query:{type:'cart'}})">开始结算</button>
      </div>
    </section>
    <!-- <m-Footer></m-Footer> -->
    <m-load ref="load"></m-load>
  </div>
</template>
<script>
import {Trade} from '@/server/server.js';
import {Util,Load,Secret} from '@/assets/commonjs/utils.js';
import {mapGetters,mapMutations} from 'vuex';
export default {
  data(){
      return {
        img:require('@/assets/images/gzlz/banner.png'),
        page:1,
        id:null,
        list:[],
        order_pay_price:null,
        bl:true,
      }
  },
  mounted() {
      this.initData();
      this.getlists();
      
  },
  computed: {
    ...mapGetters(['uid','api','bannerSrc','coinTxt','proData'])
  },
  methods: {
    initData(){
      
    },
    
    // 加入购物车
    add(goods_id,goods_sku_id,i,v){
      console.log('11111',this.bl)
      if(!this.bl) return;
      this.bl = false;
      Trade.add({goods_num:1,goods_id,goods_sku_id}).then(res=>{
        this.bl = true;
        if(res.code==1){
          this.list[i].total_num++;
          this.order_pay_price = Number(+this.order_pay_price + +v).toFixed(2);
        }
      })
    },
    // 购物车数量-1
    sub(goods_id,goods_sku_id,i,v){
      if(!this.bl||this.list[i].total_num<2) return;
      this.bl = false;
      // this.goods_num<2?'':this.goods_num--
      Trade.sub({goods_id,goods_sku_id}).then(res=>{
        this.bl = true;
        if(res.code==1){
          this.list[i].total_num--;
          this.order_pay_price = Number(+this.order_pay_price - +v).toFixed(2);
        }else{
          mui.toast(res.msg)
        }
      })
    },
    
    // 购物车列表
    getlists(){
      Trade.getlists().then(res=>{
        if(res.code==1){
          this.list = res.data.goods_list;
          this.order_pay_price = res.data.order_total_price;
        }else{
          mui.toast(res.msg)
        }
      })
    },
    ...mapMutations(['saveBanner','saveCoinTxt']),
    
    formatTime(time){
      return Util.formatTime(time).substr(0,5);
    },
  },
  beforeRouteLeave (to, from, next) {
      clearInterval(this.mineInt);
      next();
    }
};
</script>
<style lang="less" scoped>
@import "~link-less";
.body {
  line-height: 36px;
  background-color: @bg-color;
  
    .list{
      width: 690px;
      margin: 30px 30px;
      li{
        display: flex;
        justify-content: space-between;
        margin-bottom: 30px;
        position: relative;
        background-color: #232323;
        // padding-right: 30px;
        padding: 20px 30px;
        img{
          width: 140px;
          height: 140px;
          margin-right: 30px;
        }
        >div{
          flex: 1;
          p{
            font-size: 24px;
            &:nth-child(2){
              font-size: 20px;
              color: #ddd;
            }
            &:nth-child(3){
              color: red;
            }
          }
        }
        .bot{
          position: absolute;
          right: 30px;
          bottom: 20px;
          i{
            font-size: 28px;
          }
          span{
            display: inline-block;
            border: 1px solid rgb(120, 197, 228);
            padding: 2px 18px;
            border-radius: 8px;
          }
        }
      }
    }
    >.bot{
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      display: flex;
      justify-content: space-between;
      height: 120px;
      line-height: 120px;
      background-color: #232323;
      button{
        padding:  0 40px;
        background-color: red;
        border-radius: 0;
        font-size: 32px;
      }
      >p{
        flex: 1;
        text-align: right;
        margin-right: 30px;
        font-size: 32px;
      }
    }
  
}
.page{
  .cart{
    color: #3CFCAC;
    font-size: 48px !important;
    // color: red;
  }
}
</style>

