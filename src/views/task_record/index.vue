<template>
  <div>
    <el-container>
      <el-header class="head-left-content">
        <el-button @click="goExecute" class="navibar-button-style" type="primary">执行任务</el-button>
        <el-button
          id="record"
          style="background-color: #0a6659;"
          class="navibar-button-style"
          type="primary"
        >任务记录</el-button>
      </el-header>
    </el-container>
    <div class="table-style">
      <el-form :inline="true" class="demo-form-inline">
        <el-form-item label="业务">
          <el-select
            id="cur_business"
            v-model="busi_name"
            placeholder="请选择"
            style="width: 290px"
            ref="select_business"
            @change="get_select_business_label"
          >
            <el-option v-for="item in businesses" :key="item.busi_name" :value="item.busi_name"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="用户">
          <el-select
            id="cur_user"
            v-model="user_name"
            style="width: 290px"
            placeholder="请选择"
            ref="select_user"
          >
            <el-option v-for="item in users" :key="item.user_name" :value="item.user_name"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="任务">
          <el-select
            id="cur_script"
            v-model="script_name"
            style="width: 290px"
            placeholder="请选择"
            ref="select_script"
          >
            <el-option v-for="item in scripts" :key="item.script_name" :value="item.script_name"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onQuery">查询</el-button>
        </el-form-item>
      </el-form>
      <hr
        style="FILTER: alpha(opacity=100,finishopacity=0,style=3)"
        width="100%"
        color="gray"
        size="1"
      />
      <!-- <el-tag color="white" size="medium" type="info">选择主机</el-tag> -->
      <el-table :data="cur_records" style="width: 100%" border>
        <el-table-column prop="business" label="业务" width="200"></el-table-column>
        <el-table-column prop="operator" label="用户" width="200"></el-table-column>
        <el-table-column prop="mission" label="任务" width="200"></el-table-column>
        <el-table-column prop="machine_num" label="机器数" width="150"></el-table-column>
        <el-table-column prop="status" label="状态" width="150"></el-table-column>
        <el-table-column prop="start_time" label="操作时间" width="250"></el-table-column>
      </el-table>
      <div style="float: right" class="paginationClass">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="current_page"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="page_size"
          layout="total, sizes, prev, pager, next, jumper"
          :total="record_num"
        ></el-pagination>
      </div>
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
      users: [{"user_name": "所有用户"}],
      user_name: "",
      businesses: [{"busi_name": "所有业务"}],
      busi_name: "",
      scripts: [{"script_name": "所有任务"}],
      script_name: "",
      records: [],
      cur_records: [],
      record_num: 0,
      current_page: 1,
      page_size:10
    };
  },
  created() {
    /**
     * 获取所有的数据
     */
    this.$http.get("/records/").then(resp => {
      if (resp.status === 200) {
        this.records = resp.data;
        this.record_num = this.records.length;
        this.currentChangePage(this.records, this.current_page)
        this.filter_conditions();
      }
    });
  },
  mounted() {
    $("#record").click(function() {
      $("#record").css("background", "#008272");
    });
  },
  methods: {
    /**
     * 筛选数据
     */
    onQuery() {
      const business = $("#cur_business").val();
      const operator = $("#cur_user").val();
      const mission = $("#cur_script").val();
      const params = {
        business: business,
        operator: operator,
        mission: mission
      };
      const QUERY_RECORD_URL = "/records/retrieve/?business=" + business + "&operator=" + operator + "&mission=" + mission
      this.$http.get(QUERY_RECORD_URL).then(resp => {
        if (resp.status === 200) {
          this.records = resp.data;
          this.record_num = this.records.length;
          this.currentChangePage(this.records, this.current_page);
        }
      });
    },
    /**
     * 跳转到 “执行任务”界面
     */
    goExecute () {
      this.$router.push({ path: window.PROJECT_CONFIG.SITE_URL+"execute-task" });
    },
    /**
     * 调整每页的数据显式数量
     */
    handleSizeChange (page_size) {
        this.page_size = page_size
        this.handleCurrentChange(this.current_page);
    },
    /**
     * 页码切换
     */
    handleCurrentChange (current_page) {
        this.current_page = current_page
        this.currentChangePage(this.records,current_page)
       
    },
    /**
     * 分页方法（重点）
     */
   currentChangePage (records,current_page) {
      let from = (current_page - 1) * this.page_size;
      let to = current_page * this.page_size;
      this.cur_records = [];
      for (; from < to; from++) {
        if (records[from]) {
          this.cur_records.push(records[from]);
        }
      }
    },
    /**
     * 获取筛选条件
     */
    filter_conditions () {
      if (this.records) {
        let array_users = []
        let array_businesses = []
        let array_scripts = []
        this.records.forEach (item => {
          array_users.push(item.operator)
          array_businesses.push(item.business)
          array_scripts.push(item.mission)
        })
        array_users = new Set(array_users)
        array_businesses = new Set(array_businesses)
        array_scripts = new Set(array_scripts)
        array_users.forEach (user_item => {
          this.users.push({"user_name": user_item})
        })
        array_businesses.forEach (busi_item => {
          this.businesses.push({"busi_name": busi_item})
        })
        array_scripts.forEach (script_item => {
          this.scripts.push({"script_name": script_item})
        })
      }
    }
  }
};
</script>

<style scoped>
.table-style {
  position: relative;
  width: 1151px;
  top: 100px;
  left: 50%;
  transform: translate(-50%, 0);
}
.head-left-content {
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
