<template>
  <div class="content">
    <div class="search">
      <el-button type="primary" @click="handleAddDialog">
        <i class="el-icon-edit" />
        <span>新增专栏</span>
      </el-button>
      <div class="search_inp">
        <el-form ref="form">
          <el-select v-model="search.status" placeholder="专栏状态">
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
    <el-table
      :key="0"
      :data="columnList"
      border
      fit
      highlight-current-row
      style="width: 100%;"
    >
      <el-table-column
        label="ID"
        prop="id"
        width="80"
        sortable="custom"
        align="center"
      >
        <template slot-scope="{ row }">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>

      <el-table-column label="专栏内容" align="center">
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

      <el-table-column label="更新状态" width="100" align="center">
        <template slot-scope="{ row }">
          <span :style="{ color: row.isend === 0 ? 'black' : 'red' }">
            {{ row.isend === 0 ? "已完结" : "连载中" }}
          </span>
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
        width="300"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="warning" size="mini" @click="handleGetCatalog(row)">
            目录
          </el-button>
          <el-button type="primary" size="mini" @click="handleEdit(row)">
            编辑
          </el-button>
          <el-button
            size="mini"
            :type="row.status === 0 ? 'success' : ''"
            @click="handleChangeStatus(row, $index)"
          >
            {{ row.status === 0 ? "上架" : "下架" }}
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
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="search.page"
      :limit.sync="search.limit"
      @pagination="getcolumnList"
    />

    <!-- 添加信息 -->
    <el-dialog
      title="新增"
      :visible.sync="isAdd"
      width="100%"
      :before-close="handleCloseAdd"
      fullscreen
    >
      <div class="addForm">
        <el-form
          :model="ruleForm"
          :rules="rules"
          ref="ruleForm"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="标题" prop="title">
            <el-input v-model="ruleForm.title"></el-input>
          </el-form-item>

          <el-form-item label="封面">
            <el-upload
              action="https://jsonplaceholder.typicode.com/posts/"
              list-type="picture-card"
              :on-preview="handlePictureCardPreview"
              :on-remove="handleRemove"
            >
              <i class="el-icon-plus"></i>
            </el-upload>
            <el-dialog :visible.sync="dialogVisibleAdd">
              <img width="100%" :src="ruleForm.cover" alt="" />
            </el-dialog>
          </el-form-item>

          <el-form-item label="课程介绍">
            <el-input
              type="textarea"
              :autosize="{ minRows: 2, maxRows: 4 }"
              placeholder="请输入内容"
              v-model="ruleForm.try"
            >
            </el-input>
          </el-form-item>

          <el-form-item label="课程内容">
            <tinymce v-model="ruleForm.content"></tinymce>
          </el-form-item>

          <el-form-item label="课程价格">
            <el-input-number
              v-model="ruleForm.price"
              :min="1"
            ></el-input-number>
          </el-form-item>

          <el-form-item label="状态">
            <el-radio v-model="ruleForm.status" :label="0">下架</el-radio>
            <el-radio v-model="ruleForm.status" :label="1">上架</el-radio>
          </el-form-item>

          <el-form-item label="更新状态">
            <el-radio v-model="ruleForm.isend" :label="0">连载中</el-radio>
            <el-radio v-model="ruleForm.isend" :label="1">已完结</el-radio>
          </el-form-item>
        </el-form>
      </div>

      <span slot="footer" class="dialog-footer">
        <el-button @click="handleCloseAdd">取 消</el-button>
        <el-button type="primary" @click="handleSubmit">提 交</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {
  fetchList,
  updateColumn,
  createColumn,
  deleteColumn
} from "@/api/column";
import Pagination from "@/components/Pagination";
import Tinymce from "@/components/Tinymce";
const defaultRuleForm = {
  title: "",
  cover: "",
  try: "",
  content: "",
  price: 5,
  status: 0,
  isend: 0
};
export default {
  name: "column",
  components: {
    Pagination,
    Tinymce
  },
  data() {
    return {
      columnList: [],
      total: 0,
      search: {
        page: 1,
        limit: 5,
        status: 0,
        title: ""
      },
      isAdd: false,
      ruleForm: Object.assign(defaultRuleForm, {}),
      rules: {
        title: [{ required: true, message: "请输入标题", trigger: "blur" }]
      },
      dialogVisibleAdd: false,
      isEdit: false
    };
  },
  mounted() {
    // 获取视频数据
    this.getcolumnList();
  },
  methods: {
    // 获取视频数据
    async getcolumnList() {
      let mes = await fetchList(this.search);
      if (mes.code === 20000) {
        this.total = mes.data.total;
        this.columnList = mes.data.items;
      }
      console.log(this.columnList);
    },

    // 搜索数据
    handleSearch() {
      this.getcolumnList();
    },

    // 目录
    handleGetCatalog(row) {
      this.$router.push({
        path: "/course/column_detail",
        query: { columnMes: JSON.stringify(row) }
      });
    },

    // 上架与下架
    async handleChangeStatus(row, index) {
      let status = row.status === 0 ? 1 : 0;
      let mes = await updateColumn({ status, index });
      if (mes.code === 20000) {
        row.status = row.status === 0 ? 1 : 0;
        this.$message({
          message: `${row.status === 0 ? "上架成功" : "下架成功"}`,
          type: "success"
        });
        // this.getcolumnList();
      } else {
        this.$message({
          message: `${row.status === 0 ? "上架成功" : "下架成功"}`,
          type: "error"
        });
      }
    },

    // 删除
    async handleDelete(row, index) {
      let status = row.status === 0 ? 1 : 0;
      let mes = await deleteColumn({ status, index });
      if (mes.code === 20000) {
        this.$message({
          message: "删除成功",
          type: "success"
        });
        this.getcolumnList();
      } else {
        this.$message({
          message: "删除失败",
          type: "error"
        });
      }
    },

    // 新增视频对话框
    handleAddDialog() {
      this.isAdd = true;
    },

    // 关闭新增视频对话框
    handleCloseAdd() {
      this.isAdd = false;
      this.isEdit = false;
      this.ruleForm = Object.assign(defaultRuleForm, {});
    },

    // 图片上传
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },

    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisibleAdd = true;
    },

    // 开启编辑
    handleEdit(row) {
      this.isAdd = true;
      this.ruleForm = {
        title: row.title,
        cover: row.cover,
        try: row.try,
        content: row.content,
        price: row.price,
        status: row.status,
        isend: row.isend
      };
      this.isEdit = true;
    },

    // 提交编辑或添加
    handleSubmit() {
      if (this.isEdit) {
        this.$refs.ruleForm.validate(async valid => {
          if (valid) {
            let mes = await updateColumn(this.ruleForm);
            if (mes.code === 20000) {
              this.$message.success(mes.data);
              this.isAdd = false;
              this.isEdit = false;
              this.ruleForm = Object.assign(defaultRuleForm, {});
              this.getcolumnList();
            } else {
              this.$message.error("编辑数据失败");
            }
          } else {
            this.$message.error("必填项不能为空");
          }
        });
      } else {
        this.$refs.ruleForm.validate(async valid => {
          if (valid) {
            let mes = await createColumn(this.ruleForm);
            if (mes.code === 20000) {
              this.$message.success(mes.data);
              this.isAdd = false;
              this.getcolumnList();
            } else {
              this.$message.error("添加数据失败");
            }
          } else {
            this.$message.error("必填项不能为空");
          }
        });
      }
    }
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
}
.price {
  text-align: left;
  display: flex;
  flex-direction: column;
}
.price div:nth-child(2) {
  color: red;
}
.addForm {
  width: 60%;
}
</style>
