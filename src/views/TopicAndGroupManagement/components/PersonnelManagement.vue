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
          <ApprovedPatients :topicId="topicId" :topicInfo="topicInfo" @updateBadge="updateApprovedBadge"
            @detail-click="handleDetailClick" />
        </div>
      </el-tab-pane>

      <el-tab-pane name="survey">
        <span slot="label">
          <el-badge :value="badgeCounts.survey" :hidden="badgeCounts.survey === 0">
            调查问卷审核
          </el-badge>
        </span>
        <div class="tab-content">
          <SurveyAudit ref="surveyAudit" :topicId="topicId" @updateBadge="updateSurveyBadge"
            @detail-click="handleDetailClick" />
        </div>
      </el-tab-pane>

      <el-tab-pane name="registration">
        <span slot="label">
          <el-badge :value="badgeCounts.registration" :hidden="badgeCounts.registration === 0">
            报名审核
          </el-badge>
        </span>
        <div class="tab-content">
          <RegistrationAudit ref="registrationAudit" :topicId="topicId" @updateBadge="updateRegistrationBadge"
            @detail-click="handleDetailClick" />
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import ApprovedPatients from './PersonnelComponents/ApprovedPatients.vue'
import SurveyAudit from './PersonnelComponents/SurveyAudit.vue'
import RegistrationAudit from './PersonnelComponents/RegistrationAudit.vue'

export default {
  name: 'PersonnelManagement',
  components: {
    ApprovedPatients,
    SurveyAudit,
    RegistrationAudit
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
      activeTab: 'approved',
      // Badge数字显示
      badgeCounts: {
        approved: 0,    // 审核通过患者数量
        survey: 0,      // 待审核调查问卷数量
        registration: 0 // 待审核报名数量
      }
    }
  },
  methods: {
    handleTabClick(tab) {
      console.log('切换到tab:', tab.name)
      // 主动触发对应tab的数据加载
      if (tab.name === 'survey') {
        this.$refs.surveyAudit && this.$refs.surveyAudit.loadData()
      } else if (tab.name === 'registration') {
        this.$refs.registrationAudit && this.$refs.registrationAudit.loadData()
      }
    },

    // 处理详情点击事件，向上传递给父组件
    handleDetailClick(row) {
      this.$emit('detail-click', row)
    },

    // 更新badge数量的方法
    updateApprovedBadge(count) {
      this.badgeCounts.approved = count
    },

    updateSurveyBadge(count) {
      this.badgeCounts.survey = count
    },

    updateRegistrationBadge(count) {
      this.badgeCounts.registration = count
    },

    // 初始化组件数据
    init() {
      // 清空badge数字，等待子组件加载完成后更新
      this.badgeCounts = {
        approved: 0,
        survey: 0,
        registration: 0
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
}
</style>