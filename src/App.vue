<template>
  <div id="app" :class="systemCls">
      <!-- 滚动条样式等后面再搞 -->
    <!-- <div class="scroll">
      <el-scrollbar
        wrap-class="list"
        wrap-style="color: red;"
        view-style="font-weight: bold;"
        view-class="view-box"
        :native="false"
      > -->
        <ul class="header">
          <!-- <li>
                <a href="javascript:void(0);" :class="$route.name === 'example1' ? 'bk-text-button' : ''" @click.stop="goPage('execute_task')">execute_task</a>
            </li>
            <li>
                <a href="javascript:void(0);" :class="$route.name === 'example2' ? 'bk-text-button' : ''" @click.stop="goPage('task_record')">task_record</a>
          </li>-->
        </ul>
        <!-- 全局加载动画（isLoading） -->
        <!-- <main class="main-content" v-bkloading="{ isLoading: mainContentLoading, opacity: 1 }"> -->
        <main class="main-content">
          <router-view :key="routerKey" v-show="!mainContentLoading" />
        </main>
        <app-auth ref="bkAuth"></app-auth>
      <!-- </el-scrollbar>
    </div> -->
  </div>
</template>
<script>
import { mapGetters } from "vuex";

import { bus } from "@/common/bus";

export default {
  name: "app",
  data() {
    return {
      routerKey: +new Date(),
      systemCls: "mac"
    };
  },
  computed: {
    ...mapGetters(["mainContentLoading"])
  },
  watch: {},
  created() {
    const platform = window.navigator.platform.toLowerCase();
    if (platform.indexOf("win") === 0) {
      this.systemCls = "win";
    }
  },
  mounted() {
    const self = this;
    bus.$on("show-login-modal", data => {
      self.$refs.bkAuth.showLoginModal(data);
    });
    bus.$on("close-login-modal", () => {
      self.$refs.bkAuth.hideLoginModal();
      setTimeout(() => {
        window.location.reload();
      }, 0);
    });
  },
  methods: {
    /**
     * router 跳转
     *
     * @param {string} idx 页面指示
     */
    goPage(idx) {
      this.$router.push({
        name: idx
      });
    }
  }
};
</script>

<style lang="postcss">
@import "./css/reset.css";
.scroll {
  height: 100%;
  overflow-y: hidden;
}
.el-scrollbar {
  height: 100%;
}
.el-scrollbar__bar {
  &.is-vertical {
    width: 100px;
  }
}
.el-scrollbar__wrap {
  overflow-y: auto;
  overflow-x: hidden;
}
.el-scrollbar__thumb {
  /* 可设置滚动条颜色 */
  /* background: url("../../assets/logo.png"); */
}
</style>
