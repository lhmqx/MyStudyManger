<template>
  <div class="mainContent">
    <div class="top">
      <img :src="columnMes.cover" alt="" />
      <div class="content">
        <div class="title">
          <h4>{{ columnMes.title }}</h4>
          <span>{{ columnMes.isend === 0 ? "已完结" : "连载中" }}</span>
        </div>
        <p>{{ columnMes.content }}</p>
        <span style="color:red;">￥{{ columnMes.price }}</span>
        <div class="btn">
          <el-button type="warning" @click="handleChangeStatus">{{
            columnMes.status === 0 ? "上架" : "下架"
          }}</el-button>
          <el-button type="primary" plain @click="handleChangeIsEnd">{{
            columnMes.isend === 0 ? "设为连载中" : "设为已完结"
          }}</el-button>
        </div>
      </div>
    </div>

    <div class="search">
      <el-button type="primary" @click="handleAddDialog">
        <i class="el-icon-edit" />
        <span>新增目录</span>
      </el-button>
      <div class="search_inp">
        <el-form ref="form">
          <el-select v-model="search.status" placeholder="目录状态">
            <el-option label="已上架" :value="1"></el-option>
            <el-option label="已下架" :value="0"></el-option>
          </el-select>
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

    <!-- 表格部分 -->
    <div class="app-container">
      <el-table
        ref="dragTable"
        :data="columnList"
        row-key="id"
        border
        fit
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column align="center" label="ID" width="65">
          <template slot-scope="{ row, $index }">
            <span>{{ $index + 1 }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="单品内容">
          <template slot-scope="{ row }">
            <div class="context">
              <img style="width:100px;height:50px" :src="row.cover" alt="" />
              <div class="price">
                <div>{{ row.title }}</div>
                <div>￥{{ row.price }}</div>
              </div>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="订阅量" width="100" align="center">
          <template slot-scope="{ row }">
            <span>{{ row.sub_count }}</span>
          </template>
        </el-table-column>

        <el-table-column label="状态" width="100" align="center">
          <template slot-scope="{ row }">
            <el-tag :type="row.status === 0 ? 'danger' : 'success'">
              {{ row.status === 0 ? "已下架" : "已上架" }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column
          label="操作"
          align="center"
          width="300"
          class-name="small-padding fixed-width"
        >
          <template slot-scope="{ row, $index }">
            <el-button type="primary" size="mini">
              编辑
            </el-button>
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(row, $index)"
            >
              删除
            </el-button>
          </template>
        </el-table-column>

        <el-table-column align="center" label="Drag" width="80">
          <template slot-scope="{}">
            <svg-icon class="drag-handler" icon-class="drag" />
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 添加目录对话框 -->
    <add-dialog
      v-if="isAdd"
      :isAdd="isAdd"
      :closeDialog="closeDialog"
      :handleAdd="handleAdd"
    ></add-dialog>
  </div>
</template>

<script>
import { fetchDetailCourse } from "@/api/column";
import Sortable from "sortablejs";
import AddDialog from "@/components/Add/AddDialog.vue";
export default {
  name: "column_detail",
  data() {
    return {
      columnMes: {},
      columnList: [],
      total: 0,
      search: {
        page: 1,
        limit: 5,
        status: 0,
        title: ""
      },
      sortable: null,
      oldList: [],
      newList: [],
      isAdd: false
    };
  },
  components: {
    AddDialog
  },
  mounted() {
    this.columnMes = JSON.parse(this.$route.query.columnMes);
    // console.log(this.columnMes);
    // 获取专栏列表
    this.getColumnList();
  },
  methods: {
    // 获取专栏列表
    async getColumnList() {
      let mes = await fetchDetailCourse(this.search);
      if (mes.code === 20000) {
        this.total = mes.data.total;
        this.columnList = mes.data.items;
        this.oldList = this.columnList.map(v => v.id);
        this.newList = this.oldList.slice();
        this.$nextTick(() => {
          this.setSort();
        });
      }
      //   console.log(this.columnList);
    },

    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll(
        ".el-table__body-wrapper > table > tbody"
      )[0];
      this.sortable = Sortable.create(el, {
        ghostClass: "sortable-ghost", // Class name for the drop placeholder,
        setData: function(dataTransfer) {
          dataTransfer.setData("Text", "");
        },
        onEnd: evt => {
          const targetRow = this.columnList.splice(evt.oldIndex, 1)[0];
          this.columnList.splice(evt.newIndex, 0, targetRow);

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0];
          this.newList.splice(evt.newIndex, 0, tempIndex);
        }
      });
    },

    // 搜索数据
    handleSearch() {
      this.getColumnList();
    },

    //   上架与下架转换
    handleChangeStatus() {
      this.columnMes.status = this.columnMes.status === 0 ? 1 : 0;
    },

    //   完结状态改变
    handleChangeIsEnd() {
      this.columnMes.isend = this.columnMes.isend === 0 ? 1 : 0;
    },

    // 新增目录
    handleAddDialog() {
      this.isAdd = true;
    },

    // 确认新增
    handleAdd(mes) {
      let arr = [...this.columnList, ...mes];
      this.columnList = arr;
      this.isAdd = false;
    },

    // 关闭对话框
    closeDialog() {
      this.isAdd = false;
    },

    // 删除
    handleDelete(row, $index) {}
  }
};
</script>

<style>
.sortable-ghost {
  opacity: 0.8;
  color: #fff !important;
  background: #42b983 !important;
}
</style>

<style scoped>
.mainContent {
  padding: 20px;
}
.top {
  border: 1px solid #ccc;
  padding: 10px;
  display: flex;
}
.top img {
  width: 200px;
  height: 150px;
  padding: 10px;
  margin-right: 10px;
}
.content p {
  font-size: 14px;
  margin: 0 0 10px;
  color: #aaa;
}
.title {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.title span {
  font-size: 14px;
  color: #aaa;
}
.btn {
  margin-top: 10px;
  display: flex;
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

/* 表格部分 */
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
.icon-star {
  margin-right: 2px;
}
.drag-handler {
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.show-d {
  margin-top: 15px;
}
</style>
