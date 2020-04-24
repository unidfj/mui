<template>
  <div class="page">
    <m-header title="商品详情" :canback="Boolean(1)">
      <i class="iconfont iconcart cart" @click="$router.push({name:'Cart'})"></i>
    </m-header>
    <section class="body">
      <div class="top">
        <!-- <img :src="img" alt=""> -->

        <div class="swiper">
          <swiper :options="swiperOption" class="swiper-wrap" ref="mySwiper" v-if="Array(data.imgs_url).length!=0">
            <swiper-slide v-for="(item, index) in data.imgs_url" :key="index">
              <img :src="item" class="title-img" />
            </swiper-slide>
            <!-- 常见的小圆点 -->
            <div
              class="swiper-pagination"
              v-for="(item,index) in data.imgs_url"
              :key="index"
              slot="pagination"
            ></div>
          </swiper>
        </div>
        <p>
          <span>{{data.goods_name}}</span>
          <span>
            <i :class="{'iconfont iconshoucang':true,'se':data.collection_bonus==1}" @click="collectionBonus"></i>
            <i class="iconfont iconfenxiang"></i>
          </span>
        </p>
        <div class="bot">
          <p>{{data.spec[0].goods_price}}</p>
          <p>{{data.spec[0].line_price}}</p>
          <!-- <p>¥32.90</p> -->
          <p>已售{{data.sales_actual}}件</p>
        </div>
      </div>

      <div class="content">
        <p class="title">
          商品详情
        </p>
        <!-- <ul>
          <li v-for="(v,i) in [1]" :key="i">
            <img :src="img" alt="">
          </li>
        </ul> -->
        <div v-html="data.content" class="img_box"></div>
      </div>

      <div class="btn_box">
        <div @click="show=true,type='add'">加入购物车</div>
        <div @click="$router.push({name:'pOrder',query:{type:'buyNow',goods_id:id,goods_num,goods_sku_id,goods_sku_id}})">立即购买</div>
      </div>


      <div class="ale" v-show="show">
        <div class="box">
          <ul v-for="(v,i) in spec_attr" :key="i">
            <p class="title">{{v.group_name}}</p>
            <li v-for="(v2,i2) in v.spec_items" :key="i2" :class="{'se':se.indexOf(v2.item_id)!=-1}" @click="change(i,v2.item_id)">
              {{v2.spec_value}}
            </li>
          </ul>
          <div class="bot">
            <p>购买数量</p>
            <p>
              <i class="iconfont iconjianhao" @click="goods_num<2?'':goods_num--"></i>
              <span>{{goods_num}}</span>
              <i class="iconfont iconjiahao" @click="goods_num++"></i>
            </p>
          </div>
          <div class="btn_box">
            <button @click="changeShow(false)">取消</button><button @click="changeShow(true)">确定</button>
          </div>
        </div>
      </div>
    </section>
    <m-load ref="load"></m-load>
  </div>
