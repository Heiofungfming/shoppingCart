<template>
    <div>
       <!-- 面包屑导航区 -->
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>订单管理</el-breadcrumb-item>
            <el-breadcrumb-item>订单列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容"  clearable >
                        <el-button slot="append" icon="el-icon-search"  ></el-button>
                    </el-input>
                </el-col>
            </el-row>

            <el-table :data="orderList" border stripe>
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="订单编号" prop="order_number"></el-table-column>
                <el-table-column label="订单价格" prop="order_price" width="95px"></el-table-column>
                <el-table-column label="是否付款" prop="pay_status" width="70px">
                    <template slot-scope="scope">
                        <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
                        <el-tag type="danger" v-else>未付款</el-tag>
                    </template>
                </el-table-column>
                <el-table-column label="是否发货" prop="is_send" width="70px"></el-table-column>
                <el-table-column label="下单时间" prop="create_time" width="140px">
                    <template slot-scope="scope">
                        {{scope.row.create_time | dateFormat}}
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="130px">
                    <template slot-scope="">
                        <el-button
                        type="primary"
                        icon="el-icon-edit"
                        size="mini"
                        @click="showBox"
                        ></el-button>
                        <el-button
                        type="success"
                        icon="el-icon-location"
                        size="mini"
                        @click="showProgress"
                        ></el-button>
                    </template>
                </el-table-column>
            </el-table>

            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[5, 10, 15, 50]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total"
                background>
            </el-pagination>
        </el-card>

        <el-dialog
            title="修改地址"
            :visible.sync="addressDialogVisible"
            width="50%"
            @close="aaddressDialogClosed">
            <!-- 添加分类的表单 -->
            <el-form
                :model="addressForm"
                :rules="addressormRules"
                ref="addressFormRef"
                label-width="100px">
                <el-form-item label="省市区/县:" prop="address1">
                    <el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
                </el-form-item>
                 <el-form-item label="详细地址:" prop="address2">
                    <el-input v-model="addressForm.address2"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addressDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addressDialogVisible = false">确 定</el-button>
            </span>
       </el-dialog>
   
       <el-dialog
            title="物流进度"
            :visible.sync="progressDialogVisible"
            width="50%">
             <el-timeline>
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
import cityData from './citydata'

export default {
    data () {
        return {
            queryInfo:{
                query: '',
                pagenum: 1,
                pagesize: 10
            },
            total: 0,
            orderList: [],
            addressDialogVisible:false,
            addressForm: {
               address1:[],
               address2:'' 
            },
            addressormRules: {
                 address1:[{
                     required: true, message: '请输入省市区/县', trigger: 'blur' 
                 }],
                 address2:[{
                     required: true, message: '请输入详细地址', trigger: 'blur' 
                 }]
            },
            cityData,
            progressDialogVisible: false,
            progressInfo:[]
        }
    },
    created () {
        this.getOrderList()
    },
    methods: {
        async getOrderList(){
           const {data:res} = await this.$http.get('orders',{params: this.queryInfo})
        
            if (res.meta.status !== 200) {
                return this.$message.error('获取订单失败！')
            }

            console.log(res)
            this.total = res.data.total
            this.orderList = res.data.goods

        },
         handleSizeChange(newSize){
            this.queryInfo.pagesize = newSize;
            this.getOrderList();
        },
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage;
            this.getOrderList();
        },
        showBox() {
            this.addressDialogVisible = true
        },
        aaddressDialogClosed() {
            this.$refs.addressFormRef.resetFields();
        },
        async showProgress() {
            const { data:res} = await this.$http.get('/kuaidi/1106975712662')

            if (res.meta.status !==200) {
                 return this.$message.error("获取物流进度失败！");
            }

            this.progressInfo = res.data

            this.progressDialogVisible = true
            console.log(this.progressInfo)
        }
    },
    computed:{
        // orderId() {
        //     return orderList
        // }
    }
}
</script>

<style lang="less" scoped>
.el-cascader{
    width: 100%;
}
</style>