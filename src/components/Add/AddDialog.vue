<template>
  <div>
    <el-dialog
      title="提示"
      :visible.sync="isAdd"
      width="70%"
      :before-close="closeDialog"
      top="5vh"
    >
      <div class="search">
        <el-form ref="form">
          <el-button type="primary" @click="handleSearch">
            <i class="el-icon-search" />
            <span>搜索</span>
          </el-button>
          <el-form-item width="100">
            <el-input v-model="search.title" placeholder="标题"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <div class="tabMes">
        <div class="menu">
          <el-menu
            :default-active="checkIndex"
            class="el-menu-vertical-demo"
            background-color="#eee"
            active-text-color="#2d89da"
            @select="handleChangeIndex"
          >
            <el-menu-item index="1">
              <span slot="title">图文</span>
            </el-menu-item>
            <el-menu-item index="2">
              <span slot="title">音频</span>
            </el-menu-item>
            <el-menu-item index="3">
              <span slot="title">视频</span>
            </el-menu-item>
          </el-menu>
        </div>
        <div>
          <el-table
            :key="0"
            :data="mesList"
            border
            fit
            highlight-current-row
            style="width: 100%;"
            @selection-change="handleSelectionChange"
          >
            <el-table-column type="selection" width="80"> </el-table-column>
            <el-table-column label="内容">
              <template slot-scope="{ row }">
                <div class="context">
                  <img
                    style="width:100px;height:50px"
                    :src="row.cover"
                    alt=""
                  />
                  <div class="price">
                    <div>{{ row.title }}</div>
                    <div>￥{{ row.price }}</div>
                  </div>
                </div>
              </template>
            </el-table-column>
          </el-table>
          <pagination
            v-show="total > 0"
            :total="total"
            :page.sync="search.page"
            :limit.sync="search.limit"
            @pagination="getMesList"
          />
        </div>
      </div>

      <span slot="footer" class="dialog-footer">
        <el-button @click="closeDialog">取 消</el-button>
        <el-button type="primary" @click="handleAdd(checkList)"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { fetchList as media } from "@/api/media";
import { fetchList as video } from "@/api/video";
import { fetchList as audio } from "@/api/audio";
import Pagination from "@/components/Pagination";
let defaultSearch = {
  page: 1,
  limit: 5,
  title: ""
};
export default {
  name: "addDialog",
  props: ["isAdd", "closeDialog", "handleAdd"],
  data() {
    return {
      mesList: [],
      checkIndex: "1",
      total: 0,
      search: Object.assign(defaultSearch, {}),
      checkList: []
    };
  },
  components: {
    Pagination
  },
  mounted() {
    //   获取图文数据
    this.getMesList();
  },
  methods: {
    //   获取数据
    async getMesList() {
      let mes;
      this.search.page = 1;
      if (this.checkIndex === "1") {
        mes = await media(this.search);
      } else if (this.checkIndex === "2") {
        mes = await audio(this.search);
      } else if (this.checkIndex === "3") {
        mes = await video(this.search);
      }
      if (mes.code === 20000) {
        this.total = mes.data.total;
        this.mesList = mes.data.items;
      }
    },

    // 菜单选择
    handleChangeIndex(index) {
      this.checkIndex = index;
      this.getMesList();
    },

    //   搜索
    handleSearch() {
        this.getMesList();
    },

    // 改变表格选择
    handleSelectionChange(val){
        this.checkList = val;
    },


  }
};
</script>

<style scoped>
.search form {
  margin: 20px auto;
  display: flex;
  height: 36px;
  flex-direction: row-reverse;
}
.tabMes {
  display: flex;
}
.menu {
  width: 200px;
  height: 360px;
  background-color: #eee;
}
.context {
  display: flex;
}
.price {
  text-align: left;
  display: flex;
  flex-direction: column;
}
.price div:nth-child(2) {
  color: red;
}
</style>
