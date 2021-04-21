<template>
  <div>
    <el-dialog
      title="用户详情"
      :visible.sync="isDetail"
      width="70%"
      :before-close="handleCloseMes"
      top="5vh"
    >
      <div class="addForm">
        <div class="top">
          <div class="oneLine">
            <div>ID:{{ userMes.id }}</div>
            <div>用户名：{{ userMes.user.username }}</div>
            <div>
              昵称：{{
                userMes.user.nickname === "" ? "未设置" : userMes.user.nickname
              }}
            </div>
            <div>手机号：{{ userMes.user.phone }}</div>
          </div>
          <div class="twoLine">
            <div>锁定：{{ userMes.no_access == 1 ? "是" : "否" }}</div>
            <div>会员：{{ userMes.user.user_level }}</div>
            <div>会员到期时间：{{ userMes.user.user_level_expire }}</div>
            <div>注册时间：{{ userMes.user.created_time }}</div>
          </div>
        </div>
        <div class="middle">
          <el-tabs v-model="activeName" @tab-click="handleClick">
            <el-tab-pane label="课程订阅" name="first">
              <el-table
                :key="0"
                :data="listQuery"
                border
                fit
                highlight-current-row
                style="width: 100%;"
                height="250"
                v-if="activeName === 'first'"
              >
                <el-table-column label="课程标题">
                  <template slot-scope="{ row }">
                    <span>{{ row.title }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="购买价格">
                  <template slot-scope="{ row }">
                    <span>{{ row.price }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="购买时间">
                  <template slot-scope="{ row }">
                    <span>{{ row.created_time }}</span>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
            <el-tab-pane label="专栏订阅" name="second">
              <el-table
                :key="0"
                :data="listQuery"
                border
                fit
                highlight-current-row
                style="width: 100%;"
                height="250"
                v-if="activeName === 'second'"
              >
                <el-table-column label="专栏标题">
                  <template slot-scope="{ row }">
                    <span>{{ row.title }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="购买价格">
                  <template slot-scope="{ row }">
                    <span>{{ row.price }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="购买时间">
                  <template slot-scope="{ row }">
                    <span>{{ row.created_time }}</span>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
            <el-tab-pane label="订单记录" name="third">
              <el-table
                :key="0"
                :data="listQuery"
                border
                fit
                highlight-current-row
                style="width: 100%;"
                height="250"
                v-if="activeName === 'third'"
              >
                <el-table-column label="ID">
                  <template slot-scope="{ row }">
                    <span>{{ row.id }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="订单号">
                  <template slot-scope="{ row }">
                    <span>{{ row.no }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="购买价格">
                  <template slot-scope="{ row }">
                    <span>{{ row.price }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="状态">
                  <template slot-scope="{ row }">
                    <span>{{ row.status | formatStatus }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="商品">
                  <template slot-scope="{ row }">
                    <span>{{ row.title }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="购买时间">
                  <template slot-scope="{ row }">
                    <span>{{ row.created_time }}</span>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
            <el-tab-pane label="观看历史" name="fourth">
              <el-table
                :key="0"
                :data="listQuery"
                border
                fit
                highlight-current-row
                style="width: 100%;"
                height="250"
                v-if="activeName === 'fourth'"
              >
                <el-table-column label="课程标题">
                  <template slot-scope="{ row }">
                    <span>{{ row.title }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="课程类型">
                  <template slot-scope="{ row }">
                    <span>{{ row.type | formatType }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="学习时长">
                  <template slot-scope="{ row }">
                    <span>{{ row.total_time }}</span>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
            <el-tab-pane label="用户评论" name="fifth">
              <el-table
                :key="0"
                :data="listQuery"
                border
                fit
                highlight-current-row
                style="width: 100%;"
                height="250"
                v-if="activeName === 'fifth'"
              >
                <el-table-column label="评论内容">
                  <template slot-scope="{ row }">
                    <span>{{ row.content }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="评论时间">
                  <template slot-scope="{ row }">
                    <span>{{ row.created_time }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="课程标题">
                  <template slot-scope="{ row }">
                    <span>{{ row.title }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="类型">
                  <template slot-scope="{ row }">
                    <span>{{ row.type | formatType }}</span>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
          </el-tabs>
        </div>
        <div class="footer">
          <pagination
            v-show="total > 0"
            :total="total"
            :page.sync="search.page"
            :limit.sync="search.limit"
            @pagination="getMyListQuery"
          />
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  fetchUserCourse,
  fetchUserColumn,
  fetchUserOrder,
  fetchUserHistory,
  fetchUserComment
} from "@/api/user";
import Pagination from "@/components/Pagination";
export default {
  name: "userMes",
  props: ["handleCloseMes", "usermes", "isDetail"],
  data() {
    return {
      userMes: this.usermes,
      activeName: "first",
      listQuery: [],
      total: 0,
      search: {
        limit: 5,
        page: 1
      }
    };
  },
  components: {
    Pagination
  },
  mounted() {
    this.getMyListQuery();
  },
  filters: {
    formatStatus(status) {
      if (status === "success") {
        return "成功";
      } else if (status === "fail") {
        return "失败";
      } else if (status === "pendding") {
        return "待支付";
      }
    },
    formatType(type) {
      if (type === "video") {
        return "视频";
      } else if (type === "column") {
        return "专栏";
      } else if (type === "audio") {
        return "音频";
      } else if (type === "media") {
        return "图文";
      }
    }
  },
  methods: {
    //   获取数据列表
    async getMyListQuery() {
      let mes;
      if (this.activeName === "first") {
        mes = await fetchUserCourse();
      } else if (this.activeName === "second") {
        mes = await fetchUserColumn();
      } else if (this.activeName === "third") {
        mes = await fetchUserOrder();
      } else if (this.activeName === "fourth") {
        mes = await fetchUserHistory();
      } else if (this.activeName === "fifth") {
        mes = await fetchUserComment();
      }
      if (mes.code === 20000) {
        this.total = mes.data.total;
        this.listQuery = mes.data.items;
      }
      console.log(this.listQuery);
    },
    //   切换选项卡
    handleClick(tab, event) {
      this.activeName = tab.paneName;
      this.search.page = 1;
      this.getMyListQuery();
    }
  }
};
</script>

<style scoped>
.top {
  display: flex;
  flex-direction: column;
}
.oneLine,
.twoLine {
  display: flex;
  justify-content: space-around;
}
.oneLine div,
.twoLine div {
  width: 260px;
  margin: 12px 0;
}
</style>