</template>
<script>
import {Trade,Index,Base} from '@/server/server.js';
import {Util,Load,Secret} from '@/assets/commonjs/utils.js';
import {mapGetters,mapMutations} from 'vuex';
import { swiper, swiperSlide } from "vue-awesome-swiper";
require("swiper/dist/css/swiper.css"); //引入swiper.css
export default {
  components: {
    swiper,
    swiperSlide,
  },
  data(){
      return {
        img:require('@/assets/images/gzlz/banner.png'),
        id:null,
        swiperOption: {
          loop: true,
            autoplay: {
              delay: 2500,
              stopOnLastSlide: false,
              /* 触摸滑动后是否继续轮播 */
              disableOnInteraction: false
            },
            // 分页器设置
            pagination: {
              el: ".swiper-pagination",
              clickable: true,
              type: "bullets"
            }
        },
        data:{
          imgs_url:[],
          content:null,
          collection_bonus:null,
          goods_id:null,
          sales_actual:null,
          spec:[
            {
              goods_price:null,
              line_price:null,
            }
          ]
        },
        spec_attr:[],
        se:[],
        goods_num:1,
        show:false,
        type:null,
        goods_sku_id:null,
      }
  },
  mounted() {
      this.initData();
      this.detail()
      
  },
  computed: {
    ...mapGetters(['uid','api','bannerSrc','coinTxt','proData'])
  },
  methods: {
    initData(){
      this.id = this.$route.query.id;
    },
    // 收藏/取消收藏
    collectionBonus(){
      let params = {
        goodsid:this.data.goods_id,
        type:this.data.collection_bonus==1?2:1
      }
      Trade.collectionBonus(params).then(res=>{
        mui.toast(res.msg)
        if(res.code==1){
          this.data.collection_bonus = res.msg=='取消收藏成功'?2:1;
        }
      })
    },
    // 规格弹窗
    changeShow(bl){
      if(bl){
        this.add();
      }else{
        this.se = [];
      }
      this.show = false;
    },
    addCart(){
      if(this.specData){
        this.show=true
      }else{
        this.add();
      }
    },
    // 加入购物车
    add(){
      // this.goods_sku_id = this.se.join('-')
      Trade.add({goods_num:this.goods_num,goods_id:this.data.goods_id,goods_sku_id:this.goods_sku_id}).then(res=>{
        mui.toast(res.msg)
        if(res.code==1){
          
        }
      })
    },
    // 选择规格
    change(i,id){
      this.se[i] = id;
      this.goods_sku_id = this.se.join('-')
      this.$forceUpdate()
    },
    // 商品详情
    detail(){
      Trade.detail({goods_id:this.id}).then(res=>{
        if(res.code==1){
          this.data = res.data.detail;
          if(!res.data.specData) return;
          this.spec_attr = res.data.specData.spec_attr;
          if(!this.spec_attr) return;
          this.spec_attr.forEach((v,i)=>{
            this.se[i] = v.spec_items[0].item_id
          })
          console.log('se',this.se)
        }else{
          mui.toast(res.msg)
        }
      })
    },
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
  position: relative;
  >.top{
    background-color: #232323;
    margin-bottom: 30px;
    >img{
      width: 100%;
      margin-bottom: 24px;
    }
    >p{
      font-size:32px;
      font-weight:500;
      color:rgba(255,255,255,1);
      padding: 0 30px;
      display: flex;
      justify-content: space-between;
      span{
        &:nth-child(2){
          display: inline-block;
          width: 124px;
        }
        >i{
          margin-left: 20px;
        }
        .se{
          color: rgb(245, 248, 58);
        }
      }
    }
    >div{
      display: flex;
      justify-content: space-between;
      // padding: 0 30px;
      // margin-bottom: 30px;
      // height: 64px;
      line-height: 64px;
      p{
        &:first-child{
          font-size:44px;
          font-weight:500;
          color:rgba(60,252,172,1);
        }
        &:nth-child(2){
          font-size:32px;
          color:rgba(170,170,170,1);
          flex: 1;
          margin-left: 16px;
          text-decoration:line-through;
        }
        &:last-child{
          font-size:28px;
          color:rgba(170,170,170,1);
        }
      }
    }
    .bot{
      padding: 0 30px;
    }
  }
  >.content{
    background-color: #232323;
    padding-top: 22px;
    margin-bottom: 108px;
    .title{
      font-size:32px;
      font-weight:500;
      color:rgba(255,255,255,1);
      margin: 0 30px 30px;
    }
    .img_box{
      /deep/ p{
        img{
          width: 710px !important;
          // width: 100%;
          margin-left: 20px;
        }
      }
    }
    >ul{
      li{
        img{
          width: 710px;
          margin-left: 20px;
        }
      }
    }
  }
  >.btn_box{
    display: flex;
    justify-content: space-between;
    width: 100%;
    position: fixed;
    bottom: 0px;
    border-radius: 0;
    font-size: 0;
    >div{
      flex: 1;
      color: #333;
      font-size:28px;
      font-weight: 500;
      height: 88px;
      line-height: 88px;
      text-align: center;
      background-color: #3CFCAC;
      border-radius: 0;
      display: inline-block;
      &:last-child{
        background-color: #FFC36D;
      }
    }
  }
  .swiper{
    margin-bottom: 0;
    .title-img{
      width: 100%;
    }
  }
  >.ale{
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, .6);
    position: fixed;
    top: 0;
    left: 0;
    z-index: 10;
    .box{
      background-color: #fff;
      width: 690px;
      // margin: 100px 30px;
      border-radius: 10px;
      padding: 20px 30px;
      position: absolute;
      left: 30px;
      bottom: 300px;
      >ul{
        border-bottom: 1px solid rgb(120, 197, 228);
        margin-bottom: 20px;
        padding-bottom: 10px;
        .title{
          color: #000;
          font-size: 32px;
          margin-bottom: 10px;
        }
        li{
          display: inline-block;
          color: #000;
          border: 1px solid rgb(120, 197, 228);
          border-radius: 8px;
          margin-left: 20px;
          margin-bottom: 10px;
          padding: 4px 8px;
        }
        .se{
          background-color: rgb(120, 197, 228);
          color: #fff;
        }
      }
      .bot{
        display: flex;
        justify-content: space-between;
        padding: 0px 0 20px;
        p{
          color: #333;
          padding: 0 30px;
          i{
            font-size: 30px;
          }
        }
      }
      .btn_box{
        display: flex;
        justify-content: space-between;
        >button{
          height: 88px;
          line-height: 88px;
          width: 100%;
          background-color: #ccc;
          &:last-child{
            margin-left: 30px;
            background-color: rgb(120, 197, 228);
          }
        }
      }
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

