<!-- 
  底部组件（全局注册组件）
  navInd：导航的默认值
-->
<template>
  <footer class="index_footer">
    <nav>
      <a
        v-for="(item,index) in proData&&proData.home_bottom_nav.length>0&&proData.home_bottom_nav ||navNameList"
        @click="goTo(index)"
        :class="{'nav-bg':navInd == index}"
        :key="index"
      >
        <i :class="'iconfont'+' '+iconList[index]"></i>
        <span>{{item.name}}</span>
      </a>
      <!-- <a v-for='(item, i) in proData&&proData.home_bottom_nav' :key='i' @click="goTo(i+1)" :class="{'nav-bg':navInd == i + 1}">
          <img :src="navInd == i + 1 ? item.activeImg : item.passiveImg" alt="">
          <span>{{item.txt}}</span>
      </a>-->
    </nav>
  </footer>
</template>

<script>
import { mapGetters } from "vuex";
export default {
  data() {
    return {
      navInd: 0,
      iconList: [
        "iconicon-test1",
        "iconcheliang-",
        "icondiannaopc-kuai",
        "iconjiaoyijilu",
        "iconqkk_icon_wode"
      ],
      routeName: ["pan-gold", "Interaction", "Share", "Transaction", "My"],
      navNameList: [
        {
          name: "首页"
        },
        {
          name: "车联互助"
        },
        {
          name: "共享停车"
        },
        {
          name: "交易中"
        },
        {
          name: "我的"
        }
      ]
    };
  },
  mounted() {
    this.activeNav();
  },
  computed: {
    ...mapGetters(["proData"])
  },
  methods: {
    goTo(index) {
      this.navInd = index;
      sessionStorage.setItem("btmNav", index);
      this.$router.push({ name: this.routeName[index] });
    },
    activeNav() {
      if (sessionStorage.getItem("btmNav")) {
        let index = sessionStorage.getItem("btmNav");
        this.navInd = index;
      }
    }
  }
};
</script>
<style scoped lang="less">
@import "~link-less";
nav {
  box-sizing: border-box;
  width: 100%;
  height: 50px;
  color: #fff;
  // background-color: #304d79;
  background-color: #232323;
  .display-flex();
  justify-content: space-between;
  a {
    flex: 1;
    display: -webkit-flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    i {
      font-size: 20px;
    }
    img {
      width: 20px;
    }
  }
  .nav-bg {
    // color: @font-primary-color;
    color: #fff;
    span {
      color: #3cfcac;
    }
    i {
      color: #3cfcac;
    }
  }
}
</style>
