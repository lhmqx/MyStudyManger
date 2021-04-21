<template>
  <div class="content">
    <div class="search">
      <el-dropdown @command="handleForbid">
        <el-button type="danger">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item command="forbidComments">禁止评论</el-dropdown-item>
          <el-dropdown-item command="forbidVisit">禁止访问</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <div class="search_inp">
        <el-form ref="form">
          <el-form-item width="100">
            <el-input v-model="search.title" placeholder="标题"></el-input>
          </el-form-item>
          <el-button type="primary" @click="handleSearch">
            <i class="el-icon-search" />
            <span>搜索</span>
          </el-button>
        </el-form>
      </div>
    </div>
    <el-table
      :key="0"
      :data="userList"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="60"> </el-table-column>

      <el-table-column label="用户">
        <template slot-scope="{ row }">
          <div class="context">
            <img
              style="width:50px;height:50px;border-radius:50%;"
              :src="row.user.avatar"
              alt=""
            />
            <div class="user">
              <div>{{ row.user.username }}</div>
            </div>
          </div>
        </template>
      </el-table-column>

      <el-table-column label="消费总额" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.total_consume }}</span>
        </template>
      </el-table-column>

      <el-table-column label="创建时间" width="160" align="center">
        <template slot-scope="{ row }">
          <span>
            {{ row.created_time }}
          </span>
        </template>
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        width="320"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="primary" size="mini" @click="handleDetail(row)">
            详情
          </el-button>
          <el-button
            size="mini"
            type="success"
            @click="handleChangeStatus(row, $index)"
          >
            联系用户
          </el-button>
          <el-dropdown @command="handleForbidOne">
            <el-button type="danger" size="mini" style="margin-left:10px">
              黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
            </el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="forbidComments"
                >禁止评论</el-dropdown-item
              >
              <el-dropdown-item command="forbidVisit"
                >禁止访问</el-dropdown-item
              >
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="search.page"
      :limit.sync="search.limit"
      @pagination="getUserList"
    />

    <user-mes v-if="isShow" :isDetail="isDetail" :handleCloseMes="handleCloseMes" :usermes="usermes"></user-mes>
  </div>
</template>

<script>
import { fetchList } from "@/api/user";
import Pagination from "@/components/Pagination";
import Tinymce from "@/components/Tinymce";
import UserMes from "@/views/user/UserMes";
const defaultRuleForm = {
  title: "",
  cover: "",
  try: "",
  content: "",
  price: 5,
  status: 0
};
export default {
  naem: "user",
  components: {
    Pagination,
    Tinymce,
    UserMes
  },
  data() {
    return {
      userList: [],
      total: 0,
      search: {
        page: 1,
        limit: 5,
        status: 0,
        title: ""
      },
      isDetail: false,
      isShow:false,
      ruleForm: Object.assign(defaultRuleForm, {}),
      multipleSelection: [],
      usermes:{},
    };
  },
  mounted() {
    // 获取用户数据
    this.getUserList();
  },
  methods: {
    // 获取图片数据
    async getUserList() {
      let mes = await fetchList(this.search);
      if (mes.code === 20000) {
        this.total = mes.data.total;
        this.userList = mes.data.items;
      }
      // console.log(this.userList);
    },

    // 批量禁止
    handleForbid(command) {
      if (this.multipleSelection.length <= 0) {
        return this.$message.error("请先选中需要操作的用户");
      } else if (command === "forbidComments") {
        // 禁止评论
        this.$confirm("是否要禁止选中用户评论？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            this.$message.success("操作成功");
            this.multipleSelection = [];
          })
          .catch(err => {
            thsi.$message.error("操作失败");
          });
      } else if (command === "forbidVisit") {
        // 禁止访问
        this.$confirm("是否要禁止选中用户访问？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            this.$message.success("操作成功");
            this.multipleSelection = [];
          })
          .catch(err => {
            thsi.$message.error("操作失败");
          });
      }
    },

    // 单个禁止
    handleForbidOne(command) {
      if (command === "forbidComments") {
        // 禁止评论
        this.$confirm("是否要禁止选中用户评论？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            this.$message.success("操作成功");
          })
          .catch(err => {
            thsi.$message.error("操作失败");
          });
      } else if (command === "forbidVisit") {
        // 禁止访问
        this.$confirm("是否要禁止选中用户访问？", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            this.$message.success("操作成功");
          })
          .catch(err => {
            thsi.$message.error("操作失败");
          });
      }
    },

    // 表格数据选中
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    // 搜索数据
    handleSearch() {
      this.getUserList();
    },

    // 关闭用户详情
    handleCloseMes() {
      this.isDetail = false;
      this.isShow = false;
    },

    // 开启编辑
    handleDetail(row) {
      this.isDetail = true;
      this.isShow = true;
      this.usermes = row;
    },
  }
};
</script>

<style scoped>
.content {
  padding: 30px;
}
.search {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 36px;
  margin: 20px;
}
.search_inp form {
  display: flex;
  height: 36px;
}
.context {
  display: flex;
  align-items: center;
}
.user {
  text-align: left;
  display: flex;
  flex-direction: column;
}
.user div:nth-child(2) {
  color: red;
}
.addForm {
  width: 60%;
}
</style>
