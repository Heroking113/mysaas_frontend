<template>
  <div>
    <el-container>
      <el-header class="head-left-content">
        <el-button @click="goExecute" class="navibar-button-style" type="primary">执行任务</el-button>
        <el-button id="record" style="background-color: #0a6659;" class="navibar-button-style" type="primary">任务记录</el-button>
      </el-header>
    </el-container>

  </div>
</template>

<script>
import $ from 'jquery'
export default {
  components: {},
  data() {
    return {
    };
  },
  created () {},
  mounted () {
    $("#record").click(function(){
      $("#record").css("background", "#008272")
    })
  },
  methods: {
    goExecute () {
      this.$router.push({ path: "/execute-task" })
    },
    handleSelect(key, keyPath) {
    }
  }
};
</script>

<style scoped>
.head-left-content {
  overflow: hidden;
  background: #008272;
  max-height: 50px;
  padding: 0;
  float: left;
}

.navibar-button-style {
  height: 50px;
  width: 100px;
  border-radius: 0;
  font-size: 18px;
  font-weight: 300;
  margin: 0;
  border: 0;
  padding: 1rem 0 1.8rem;
}

.navibar-button-style:hover {
  background-color: #0a6659;
}

</style>
