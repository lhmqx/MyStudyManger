<template>
  <div class="content">
    <div class="search">
      <el-button type="primary" @click="handleExport">
        <span>导出选中</span>
      </el-button>
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
      :data="orderList"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="40"> </el-table-column>

      <el-table-column label="订单号" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>

      <el-table-column label="商品名称" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.goods[0].title }}</span>
        </template>
      </el-table-column>

      <el-table-column label="订单类型" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.type === "default" ? "普通" : "拼团" }}</span>
        </template>
      </el-table-column>

      <el-table-column label="订单状态" width="100" align="center">
        <template slot-scope="{ row }">
          <el-tag :type="row.status === 'success' ? 'success' : 'danger'">
            {{ row.status | filterStatus }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column label="原价/实付(元)" width="140" align="center">
        <template slot-scope="{ row }">
          <span style="color:#aaa">{{ row.total_price }}/</span>
          <span style="color:red">{{ row.price }}</span>
        </template>
      </el-table-column>

      <el-table-column label="支付方式" width="100" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.pay_method === "wechat" ? "微信支付" : "" }}</span>
        </template>
      </el-table-column>

      <el-table-column label="创建/支付时间" width="160" align="center">
        <template slot-scope="{ row }">
          <div>{{ row.created_time }}</div>
          <div>{{ row.pay_time }}</div>
        </template>
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        width="100"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
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
      @pagination="getOrderList"
    />
  </div>
</template>

<script>
import { fetchOrder, deleteOrder } from "@/api/pay";
import Pagination from "@/components/Pagination";
import { parseTime } from "@/utils";
const defaultRuleForm = {
  title: "",
  cover: "",
  try: "",
  content: "",
  price: 5,
  status: 0
};
export default {
  name: "order",
  components: {
    Pagination
  },
  data() {
    return {
      orderList: [],
      total: 0,
      search: {
        page: 1,
        limit: 5,
        title: ""
      },
      filename: "excel-list",
      autoWidth: true,
      bookType: "xlsx",
      list: []
    };
  },
  mounted() {
    // 获取图片数据
    this.getOrderList();
  },
  filters: {
    filterStatus(status) {
      if (status === "success") return "成功";
      if (status === "pendding") return "待支付";
      if (status === "fail") return "失败";
    }
  },
  methods: {
    // 获取图片数据
    async getOrderList() {
      let mes = await fetchOrder(this.search);
      if (mes.code === 20000) {
        this.total = mes.data.total;
        this.orderList = mes.data.items;
      }
      // console.log(this.orderList);
    },

    // 搜索数据
    handleSearch() {
      this.getOrderList();
    },

    // 选择数据
    handleSelectionChange(options) {
      options.forEach(item => {
        let obj = {
          id: item.id,
          title: item.goods[0].title,
          type: item.type === "default" ? "普通" : "拼团",
          status: this.forMapStatus(item.status),
          total_price: item.total_price,
          price: item.price,
          pay_method: item.pay_method === "wechat" ? "微信支付" : "其他",
          created_time: item.created_time,
          pay_time: item.pay_time
        };
        this.list.push(obj);
      });
    },

    // 格式化状态
    forMapStatus(status) {
      if (status === "success") return "成功";
      if (status === "pendding") return "待支付";
      if (status === "fail") return "失败";
    },

    // 删除
    handleDelete(row, a) {
      // let status = row.status === 0 ? 1 : 0;
      this.$confirm("是否要删除该记录", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        let index = this.orderList.findIndex(item => item == row);
        if (index != -1) {
          this.orderList.splice(index, 1);
        }
        // let mes = await deleteOrder({ status, index });
        this.$notify({
          title: "提示",
          message: "删除成功",
          type: "success"
        });
      });
    },

    // 导出文件
    handleExport() {
      import("@/vendor/Export2Excel").then(excel => {
        const tHeader = [
          "订单号",
          "商品名称",
          "订单类型",
          "订单状态",
          "原价(元)",
          "实付(元)",
          "支付方式",
          "创建时间",
          "支付时间"
        ];
        const filterVal = [
          "id",
          "title",
          "type",
          "status",
          "total_price",
          "price",
          "pay_method",
          "created_time",
          "pay_time"
        ];
        const list = this.list;
        const data = this.formatJson(filterVal, list);
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: this.filename,
          autoWidth: this.autoWidth,
          bookType: this.bookType
        });
      });
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v =>
        filterVal.map(j => {
          if (j === "timestamp") {
            return parseTime(v[j]);
          } else {
            return v[j];
          }
        })
      );
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
