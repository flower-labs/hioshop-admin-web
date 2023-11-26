<template>
  <div class="content-page">
    <div class="content-nav">
      <el-breadcrumb class="breadcrumb" separator="/">
        <el-breadcrumb-item>预约列表</el-breadcrumb-item>
      </el-breadcrumb>
      <div class="operation-nav"></div>
    </div>
    <div class="content-main">
      <div class="form-table-box">
        <el-table :data="tableData" style="width: 100%" border stripe>
          <el-table-column prop="id" label="ID" width="70px"></el-table-column>
          <el-table-column
            prop="reserve_name"
            label="服务类型"
          ></el-table-column>
          <el-table-column
            prop="reserve_time"
            :formatter="dateFormat"
            label="预约时间"
          ></el-table-column>
          <el-table-column
            prop="reserve_price"
            label="预约价格"
          ></el-table-column>
          <el-table-column prop="enabled" label="预约状态" width="80px">
            <template scope="scope">
              {{ scope.row.status == 1001 ? "待确认" : "" }}
              {{ scope.row.status == 1002 ? "已确认" : "" }}
              {{ scope.row.status == 1003 ? "服务中" : "" }}
              {{ scope.row.status == 1004 ? "待评价" : "" }}
              {{ scope.row.status == 1005 ? "已经评价" : "" }}
              {{ scope.row.status == 1006 ? "服务取消" : "" }}
              {{ scope.row.status == 1007 ? "服务关闭" : "" }}
            </template>
          </el-table-column>
          <el-table-column prop="plate_number" label="车牌号"></el-table-column>
          <el-table-column prop="phone_number" label="手机号"></el-table-column>
          <el-table-column label="操作" width="250">
            <template scope="scope">
              <el-button size="small" @click="onStatusChange(scope.row)"
                >更新状态</el-button
              >
              <el-button size="small" type="danger" disabled>删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="page-box">
        <el-pagination
          @current-change="handlePageChange"
          :current-page="page"
          :page-size="10"
          layout="total, prev, pager, next, jumper"
          :total="total"
        >
        </el-pagination>
      </div>
      <el-dialog
        title="请选择更新后的状态"
        :visible.sync="dialogVisible"
        width="30%"
        :before-close="handleClose"
      >
        <el-select v-model="selectedValue" placeholder="请选择最新状态">
          <el-option
            v-for="item in selectOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="handleStatusChange"
            >确 定</el-button
          >
        </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import moment from "moment";
export default {
  data() {
    return {
      page: 1,
      total: 0,
      currentRecordId: "",
      selectedValue: 1002,
      selectOptions: [
        {
          value: 1002,
          label: "已确认",
        },
        {
          value: 1003,
          label: "服务中",
        },
        {
          value: 1004,
          label: "待评价",
        },
      ],
      dialogVisible: false,
      filterForm: {
        name: "",
      },
      tableData: [],
    };
  },
  methods: {
    handlePageChange(val) {
      this.page = val;
      //保存到localStorage
      localStorage.setItem("adPage", this.page);
      localStorage.setItem("adFilterForm", JSON.stringify(this.filterForm));
      this.getList();
    },
    onStatusChange(row) {
      this.currentRecordId = row.id;
      this.dialogVisible = true;
    },
    handleStatusChange() {
      this.axios
        .post("reserve/updateReserveStatus", {
          id: this.currentRecordId,
          status: this.selectedValue,
        })
        .then((response) => {
          console.log("response", response);
          if (response.data.errno === 0) {
            this.$message({
              type: "success",
              message: "状态更新成功!",
            });
            this.dialogVisible = false;
            this.getList();
          }
        });
    },
    getList() {
      this.axios
        .get("reserve", {
          params: {
            page: this.page,
          },
        })
        .then((response) => {
          this.tableData = response.data.data.data;
          this.page = response.data.data.currentPage;
          this.total = response.data.data.count;
        });
    },
    //时间格式化
    dateFormat: function (row, column) {
      var date = row[column.property];
      if (date == undefined) {
        return "";
      }
      return moment(date * 1000).format("YYYY-MM-DD HH:mm:ss");
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
  },
  components: {},
  mounted() {
    this.getList();
  },
};
</script>

<style scoped>
</style>
