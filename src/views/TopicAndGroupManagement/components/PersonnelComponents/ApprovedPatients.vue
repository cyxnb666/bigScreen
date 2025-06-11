<template>
  <div class="approved-patients">
    <!-- 查询条件 -->
    <div class="search-form">
      <el-form :inline="true" :model="searchForm" class="demo-form-inline">
        <el-form-item label="跟进人:">
          <el-input v-model="searchForm.superintendentName" placeholder="请输入跟进人" clearable style="width: 200px;" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchPatients">查询</el-button>
          <el-button @click="resetSearch">重置</el-button>
        </el-form-item>
      </el-form>
    </div>

    <!-- 统计信息和患者列表表格 -->
    <div class="table-section">
      <div class="table-header">
        <div class="statistics-info">
          审核通过患者列表(招募人数: {{ recruitmentStats.totalCount }}人; 已招募: {{ recruitmentStats.recruitedCount }}人; 待招募:
          {{ recruitmentStats.remainingCount }}人)
        </div>
      </div>
      <el-table :data="tableData" border style="width: 100%" height="250" v-loading="loading">
        <el-table-column prop="customerName" label="患者名称" align="center" />
        <el-table-column prop="superintendentName" label="跟进人" align="center" />
        <el-table-column prop="customerNo" label="登记号" align="center" />
        <el-table-column prop="age" label="年龄" align="center" />
        <el-table-column prop="signUpTime" label="入组时间" align="center" />
        <el-table-column prop="firstVisitTime" label="首次随访时间" align="center" />
        <el-table-column prop="phone" label="联系方式" align="center" />
        <el-table-column label="操作" align="center" width="210">
          <template slot-scope="scope">
            <el-button type="text" @click="handleDetail(scope.row)">详情</el-button>
            <el-button type="text" @click="handleFollowUpRecord(scope.row)">跟进记录</el-button>
            <el-button type="text" @click="handleChangeFollowUpPerson(scope.row)">变更跟进人</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页组件 -->
      <div class="pagination-wrapper">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
          :current-page="pagination.current" :page-sizes="[10, 20, 30, 40]" :page-size="pagination.size"
          layout="total, sizes, prev, pager, next, jumper" :total="pagination.total">
        </el-pagination>
      </div>
    </div>

    <!-- 跟进记录对话框 -->
    <FollowUpDialog ref="FollowUpDialog" />
  </div>
</template>

<script>
import FollowUpDialog from './mixins/FollowUpDialog.vue'
import { listTopicRecruitment } from '../../api'

export default {
  name: 'ApprovedPatients',
  components: {
    FollowUpDialog
  },
  props: {
    topicId: {
      type: [String, Number],
      default: ''
    },
    topicInfo: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      searchForm: {
        superintendentName: ''
      },
      tableData: [],
      loading: false,

      // 招募统计信息
      recruitmentStats: {
        totalCount: 0,    // 招募人数
        recruitedCount: 0, // 已招募人数
        remainingCount: 0  // 待招募人数
      },

      // 分页数据
      pagination: {
        current: 1,
        size: 10,
        total: 0
      }
    }
  },
  methods: {
    loadData() {
      if (!this.topicId) return

      this.loading = true

      const params = {
        condition: {
          auditStatus: "6", // 审核通过
          queryType: "1",   // 审核通过患者列表
          superintendentName: this.searchForm.superintendentName,
          topicId: this.topicId
        },
        pageIndex: this.pagination.current,
        pageSize: this.pagination.size
      }

      listTopicRecruitment(params)
        .then(res => {
          this.tableData = res.items || []
          this.pagination.total = res.totalSize || 0

          // 更新统计信息 - 使用总数量而不是当前页数量
          this.recruitmentStats = {
            totalCount: this.topicInfo?.recruitCount || 0,
            recruitedCount: this.pagination.total, // 使用总数量
            remainingCount: Math.max(0, (this.topicInfo?.recruitCount || 0) - this.pagination.total)
          }

          // 通知父组件更新badge数量 - 使用extendData.auditAmount
          this.$emit('updateBadge', res.extendData?.auditAmount || 0)
        })
        .catch(() => {
          this.$message.error('获取审核通过患者列表失败')
          this.tableData = []
          this.pagination.total = 0
        })
        .finally(() => {
          this.loading = false
        })
    },

    // 查询方法
    searchPatients() {
      this.pagination.current = 1 // 查询时回到第一页
      this.loadData()
    },

    resetSearch() {
      this.searchForm.superintendentName = ''
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

    // 操作按钮方法
    handleDetail(row) {
      // 向父组件传递详情点击事件
      this.$emit('detail-click', row)
    },

    handleFollowUpRecord(row) {
      console.log('打开跟进记录:', row)
      // 打开跟进记录对话框
      this.$refs.FollowUpDialog.open(row)
    },

    handleChangeFollowUpPerson(row) {
      console.log('变更跟进人:', row)
      // 待实现：打开变更跟进人弹窗
    }
  },
  watch: {
    topicId: {
      handler(newVal, oldVal) {
        // 转换为字符串进行比较，避免类型差异导致的重复调用
        const newValStr = String(newVal);
        const oldValStr = String(oldVal);

        // 只有当 topicId 真正变化且有效时才加载数据
        if (newVal && newValStr !== oldValStr && newValStr !== 'undefined') {
          this.pagination.current = 1 // 切换课题时回到第一页
          this.searchForm.superintendentName = '' // 清空搜索条件
          this.loadData()
        }
      },
      immediate: true
    }
  }
}
</script>

<style lang="scss" scoped>
.approved-patients {
  height: 100%;
  display: flex;
  flex-direction: column;

  .search-form {
    margin-bottom: 20px;
    flex-shrink: 0; // 防止搜索表单被压缩
  }

  .table-section {
    flex: 1;
    display: flex;
    flex-direction: column;
    min-height: 0; // 允许flex子项缩小

    .table-header {
      display: flex;
      justify-content: flex-end;
      margin-bottom: 10px;
      flex-shrink: 0; // 防止统计信息被压缩

      .statistics-info {
        font-size: 14px;
        color: #606266;
        font-weight: 500;
      }
    }

    .el-table {
      flex: 1;
      margin-bottom: 20px;
      overflow: auto; // 确保表格可以滚动
      min-height: 0; // 允许flex子项缩小
    }

    .pagination-wrapper {
      flex-shrink: 0; // 防止分页组件被压缩
      display: flex;
      justify-content: flex-end;
      margin: 20px 0;
    }
  }
}
</style>