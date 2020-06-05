<template>
  <div>
    <el-container>
      <el-header class="head-left-content">
        <el-button
          id="execute"
          style="background-color: #0a6659;"
          class="navibar-button-style"
          type="primary"
        >执行任务</el-button>
        <el-button @click="goRecord" class="navibar-button-style" type="primary">任务记录</el-button>
      </el-header>
    </el-container>
    <div class="table-style">
      <el-form :inline="true" class="demo-form-inline">
        <el-form-item label="选择业务">
          <el-select
            id="cur_business"
            v-model="bk_biz_id"
            placeholder="请选择"
            ref="select_business"
            @change="get_select_business_label"
          >
            <el-option
              v-for="item in baseBusinessData"
              :key="item.bk_biz_id"
              :label="item.bk_biz_name"
              :value="item.bk_biz_id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="选择脚本">
          <el-select
            id="cur_script"
            v-model="content"
            style="width: 280px"
            placeholder="请选择"
            ref="select_script"
            @change="get_select_script_label"
          >
            <el-option
              v-for="item in baseScriptData"
              :key="item.content"
              :label="item.name"
              :value="item.content"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">执行</el-button>
        </el-form-item>
      </el-form>
      <hr
        style="FILTER: alpha(opacity=100,finishopacity=0,style=3)"
        width="100%"
        color="gray"
        size="1"
      />
      <!-- <el-tag color="white" size="medium" type="info">选择主机</el-tag> -->
      <el-table
        ref="multipleTable"
        :data="tableData"
        tooltip-effect="dark"
        style="width: 68%"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column prop="host.bk_host_innerip" label="IP" width="250"></el-table-column>
        <el-table-column prop="host.bk_os_name" label="操作系统" width="250"></el-table-column>
        <el-table-column fixed="right" label="操作" width="100">
          <template slot-scope="scope">
            <el-button
              @click.native.prevent="deleteRow(scope.$index, tableData)"
              type="text"
              size="small"
            >待定</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
import $ from "jquery";
import qs from "qs";
export default {
  components: {},
  data() {
    return {
      tableData: [],
      baseBusinessData: [],
      bk_biz_id: "",
      baseScriptData: [],
      content: "",
      selected_bk_biz_id: "",
      selected_script: "",
      hostIps: new Array()
    };
  },
  created() {
    this.$http.get("/query_all_info/").then(resp => {
      this.tableData = resp.data.host_data;
      this.baseBusinessData = resp.data.business_data;
      this.baseScriptData = resp.data.script_data;
    });
  },
  mounted() {
    $("#execute").click(function() {
      $("#execute").css("background", "#008272");
    });
  },
  methods: {
    onSubmit() {
      /*
      提交脚本执行
      */
      let params = {}; // new FormData();
      // 判断是否有选择业务
      if (this.selected_bk_biz_id !== "") {
        params["bk_biz_id"] = this.selected_bk_biz_id;
        // params.append("bk_biz_id", this.selected_bk_biz_id);
      } else {
        this.$message.warning("请选择业务");
        return;
      }
      // 判断是否有选择脚本
      if (this.selected_script !== "") {
        params["script"] = this.selected_script;
        // params.append("script", this.selected_script);
      } else {
        this.$message.warning("请选择脚本");
        return;
      }
      // 判断是否有选择主机
      if (this.hostIps.length > 0) {
        let temp_host_ips = [];
        this.hostIps.forEach(item => {
          temp_host_ips.push(item);
        });
        params["host_ips"] = temp_host_ips.join(",");
        // params.append("host_ips", temp_host_ips);
      } else {
        this.$message.warning("请至少选择一台主机");
        return;
      }
      this.$http.post("/execute_script/", qs.stringify(params)).then(resp => {
        if (resp.result) {
          this.$message.success("执行成功!")
        } else {
          this.$message.error("执行失败!")
        }
      });
    },
    /**
     * 跳转到“任务记录”界面
     */
    goRecord() {
      this.$router.push({ path: "/task-record" });
    },
    /**
     * 将选中的根据业务筛选主机并展示
     */
    get_select_business_label(val) {
      this.selected_bk_biz_id = val;

      let params = new FormData();
      params.append("bk_biz_id", this.selected_bk_biz_id);
      this.$http.post("/query_host_info/", params).then(resp => {
        if (resp.result) {
          this.tableData = resp.data.host_data;
        }
      });
    },
    /**
     * 获取选中的脚本命令
     */
    get_select_script_label(val) {
      this.selected_script = val;
    },
    /**
     * 存储选中的主机Ip
     */
    handleSelectionChange(hosts) {
      this.hostIps.splice(0, this.hostIps.length);
      for (let item in hosts) {
        this.hostIps.push(hosts[item]["host"]["bk_host_innerip"]);
      }
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

.table-style {
  position: relative;
  width: 1000px;
  top: 200px;
  left: 50%;
  transform: translate(-50%, 0);
}

.navibar-button-style:hover {
  background-color: #0a6659;
}
</style>
