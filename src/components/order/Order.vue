<template>
  <div>
    <!-- 面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!--订单列表数据-->
      <el-table :data="orderlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">{{ scope.row.create_time | dataFormat }}</template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showBox"></el-button>
            <el-button type="success" icon="el-icon-location" size="mini" @click="showProgressBox"></el-button>

          </template>
        </el-table-column>
      </el-table>
      <!--分页区域-->
      <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[5, 10, 15]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
      </el-pagination>
    </el-card>

    <!--修改地址的对话框-->
    <el-dialog
        title="修改地址"
        :visible.sync="addressVisible"
        width="50%" @close="addressDialogClosed">
      <el-form :model="addressForm" :rules="addressFormrules" ref="addressFormRef" label-width="100px">
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader :options="citydata" v-model="addressForm.address1"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addressVisible = false">取 消</el-button>
    <el-button type="primary" @click="addressVisible = false">确 定</el-button>
  </span>
    </el-dialog>
    <!--展示物流进度的对话框-->
    <el-dialog
        title="物流进度"
        :visible.sync="progressVisible"
        width="50%">
<!--时间线-->
      <el-timeline :reverse="reverse">
        <el-timeline-item
            v-for="(activity, index) in progressInfo"
            :key="index"
            :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>

    </el-dialog>
  </div>
</template>

<script>
import citydata from "./citydata";

export default {
  name: "Order",
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormrules: {
        address1: [{required: true, message: '请选择省市区/县', trigger: 'blur'}],
        address2: [{required: true, message: '请填写详细地址', trigger: 'blur'}]
      },
      citydata,
      progressVisible:false,

      progressInfo: []

    }

  },
  created() {
    this.getOrderList()

  },
  methods: {
    async getOrderList() {
      const {data: res} = await this.$http.get('orders', {params: this.queryInfo})
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')

      }
      console.log(res)
      this.total = res.data.total
      this.orderlist = res.data.goods
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()

    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    //展示修改地址的对话框
    showBox() {
      this.addressVisible = true


    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
    },
   async showProgressBox() {
    const {data:res} = await this.$http.get('../../static/progress.json')
     // 单号没效的用本地文件 await this.$http.get('../../static/progress.json')
     if (res.meta.status !== 200) {
       return this.$message.error('获取物流进度失败')
     }
      this.progressInfo = res.data
      this.progressVisible = true
     console.log(this.progressInfo)

    }

  }
}
</script>

<style lang="less" scoped>
@import "../../plugins/timeline/timeline.css";
@import "../../plugins/timeline-item/timeline-item.css";

.el-cascader {
  width: 100%;
}

</style>