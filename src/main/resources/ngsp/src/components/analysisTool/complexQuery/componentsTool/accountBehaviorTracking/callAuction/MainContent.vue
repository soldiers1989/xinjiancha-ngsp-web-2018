<!--集合竞价-->
<template>
  <div class="callAuction">
    <export-btn @handleExportData="handleExportData"></export-btn>
    <common-table-pagination-component
      :loading="loading"
      :tableData="tableData"
      :columns="tableColumns"
      :pagination="pagination"
      :totalNum="totalNum"
      @handlePaginationChange="handlePaginationChange"
    ></common-table-pagination-component>
  </div>
</template>
<script>
import { mapGetters } from 'vuex'
import commonTablePaginationComponent from '../../common/commonTablePaginationComponent'
import { callAuction, exportCallAuction } from '@/service/analysisTool/complexQuery/accountBehaviorTracking'
import {tableColumns} from './tableColumns'
export default {
  components: {
    commonTablePaginationComponent,
    ExportBtn: () => import('../../common/ExportButtonComponent')
  },
  computed: {
    ...mapGetters(['queryTrackForm4'])
  },
  watch: {
    'queryTrackForm4': {
      handler() {
        this.queryData = this.queryTrackForm4
        this.reqTableData(1)
      },
      deep: true
    }
  },
  data() {
    return {
      queryData: {},
      tableColumns,
      tableData: [],
      totalNum: 0,
      pagination: {
        currentPage: 1,
        pageSizeNumber: 10
      },
      loading: false
    }
  },
  methods: {
    handlePaginationChange(pagination) {
      this.pagination = pagination
      this.reqTableData(0)
    },
    reqTableData(type) {
      let params = {
        date: this.queryData.date,
        secCode: this.queryData.secCode,
        orderNo: this.queryData.orderNo,
        ngspUid: type ? '' : this.currentngspUid, // 后台上一次返回的ngspUid，如果是isNewQuery，置空
        pageIndex: this.pagination.currentPage,
        pageRows: this.pagination.pageSizeNumber,
        isExport: 0, // 1代表导出报表
        exportType: '', // excel,csv
        isNewQuery: type // 判断是否是新查询,1是新查询，0是分页或导出
      }
      this.loading = true
      callAuction(params).then(resp => {
        this.loading = false
        if (resp) {
          this.currentngspUid = resp.ngsp_uid
          this.tableData = resp.data
          this.totalNum = Number(resp.totalRowCount)
        } else {
          this.currentngspUid = ''
          this.tableData = []
        }
      })
    },
    handleExportData(type) {
      if (!this.currentngspUid) {
        this.$message.error('暂无数据')
        return
      }
      let params = {
        exportType: type,
        isExport: 1,
        isNewQuery: 0,
        ngspUid: this.currentngspUid,
        tableColumns: this.tableColumns
      }
      exportCallAuction(params)
    }
  }
}
</script>
<style lang="less" scoped>
  .callAuction {

  }
</style>
