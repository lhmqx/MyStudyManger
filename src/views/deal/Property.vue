<template>
  <div class="protertyMain">
    <div class="top">
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span>可用余额（元）</span>
          <el-button type="primary" @click="handleaApply">申请提现</el-button>
        </div>
        <div class="price">
          <h1>20.00</h1>
        </div>
      </el-card>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span>累计收入（元）</span>
        </div>
        <div class="price">
          <h1>20.00</h1>
        </div>
      </el-card>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span>待结算金额（元）</span>
        </div>
        <div class="price">
          <h1>21</h1>
        </div>
      </el-card>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span>冻结金额（元）</span>
        </div>
        <div class="price">
          <h1>20.00</h1>
        </div>
      </el-card>
    </div>
    <div class="propertyTable">
      <el-table
        v-loading="listLoading"
        :data="assets"
        border
        fit
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column align="center" label="交易时间">
          <template slot-scope="{ row }">
            <span>{{ row.created_time }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="提款账号">
          <template slot-scope="{ row }">
            <span>{{ row.account }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="开户人">
          <template slot-scope="{ row }">
            <span>{{ row.name }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="提现金额">
          <template slot-scope="{ row }">
            <span>{{ row.price }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="状态">
          <template slot-scope="{ row }">
            <el-tag :type="row.status !== 0 ? 'danger' : 'success'">{{
              row.status !== 0 ? "审核中" : "提现成功"
            }}</el-tag>
          </template>
        </el-table-column>
      </el-table>
      <pagination
        v-show="total > 0"
        :total="total"
        :page.sync="page.page"
        :limit.sync="page.limit"
        @pagination="getAsset"
      />
    </div>

    <el-dialog
      title="提现"
      :visible.sync="dialogVisible"
      width="60%"
      :before-close="handleClose"
    >
      <el-form :model="form" class="demo-form">
        <el-form-item label="提现金额">
          <el-input-number
            v-model="deposit.price"
            @change="handleChangePrice"
            :min="1"
          ></el-input-number>
        </el-form-item>
        <el-form-item label="提现账户">
          <el-select v-model="deposit.account" placeholder="请选择">
            <el-option
              label="中国银行     6216**********7287"
              value="6216**********7287"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleSubmit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { fetchAssets } from "@/api/pay";
import Pagination from "@/components/Pagination";
export default {
  name: "property",
  components: {
    Pagination
  },
  data() {
    return {
      assets: [],
      total: 0,
      listLoading: false,
      page: {
        page: 1,
        limit: 5
      },
      dialogVisible: false,
      deposit: {
        account: "",
        price: 0
      }
    };
  },
  mounted() {
    // 获取资产详情
    this.getAsset();
  },
  methods: {
    // 获取资产详情
    async getAsset() {
      this.listLoading = true;
      let mes = await fetchAssets(this.page);
      if (mes.code === 20000) {
        this.listLoading = false;
        this.assets = mes.data.items;
        console.log(this.assets);
        this.total = mes.data.total;
      }
    },
    // 申请提现
    handleaApply() {
      this.dialogVisible = true;
    },

    // 修改提现金额
    handleChangePrice() {},

    // 确认提交
    handleSubmit() {
      this.dialogVisible = false;
    }
  }
};
</script>

<style scoped>
.protertyMain {
  padding: 20px;
}
.top {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.box-card {
  width: 100%;
  margin-right: 20px;
  height: 120px;
  border: 1px solid #ddd;
}
.box-card:last-child {
  margin-right: 0;
}
.clearfix {
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.price {
  margin-top: -30px;
}

.propertyTable {
  margin-top: 40px;
}
</style>
