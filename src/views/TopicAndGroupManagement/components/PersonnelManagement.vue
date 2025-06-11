<template>
  <div class="PersonnelManagement">
    <el-tabs v-model="activeTab" @tab-click="handleTabClick">
      <el-tab-pane name="approved">
        <span slot="label">
          <el-badge :value="badgeCounts.approved" :hidden="badgeCounts.approved === 0">
            审核通过患者列表
          </el-badge>
        </span>
        <div class="tab-content">
          <!-- 审核通过患者列表内容 -->
          <div class="approved-patients">
            <!-- 查询条件 -->
            <div class="search-form">
              <el-form :inline="true" :model="approvedSearchForm" class="demo-form-inline">
                <el-form-item label="跟进人:">
                  <el-input v-model="approvedSearchForm.followUpPerson" placeholder="请输入跟进人" clearable
                    style="width: 200px;" />
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" @click="searchApprovedPatients">查询</el-button>
                  <el-button @click="resetApprovedSearch">重置</el-button>
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
              <el-table :data="approvedTableData" border style="width: 100%" v-loading="approvedLoading">
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
                <el-pagination @size-change="handleApprovedSizeChange" @current-change="handleApprovedCurrentChange"
                  :current-page="approvedPagination.current" :page-sizes="[10, 20, 30, 40]"
                  :page-size="approvedPagination.size" layout="total, sizes, prev, pager, next, jumper"
                  :total="approvedPagination.total">
                </el-pagination>
              </div>
            </div>
          </div>
        </div>
      </el-tab-pane>

      <el-tab-pane name="survey">
        <span slot="label">
          <el-badge :value="badgeCounts.survey" :hidden="badgeCounts.survey === 0">
            调查问卷审核
          </el-badge>
        </span>
        <div class="tab-content">
          <!-- 调查问卷审核内容 -->
          <div class="survey-audit">
            <!-- 查询条件 -->
            <div class="search-form">
              <el-form :inline="true" :model="surveySearchForm" class="demo-form-inline">
                <el-form-item label="审核状态:">
                  <el-select v-model="surveySearchForm.auditStatus" placeholder="请选择审核状态" clearable
                    style="width: 150px;">
                    <el-option label="待审核" value="1" />
                    <el-option label="审核通过" value="2" />
                    <el-option label="审核不通过" value="3" />
                  </el-select>
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" @click="searchSurveyAudit">查询</el-button>
                  <el-button @click="resetSurveySearch">重置</el-button>
                </el-form-item>
              </el-form>
            </div>

            <!-- 调查问卷审核表格 -->
            <el-table :data="surveyTableData" border style="width: 100%" v-loading="surveyLoading"
              @selection-change="handleSurveySelectionChange" ref="surveyTable">
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
              <el-pagination @size-change="handleSurveySizeChange" @current-change="handleSurveyCurrentChange"
                :current-page="surveyPagination.current" :page-sizes="[10, 20, 30, 40]"
                :page-size="surveyPagination.size" layout="total, sizes, prev, pager, next, jumper"
                :total="surveyPagination.total">
              </el-pagination>
            </div>

            <!-- 审核操作按钮 -->
            <div class="audit-actions">
              <el-button type="success" @click="handleBatchAudit('survey', 'approve')"
                :disabled="surveySelectedRows.length === 0">
                审核通过
              </el-button>
              <el-button type="danger" @click="handleBatchAudit('survey', 'reject')"
                :disabled="surveySelectedRows.length === 0">
                审核不通过
              </el-button>
            </div>
          </div>
        </div>
      </el-tab-pane>

      <el-tab-pane name="registration">
        <span slot="label">
          <el-badge :value="badgeCounts.registration" :hidden="badgeCounts.registration === 0">
            报名审核
          </el-badge>
        </span>
        <div class="tab-content">
          <!-- 报名审核内容 -->
          <div class="registration-audit">
            <!-- 查询条件 -->
            <div class="search-form">
              <el-form :inline="true" :model="registrationSearchForm" class="demo-form-inline">
                <el-form-item label="审核状态:">
                  <el-select v-model="registrationSearchForm.auditStatus" placeholder="请选择审核状态" clearable
                    style="width: 150px;">
                    <el-option label="待审核" value="1" />
                    <el-option label="审核通过" value="2" />
                    <el-option label="审核不通过" value="3" />
                  </el-select>
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" @click="searchRegistrationAudit">查询</el-button>
                  <el-button @click="resetRegistrationSearch">重置</el-button>
                </el-form-item>
              </el-form>
            </div>

            <!-- 报名审核表格 -->
            <el-table :data="registrationTableData" border style="width: 100%" v-loading="registrationLoading"
              @selection-change="handleRegistrationSelectionChange" ref="registrationTable">
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
              <el-pagination @size-change="handleRegistrationSizeChange"
                @current-change="handleRegistrationCurrentChange" :current-page="registrationPagination.current"
                :page-sizes="[10, 20, 30, 40]" :page-size="registrationPagination.size"
                layout="total, sizes, prev, pager, next, jumper" :total="registrationPagination.total">
              </el-pagination>
            </div>

            <!-- 审核操作按钮 -->
            <div class="audit-actions">
              <el-button type="success" @click="handleBatchAudit('registration', 'approve')"
                :disabled="registrationSelectedRows.length === 0">
                审核通过
              </el-button>
              <el-button type="danger" @click="handleBatchAudit('registration', 'reject')"
                :disabled="registrationSelectedRows.length === 0">
                审核不通过
              </el-button>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  name: 'PersonnelManagement',
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
      activeTab: 'approved',
      // Badge数字显示
      badgeCounts: {
        approved: 0,    // 审核通过患者数量
        survey: 32,     // 待审核调查问卷数量
        registration: 8 // 待审核报名数量
      },

      // 审核通过患者列表相关数据
      approvedSearchForm: {
        followUpPerson: ''
      },
      approvedTableData: [],
      approvedLoading: false,

      // 招募统计信息
      recruitmentStats: {
        totalCount: 10,    // 招募人数
        recruitedCount: 8, // 已招募人数
        remainingCount: 2  // 待招募人数
      },

      // 审核通过患者分页数据
      approvedPagination: {
        current: 1,
        size: 10,
        total: 0
      },

      // 调查问卷审核相关数据
      surveySearchForm: {
        auditStatus: ''
      },
      surveyTableData: [],
      surveyLoading: false,
      surveySelectedRows: [], // 调查问卷审核选中的行
      surveyPagination: {
        current: 1,
        size: 10,
        total: 0
      },

      // 报名审核相关数据
      registrationSearchForm: {
        auditStatus: ''
      },
      registrationTableData: [],
      registrationLoading: false,
      registrationSelectedRows: [], // 报名审核选中的行
      registrationPagination: {
        current: 1,
        size: 10,
        total: 0
      }
    }
  },
  methods: {
    handleTabClick(tab) {
      console.log('切换到tab:', tab.name)
      // 根据不同的tab加载对应的数据
      this.loadTabData(tab.name)
    },

    loadTabData(tabName) {
      switch (tabName) {
        case 'approved':
          this.loadApprovedPatients()
          break
        case 'survey':
          this.loadSurveyAudit()
          break
        case 'registration':
          this.loadRegistrationAudit()
          break
      }
    },

    loadApprovedPatients() {
      // 加载审核通过患者列表数据
      console.log('加载审核通过患者列表数据', 'current:', this.approvedPagination.current, 'size:', this.approvedPagination.size)
      this.approvedLoading = true

      // 模拟API调用
      setTimeout(() => {
        this.approvedTableData = [
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
            id: 2,
            patientName: '李四',
            followUpPerson: '王医生',
            registrationNo: 'REG002',
            age: 52,
            enrollmentTime: '2024-01-16',
            firstFollowUpTime: '2024-01-21',
            contactInfo: '13987654321'
          }
        ]

        // 设置分页总数
        this.approvedPagination.total = 25 // 模拟总数

        // 更新统计信息
        this.recruitmentStats = {
          totalCount: this.topicInfo?.recruitCount || 10,
          recruitedCount: this.approvedTableData.length,
          remainingCount: (this.topicInfo?.recruitCount || 10) - this.approvedTableData.length
        }

        // 更新badge数量
        this.badgeCounts.approved = this.approvedTableData.length

        this.approvedLoading = false
      }, 500)
    },

    loadSurveyAudit() {
      // 加载调查问卷审核数据
      console.log('加载调查问卷审核数据', 'topicId:', this.topicId, 'current:', this.surveyPagination.current, 'size:', this.surveyPagination.size)
      this.surveyLoading = true

      // 模拟API调用 - 实际使用时替换为调查问卷审核的API
      setTimeout(() => {
        this.surveyTableData = [
          {
            id: 1,
            patientName: '王五',
            followUpPerson: '张医生',
            registrationNo: 'REG003',
            age: 38,
            enrollmentTime: '2024-01-17',
            firstFollowUpTime: '2024-01-22',
            contactInfo: '13511112222',
            auditStatus: '1' // 待审核
          },
          {
            id: 2,
            patientName: '赵六',
            followUpPerson: '李医生',
            registrationNo: 'REG004',
            age: 41,
            enrollmentTime: '2024-01-18',
            firstFollowUpTime: '2024-01-23',
            contactInfo: '13633334444',
            auditStatus: '2' // 审核通过
          },
          {
            id: 3,
            patientName: '孙七',
            followUpPerson: '陈医生',
            registrationNo: 'REG005',
            age: 33,
            enrollmentTime: '2024-01-19',
            firstFollowUpTime: '2024-01-24',
            contactInfo: '13755556666',
            auditStatus: '1' // 待审核
          }
        ]

        // 设置分页总数
        this.surveyPagination.total = 32 // 模拟总数

        this.surveyLoading = false
      }, 500)
    },

    loadRegistrationAudit() {
      // 加载报名审核数据
      console.log('加载报名审核数据', 'topicId:', this.topicId, 'current:', this.registrationPagination.current, 'size:', this.registrationPagination.size)
      this.registrationLoading = true

      // 模拟API调用 - 实际使用时替换为报名审核的API
      setTimeout(() => {
        this.registrationTableData = [
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
          }
        ]

        // 设置分页总数
        this.registrationPagination.total = 18 // 模拟总数

        this.registrationLoading = false
      }, 500)
    },

    // 查询方法
    searchApprovedPatients() {
      console.log('查询审核通过患者:', this.approvedSearchForm)
      this.approvedPagination.current = 1 // 查询时回到第一页
      this.loadApprovedPatients()
    },

    resetApprovedSearch() {
      this.approvedSearchForm.followUpPerson = ''
      this.approvedPagination.current = 1 // 重置时回到第一页
      this.loadApprovedPatients()
    },

    searchSurveyAudit() {
      console.log('查询调查问卷审核:', this.surveySearchForm)
      this.surveyPagination.current = 1 // 查询时回到第一页
      this.loadSurveyAudit()
    },

    resetSurveySearch() {
      this.surveySearchForm.auditStatus = ''
      this.surveyPagination.current = 1 // 重置时回到第一页
      this.loadSurveyAudit()
    },

    searchRegistrationAudit() {
      console.log('查询报名审核:', this.registrationSearchForm)
      this.registrationPagination.current = 1 // 查询时回到第一页
      this.loadRegistrationAudit()
    },

    resetRegistrationSearch() {
      this.registrationSearchForm.auditStatus = ''
      this.registrationPagination.current = 1 // 重置时回到第一页
      this.loadRegistrationAudit()
    },

    // 分页处理方法
    // 审核通过患者分页
    handleApprovedSizeChange(size) {
      this.approvedPagination.size = size
      this.approvedPagination.current = 1 // 改变每页条数时回到第一页
      this.loadApprovedPatients()
    },

    handleApprovedCurrentChange(current) {
      this.approvedPagination.current = current
      this.loadApprovedPatients()
    },

    // 调查问卷审核分页
    handleSurveySizeChange(size) {
      this.surveyPagination.size = size
      this.surveyPagination.current = 1 // 改变每页条数时回到第一页
      this.loadSurveyAudit()
    },

    handleSurveyCurrentChange(current) {
      this.surveyPagination.current = current
      this.loadSurveyAudit()
    },

    // 报名审核分页
    handleRegistrationSizeChange(size) {
      this.registrationPagination.size = size
      this.registrationPagination.current = 1 // 改变每页条数时回到第一页
      this.loadRegistrationAudit()
    },

    handleRegistrationCurrentChange(current) {
      this.registrationPagination.current = current
      this.loadRegistrationAudit()
    },

    // 表格选择处理
    handleSurveySelectionChange(selection) {
      this.surveySelectedRows = selection
    },

    handleRegistrationSelectionChange(selection) {
      this.registrationSelectedRows = selection
    },

    // 批量审核处理
    handleBatchAudit(type, action) {
      const selectedRows = type === 'survey' ? this.surveySelectedRows : this.registrationSelectedRows
      const actionText = action === 'approve' ? '审核通过' : '审核不通过'
      const typeText = type === 'survey' ? '调查问卷' : '报名'

      if (selectedRows.length === 0) {
        this.$message.warning('请先选择要审核的记录')
        return
      }

      this.$confirm(`确定要将选中的 ${selectedRows.length} 条${typeText}记录${actionText}吗？`, '确认', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        // 这里调用对应的审核API
        this.performBatchAudit(type, action, selectedRows)
      }).catch(() => {
        // 取消操作
      })
    },

    performBatchAudit(type, action, selectedRows) {
      const ids = selectedRows.map(row => row.id)
      const newStatus = action === 'approve' ? '2' : '3'
      const actionText = action === 'approve' ? '审核通过' : '审核不通过'
      const typeText = type === 'survey' ? '调查问卷' : '报名'

      console.log(`执行${typeText}${actionText}操作:`, {
        type,
        action,
        ids,
        newStatus
      })

      // 模拟API调用
      setTimeout(() => {
        // 更新本地数据状态
        if (type === 'survey') {
          this.surveyTableData.forEach(item => {
            if (ids.includes(item.id)) {
              item.auditStatus = newStatus
            }
          })
          this.$refs.surveyTable.clearSelection()
          this.surveySelectedRows = []
        } else {
          this.registrationTableData.forEach(item => {
            if (ids.includes(item.id)) {
              item.auditStatus = newStatus
            }
          })
          this.$refs.registrationTable.clearSelection()
          this.registrationSelectedRows = []
        }

        this.$message.success(`${typeText}${actionText}操作成功`)

        // 实际项目中，这里应该调用对应的API，例如:
        // if (type === 'survey') {
        //   return auditSurveyBatch({ ids, status: newStatus })
        // } else {
        //   return auditRegistrationBatch({ ids, status: newStatus })
        // }
      }, 1000)
    },

    // 操作按钮方法
    handleDetail(row) {
      console.log('查看详情:', row)
      // 待实现：打开详情弹窗或跳转详情页面
    },

    handleFollowUpRecord(row) {
      console.log('跟进记录:', row)
      // 待实现：打开跟进记录弹窗
    },

    handleChangeFollowUpPerson(row) {
      console.log('变更跟进人:', row)
      // 待实现：打开变更跟进人弹窗
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
    },

    // 获取所有tab的数据统计
    loadBadgeCounts() {
      if (!this.topicId) return

      // 模拟API调用获取各个tab的数据数量
      // 实际使用时替换为真实的API调用
      setTimeout(() => {
        this.badgeCounts = {
          approved: Math.floor(Math.random() * 50), // 模拟审核通过患者数量
          survey: Math.floor(Math.random() * 50),   // 模拟待审核调查问卷数量  
          registration: Math.floor(Math.random() * 20) // 模拟待审核报名数量
        }
      }, 500)
    },

    // 初始化组件数据
    init() {
      if (this.topicId) {
        this.loadBadgeCounts() // 加载badge数字
        this.loadTabData(this.activeTab)
      }
    }
  },
  watch: {
    topicId: {
      handler(newVal) {
        if (newVal) {
          this.init()
        }
      },
      immediate: true
    }
  },
  created() {
    // 组件创建时的初始化
  },
  mounted() {
    // 组件挂载完成后的操作
  }
}
</script>

<style lang="scss" scoped>
.PersonnelManagement {
  height: 100%;

  .el-tabs {
    height: 100%;

    :deep(.el-tabs__content) {
      height: calc(100% - 55px);
      overflow: hidden;
    }

    :deep(.el-tab-pane) {
      height: 100%;
    }

    // Badge样式优化
    :deep(.el-tabs__item) {
      position: relative;
      padding: 6px 20px 0px 0px;

      .el-badge {
        position: relative;
        display: inline-block;

        .el-badge__content {
          position: absolute;
          top: -8px;
          right: -10px;
          transform: none;
          min-width: 16px;
          height: 16px;
          line-height: 16px;
          padding: 0 4px;
          border-radius: 8px;
          font-size: 12px;
          z-index: 10;
        }
      }
    }
  }

  .tab-content {
    height: 100%;
    padding: 20px;
    box-sizing: border-box;
    overflow-y: auto;
  }

  // 审核通过患者列表样式
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
      }
    }
  }

  // 调查问卷审核和报名审核样式
  .survey-audit,
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

  // 分页样式
  .pagination-wrapper {
    display: flex;
    justify-content: flex-end;
    margin: 20px 0;
  }
}
</style>