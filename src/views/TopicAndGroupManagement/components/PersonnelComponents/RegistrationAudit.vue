<template>
  <div class="registration-audit">
    <!-- 查询条件 -->
    <div class="search-form">
      <el-form :inline="true" :model="searchForm" class="demo-form-inline">
        <el-form-item label="审核状态:">
          <el-select v-model="searchForm.auditStatus" placeholder="请选择审核状态" clearable
            style="width: 150px;">
            <el-option label="待审核" value="1" />
            <el-option label="审核通过" value="2" />
            <el-option label="审核不通过" value="3" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchAudit">查询</el-button>
          <el-button @click="resetSearch">重置</el-button>
        </el-form-item>
      </el-form>
    </div>

    <!-- 报名审核表格 -->
    <el-table :data="tableData" border height="250" style="width: 100%" v-loading="loading"
      @selection-change="handleSelectionChange" ref="auditTable">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column prop="patientName" label="患者名称" align="center" />
      <el-table-column prop="followUpPerson" label="跟进人" align="center" />
      <el-table-column prop="registrationNo" label="登记号" align="center" />
      <el-table-column prop="age" label="年龄" align="center" />
      <el-table-column prop="enrollmentTime" label="入组时间" align="center" />
      <el-table-column prop="firstFollowUpTime" label="首次随访时间" align="center" />
      <el-table-column prop="contactInfo" label="联系方式" align="center" />
      <el-table-column label="审核状态" align="center">
        <template slot-scope="scope">
          <el-tag :type="getAuditStatusType(scope.row.auditStatus)" size="small">
            {{ getAuditStatusText(scope.row.auditStatus) }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button type="text" @click="handleDetail(scope.row)">详情</el-button>
          <el-button type="text" @click="handleFollowUpRecord(scope.row)">跟进记录</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页组件 -->
    <div class="pagination-wrapper">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="pagination.current" :page-sizes="[10, 20, 30, 40]"
        :page-size="pagination.size" layout="total, sizes, prev, pager, next, jumper"
        :total="pagination.total">
      </el-pagination>
    </div>

    <!-- 审核操作按钮 -->
    <div class="audit-actions">
      <el-button type="success" @click="handleBatchAudit('approve')"
        :disabled="selectedRows.length === 0">
        审核通过
      </el-button>
      <el-button type="danger" @click="handleBatchAudit('reject')"
        :disabled="selectedRows.length === 0">
        审核不通过
      </el-button>
    </div>

    <!-- 跟进记录对话框 -->
    <FollowUpDialog ref="FollowUpDialog" />
  </div>
</template>

<script>
import FollowUpDialog from './mixins/FollowUpDialog.vue'

export default {
  name: 'RegistrationAudit',
  components: {
    FollowUpDialog
  },
  props: {
    topicId: {
      type: [String, Number],
      default: ''
    }
  },
  data() {
    return {
      searchForm: {
        auditStatus: ''
      },
      tableData: [],
      loading: false,
      selectedRows: [], // 选中的行
      pagination: {
        current: 1,
        size: 10,
        total: 0
      }
    }
  },
  methods: {
    loadData() {
      // 加载报名审核数据
      console.log('加载报名审核数据', 'topicId:', this.topicId, 'current:', this.pagination.current, 'size:', this.pagination.size)
      this.loading = true

      // 模拟API调用 - 实际使用时替换为报名审核的API
      setTimeout(() => {
        this.tableData = [
          {
            id: 1,
            patientName: '周八',
            followUpPerson: '刘医生',
            registrationNo: 'REG006',
            age: 29,
            enrollmentTime: '2024-01-20',
            firstFollowUpTime: '2024-01-25',
            contactInfo: '13877778888',
            auditStatus: '1' // 待审核
          },
          {
            id: 2,
            patientName: '吴九',
            followUpPerson: '黄医生',
            registrationNo: 'REG007',
            age: 47,
            enrollmentTime: '2024-01-21',
            firstFollowUpTime: '2024-01-26',
            contactInfo: '13999990000',
            auditStatus: '3' // 审核不通过
          },
          {
            id: 3,
            patientName: '郑十',
            followUpPerson: '何医生',
            registrationNo: 'REG008',
            age: 35,
            enrollmentTime: '2024-01-22',
            firstFollowUpTime: '2024-01-27',
            contactInfo: '13111111111',
            auditStatus: '1' // 待审核
          },
          {
            id: 4,
            patientName: '钱十一',
            followUpPerson: '徐医生',
            registrationNo: 'REG009',
            age: 42,
            enrollmentTime: '2024-01-23',
            firstFollowUpTime: '2024-01-28',
            contactInfo: '13222222222',
            auditStatus: '2' // 审核通过
          }
        ]

        // 设置分页总数
        this.pagination.total = 18 // 模拟总数

        // 通知父组件更新badge数量（计算待审核数量）
        const pendingCount = this.tableData.filter(item => item.auditStatus === '1').length
        this.$emit('updateBadge', pendingCount)

        this.loading = false
      }, 500)
    },

    // 查询方法
    searchAudit() {
      console.log('查询报名审核:', this.searchForm)
      this.pagination.current = 1 // 查询时回到第一页
      this.loadData()
    },

    resetSearch() {
      this.searchForm.auditStatus = ''
      this.pagination.current = 1 // 重置时回到第一页
      this.loadData()
    },

    // 分页处理方法
    handleSizeChange(size) {
      this.pagination.size = size
      this.pagination.current = 1 // 改变每页条数时回到第一页
      this.loadData()
    },

    handleCurrentChange(current) {
      this.pagination.current = current
      this.loadData()
    },

    // 表格选择处理
    handleSelectionChange(selection) {
      this.selectedRows = selection
    },

    // 批量审核处理
    handleBatchAudit(action) {
      const actionText = action === 'approve' ? '审核通过' : '审核不通过'

      if (this.selectedRows.length === 0) {
        this.$message.warning('请先选择要审核的记录')
        return
      }

      this.$confirm(`确定要将选中的 ${this.selectedRows.length} 条报名记录${actionText}吗？`, '确认', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        // 这里调用对应的审核API
        this.performBatchAudit(action, this.selectedRows)
      }).catch(() => {
        // 取消操作
      })
    },

    performBatchAudit(action, selectedRows) {
      const ids = selectedRows.map(row => row.id)
      const newStatus = action === 'approve' ? '2' : '3'
      const actionText = action === 'approve' ? '审核通过' : '审核不通过'

      console.log(`执行报名${actionText}操作:`, {
        action,
        ids,
        newStatus
      })

      // 模拟API调用
      setTimeout(() => {
        // 更新本地数据状态
        this.tableData.forEach(item => {
          if (ids.includes(item.id)) {
            item.auditStatus = newStatus
          }
        })
        this.$refs.auditTable.clearSelection()
        this.selectedRows = []

        this.$message.success(`报名${actionText}操作成功`)

        // 重新计算并更新badge数量
        const pendingCount = this.tableData.filter(item => item.auditStatus === '1').length
        this.$emit('updateBadge', pendingCount)

        // 实际项目中，这里应该调用对应的API，例如:
        // return auditRegistrationBatch({ ids, status: newStatus })
      }, 1000)
    },

    // 操作按钮方法
    handleDetail(row) {
      console.log('查看详情:', row)
      // 待实现：打开详情弹窗或跳转详情页面
    },

    handleFollowUpRecord(row) {
      console.log('打开跟进记录:', row)
      // 打开跟进记录对话框
      this.$refs.FollowUpDialog.open(row)
    },

    // 获取审核状态显示文本
    getAuditStatusText(status) {
      const statusMap = {
        '1': '待审核',
        '2': '审核通过',
        '3': '审核不通过'
      }
      return statusMap[status] || '未知状态'
    },

    // 获取审核状态标签类型
    getAuditStatusType(status) {
      const typeMap = {
        '1': 'warning',
        '2': 'success',
        '3': 'danger'
      }
      return typeMap[status] || 'info'
    }
  },
  watch: {
    topicId: {
      handler(newVal) {
        if (newVal) {
          this.loadData()
        }
      },
      immediate: true
    }
  }
}
</script>

<style lang="scss" scoped>
.registration-audit {
  height: 100%;
  display: flex;
  flex-direction: column;

  .search-form {
    margin-bottom: 20px;
    flex-shrink: 0; // 防止搜索表单被压缩
  }

  .el-table {
    flex: 1;
    margin-bottom: 20px;
    overflow: auto; // 确保表格可以滚动
    min-height: 0; // 允许flex子项缩小
  }

  .pagination-wrapper {
    display: flex;
    justify-content: flex-end;
    margin: 20px 0;
    flex-shrink: 0; // 防止分页组件被压缩
  }

  .audit-actions {
    display: flex;
    justify-content: center; // 居中显示审核按钮
    align-items: center;
    gap: 10px;
    padding: 10px 0;
    border-top: 1px solid #EBEEF5;
    flex-shrink: 0; // 防止审核按钮区域被压缩
  }
}
</style>