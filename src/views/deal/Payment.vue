<template>
  <div class="payMentMain">
    <div class="top">
      <el-button type="primary" @click="handleAddDialog">
        <i class="el-icon-edit" />
        <span>新增收款账号</span>
      </el-button>
    </div>
    <div class="payMentTable">
      <el-table
        v-loading="listLoading"
        :data="payMentList"
        border
        fit
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column align="center" label="账号">
          <template slot-scope="{ row }">
            <span>{{ row.account }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="开户人">
          <template slot-scope="{ row }">
            <span>{{ row.name }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="开户行">
          <template slot-scope="{ row }">
            <span>{{ row.path }}</span>
          </template>
        </el-table-column>

        <el-table-column
          label="操作"
          align="center"
          width="240"
          class-name="small-padding fixed-width"
        >
          <template slot-scope="{ row, $index }">
            <el-button type="primary" size="mini" @click="handleEdit(row)">
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
      </el-table>
      <pagination
        v-show="total > 0"
        :total="total"
        :page.sync="page.page"
        :limit.sync="page.limit"
        @pagination="getPayMentList"
      />
    </div>

    <!-- 表单 -->
    <el-dialog
      :title="isEdit ? '编辑' : '新增'"
      :visible.sync="dialogFormVisible"
      :before-close="handleCloseAdd"
    >
      <el-form :model="form" label-width="120px" :rules="rules" ref="form">
        <el-form-item label="账号" prop="account">
          <el-input style="width:400px" v-model="form.account"></el-input>
        </el-form-item>
        <el-form-item label="活动区域">
          <el-select
            v-model="form.province"
            @change="changeProvince"
            placeholder="请选择省"
            style="width:120px;margin-right:20px"
          >
            <el-option
              v-for="item in options.provinces"
              :label="item.AddName"
              :value="item.AddName"
              :key="item.id"
            ></el-option>
          </el-select>
          <el-select
            v-model="form.city"
            @change="changeCity"
            placeholder="请选择市"
            style="width:120px;margin-right:20px"
          >
            <el-option
              v-for="item in options.city"
              :label="item.AddName"
              :value="item.AddName"
              :key="item.id"
            ></el-option>
          </el-select>
          <el-select
            v-model="form.area"
            placeholder="请选择区"
            style="width:120px"
          >
            <el-option
              v-for="item in options.area"
              :label="item.AddName"
              :value="item.AddName"
              :key="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="详细地址" prop="path">
          <el-input style="width:400px" v-model="form.path"></el-input>
        </el-form-item>
        <el-form-item label="所属银行" prop="bank">
          <el-select
            v-model="form.bank"
            placeholder="请选择所属银行"
            style="width:400px"
          >
            <el-option label="中国银行" value="中国银行"></el-option>
            <el-option label="中原银行" value="中原银行"></el-option>
            <el-option label="建设银行" value="建设银行"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input style="width:400px" v-model="form.name"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="handleCloseAdd">取 消</el-button>
        <el-button type="primary" @click="handleEditOrAdd">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { fetchPayments } from "@/api/pay";
import Pagination from "@/components/Pagination";
import address from "@/tools/address";
const defaultForm = {
  account: "",
  province: "",
  city: "",
  area: "",
  name: "",
  path: "",
  bank: ""
};
export default {
  name: "payMent",
  components: {
    Pagination
  },
  data() {
    return {
      payMentList: [],
      listLoading: false,
      total: 0,
      page: {
        page: 1,
        limit: 5
      },
      options: {
        provinces: [],
        city: [],
        area: []
      },
      form: Object.assign({}, defaultForm),
      rules: {
        account: [{ required: true, message: "请输入账号", trigger: "blur" }],
        path: [{ required: true, message: "请输入详细地址", trigger: "blur" }],
        bank: [{ required: true, message: "请输入所属银行", trigger: "blur" }],
        name: [{ required: true, message: "请输入姓名", trigger: "blur" }]
      },
      isEdit: false,
      dialogFormVisible: false,
      id: -1
    };
  },
  mounted() {
    // 获支付数据
    this.getPayMentList();
  },
  methods: {
    // 获取支付数据
    async getPayMentList() {
      this.listLoading = true;
      let mes = await fetchPayments(this.page);
      if (mes.code === 20000) {
        this.listLoading = false;
        this.payMentList = mes.data.items;
        this.total = mes.data.total;
      }
    },

    // 新增
    handleAddDialog() {
      this.form = Object.assign({}, defaultForm);
      this.dialogFormVisible = true;
      this.getProvince();
    },

    // 编辑
    handleEdit(row) {
      this.isEdit = true;
      this.dialogFormVisible = true;
      this.initForm(row);
    },

    // 关闭对话框
    handleCloseAdd() {
      this.dialogFormVisible = false;
    },

    // 删除
    handleDelete(row) {
      // console.log(row);
      let index = this.payMentList.findIndex(item => item.id === row.id);
      if (index !== -1) {
        this.$confirm("是否要删除该记录", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            this.payMentList.splice(index, 1);
            this.$notify({
              title: "提示",
              message: "删除成功",
              type: "success"
            });
          })
          .catch(err => {});
      }
    },

    // 省市县三级联动
    // 渲染省
    getProvince() {
      address.site_address.forEach(item => {
        if (item.TopID === 0) {
          this.options.provinces.push(item);
        }
      });
    },

    // 渲染市
    getCity(provinceAddName) {
      this.options.city = [];
      this.options.area = [];
      this.form.city = "";
      this.form.area = "";
      let index = address.site_address.findIndex(
        item => item.AddName === provinceAddName
      );
      if (index != -1) {
        address.site_address.forEach(item => {
          if (item.TopID === address.site_address[index].ID) {
            this.options.city.push(item);
          }
        });
      }
    },

    // 渲染县
    getArea(cityAddName) {
      this.options.area = [];
      let index = address.site_address.findIndex(
        item => item.AddName === cityAddName
      );
      address.site_address.forEach(item => {
        if (item.TopID === address.site_address[index].ID) {
          this.options.area.push(item);
        }
      });
    },

    // 省改变
    changeProvince(item) {
      this.getCity(item);
    },

    // 市改变
    changeCity(item) {
      this.getArea(item);
    },

    // 编辑或新增提交
    handleEditOrAdd() {
      this.$refs.form.validate(valid => {
        if (valid) {
          if (!this.form.province || !this.form.city || !this.form.area) {
            return this.$message.error("省市县信息不能为空");
          }
          let obj = {
            account: this.form.account.replace(
              this.form.account.substr(4, 10),
              "**********"
            ),
            area: this.form.area,
            bank: this.form.bank,
            city: this.form.city,
            created_time: this.$moment(new Date()).format(
              "YYYY-MM-DD HH:mm:ss"
            ),
            id: this.payMentList[this.payMentList.length - 1].id + 1,
            name: this.form.name,
            path: this.form.path,
            province: this.form.province,
            status: 1
          };
          if (this.isEdit) {
            // 编辑模式
            obj.id = this.id;
            let index = this.payMentList.findIndex(item => item.id === this.id);
            if (index != -1) {
              this.payMentList[index] = { ...obj };
            }
            this.$notify({
              title: "Success",
              message: "编辑收款账号成功",
              type: "success"
            });
            this.isEdit = false;
          } else {
            // 新增模式
            this.payMentList.push(obj);
            this.$notify({
              title: "Success",
              message: "新增收款账号成功",
              type: "success"
            });
          }
          this.dialogFormVisible = false;
        }
      });
    },

    // 初始化form
    initForm(obj) {
      if (obj) {
        this.form.account = obj.account;
        this.form.name = obj.name;
        this.form.path = obj.path;
        this.form.bank = obj.bank;
        this.id = obj.id;
        this.getProvince();
        this.form.province = obj.province;
        this.getCity(this.form.province);
        this.form.city = obj.city;
        this.getArea(this.form.city);
        this.form.area = obj.area;
      }
    }
  }
};
</script>

<style scoped>
.payMentMain {
  padding: 20px;
}
.payMentTable {
  margin-top: 20px;
}
</style>
