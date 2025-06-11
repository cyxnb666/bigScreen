<template>
  <div class="approved-patients">
    <!-- 查询条件 -->
    <div class="search-form">
      <el-form :inline="true" :model="searchForm" class="demo-form-inline">
        <el-form-item label="跟进人:">
          <el-input v-model="searchForm.followUpPerson" placeholder="请输入跟进人" clearable
            style="width: 200px;" />
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
      <el-table :data="tableData" border style="width: 100%" v-loading="loading">
        <el-table-column prop="patientName" label="患者名称" align="center" />
        <el-table-column prop="followUpPerson" label="跟进人" align="center" />
        <el-table-column prop="registrationNo" label="登记号" align="center" />
        <el-table-column prop="age" label="年龄" align="center" />
        <el-table-column prop="enrollmentTime" label="入组时间" align="center" />
        <el-table-column prop="firstFollowUpTime" label="首次随访时间" align="center" />
        <el-table-column prop="contactInfo" label="联系方式" align="center" />
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
          :current-page="pagination.current" :page-sizes="[10, 20, 30, 40]"
          :page-size="pagination.size" layout="total, sizes, prev, pager, next, jumper"
          :total="pagination.total">
        </el-pagination>
      </div>
    </div>

    <!-- 跟进记录对话框 -->
    <FollowUpDialog ref="FollowUpDialog" />
  </div>
</template>

<script>
import FollowUpDialog from './mixins/FollowUpDialog.vue'

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
        followUpPerson: ''
      },
      tableData: [],
      loading: false,

      // 招募统计信息
      recruitmentStats: {
        totalCount: 10,    // 招募人数
        recruitedCount: 8, // 已招募人数
        remainingCount: 2  // 待招募人数
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
      // 加载审核通过患者列表数据
      console.log('加载审核通过患者列表数据', 'topicId:', this.topicId, 'current:', this.pagination.current, 'size:', this.pagination.size)
      this.loading = true

      // 模拟API调用
      setTimeout(() => {
        this.tableData = [
          {
            id: 1,
            patientName: '张三',
            followUpPerson: '李医生',
            registrationNo: 'REG001',
            age: 45,
            enrollmentTime: '2024-01-15',
            firstFollowUpTime: '2024-01-20',
            contactInfo: '13812345678'
          },
          {
            id: 2,
            patientName: '李四',
            followUpPerson: '王医生',
            registrationNo: 'REG002',
            age: 52,
            enrollmentTime: '2024-01-16',
            firstFollowUpTime: '2024-01-21',
            contactInfo: '13987654321'
          },
          {
            id: 3,
            patientName: '王五',
            followUpPerson: '张医生',
            registrationNo: 'REG003',
            age: 38,
            enrollmentTime: '2024-01-17',
            firstFollowUpTime: '2024-01-22',
            contactInfo: '13511112222'
          },
          {
            id: 4,
            patientName: '赵六',
            followUpPerson: '李医生',
            registrationNo: 'REG004',
            age: 41,
            enrollmentTime: '2024-01-18',
            firstFollowUpTime: '2024-01-23',
            contactInfo: '13633334444'
          },
          {
            id: 5,
            patientName: '孙七',
            followUpPerson: '陈医生',
            registrationNo: 'REG005',
            age: 33,
            enrollmentTime: '2024-01-19',
            firstFollowUpTime: '2024-01-24',
            contactInfo: '13755556666'
          },
          {
            id: 6,
            patientName: '周八',
            followUpPerson: '刘医生',
            registrationNo: 'REG006',
            age: 29,
            enrollmentTime: '2024-01-20',
            firstFollowUpTime: '2024-01-25',
            contactInfo: '13877778888'
          }
        ]

        // 设置分页总数
        this.pagination.total = 25 // 模拟总数

        // 更新统计信息
        this.recruitmentStats = {
          totalCount: this.topicInfo?.recruitCount || 10,
          recruitedCount: this.tableData.length,
          remainingCount: (this.topicInfo?.recruitCount || 10) - this.tableData.length
        }

        // 通知父组件更新badge数量
        this.$emit('updateBadge', this.tableData.length)

        this.loading = false
      }, 500)
    },

    // 查询方法
    searchPatients() {
      console.log('查询审核通过患者:', this.searchForm)
      this.pagination.current = 1 // 查询时回到第一页
      this.loadData()
    },

    resetSearch() {
      this.searchForm.followUpPerson = ''
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
      console.log('查看详情:', row)
      // 待实现：打开详情弹窗或跳转详情页面
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