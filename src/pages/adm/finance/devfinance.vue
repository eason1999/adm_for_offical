<template>
  <div class="finance-dev-wrapper">
    <h2>DEV财务信息</h2>
    <div class="dev-top-wrapper clearfix">
      <div class="dev-search pull-right">
        <el-input placeholder="请输入内容" v-model="keyword" @keyup.native.enter="load()"></el-input>
        <el-button type="primary" @click="load()">搜索</el-button>
      </div>
    </div>
    <el-table :data="tableData" stripe style="width: 100%" v-loading.fullscreen.lock="loadings" element-loading-text="拼命加载中">
      <el-table-column label="日期" sortable show-overflow-tooltip>
        <template scope="scope">
          <span>{{scope.row.submitTime | date }}</span> 
        </template>
      </el-table-column>
      <el-table-column prop="devName" label="公司名称" show-overflow-tooltip></el-table-column>
      <el-table-column prop="accountType" label="账户类型" show-overflow-tooltip></el-table-column>
      <el-table-column prop="bankName" label="开户行名称" show-overflow-tooltip></el-table-column>
      <el-table-column prop="accountNo" label="银行账号" show-overflow-tooltip></el-table-column>
      <el-table-column label="证件预览">
        <template scope="scope">
          <el-button type="info" size="small" @click="previewImg(scope.row.fileUrl)">预览</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="pager-wrapper clearfix" v-if="tableData.length">
      <pager :total-records="totalRecords" @pagechange="load" :page-sizes="pageSize" :page-nums="pageNum"></pager>
    </div>
    <el-dialog v-model="dialogVisible" size="tiny">
      <img width="100%" :src="dialogImageUrl">
    </el-dialog>
  </div>
</template>

<script type="ecmascript-6">
import pager from '../../../components/pager/pager.vue';
export default {
  data () {
    return {
      tableData: [],
      totalRecords: -1,
      pageNum: 1,
      pageSize: 20,
      keyword: '',
      dialogVisible: false,
      dialogImageUrl: 'about:blank;',
      loadings: false
    };
  },
  mounted () {
    this.$nextTick(() => {
      this.load();
    });
  },
  components: { pager },
  methods: {
    previewImg (fileUrl) {
      this.$nextTick(() => {
        this.dialogImageUrl = '';
        this.dialogImageUrl = fileUrl;
        this.dialogVisible = true;
      });
    },
    load (pageNum, pageSize) {
      let params = {};
      params.keyword = this.keyword;
      params.pageNum = pageNum || this.pageNum;
      params.pageSize = pageSize || this.pageSize;
      this.loadings = true;
      this.$http.get('/v1/adm/dev/finance/{pageNum}/{pageSize}', {params: params}).then((res) => {
        this.loadings = false;
        let data = res.body;
        if (data.ret!=1) {
          return this.$alert(data.message, '提示：', {
            confirmButtonText: '确定'
          });
        }
        let result = data.result;
        this.tableData = result.results;
        this.pageSize = result.pageSize;
        this.totalRecords = result.totalRecords;
      }, () => {this.loadings = false;});
    }
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .finance-dev-wrapper
    overflow: hidden
    .dev-top-wrapper
      margin-bottom: 15px
      .dev-search
        max-width: 300px
        font-size: 0
        .el-input
          width: 230px
          input
            border-radius: 4px 0 0 4px
        button 
          border-radius: 0 4px 4px 0
    .pager-wrapper
      margin-top: 15px      
</style>
