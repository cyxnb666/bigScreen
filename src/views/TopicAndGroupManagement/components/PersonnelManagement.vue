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
          <ApprovedPatients 
            :topicId="topicId" 
            :topicInfo="topicInfo"
            @updateBadge="updateApprovedBadge"
          />
        </div>
      </el-tab-pane>

      <el-tab-pane name="survey">
        <span slot="label">
          <el-badge :value="badgeCounts.survey" :hidden="badgeCounts.survey === 0">
            调查问卷审核
          </el-badge>
        </span>
        <div class="tab-content">
          <SurveyAudit 
            :topicId="topicId" 
            @updateBadge="updateSurveyBadge"
          />
        </div>
      </el-tab-pane>

      <el-tab-pane name="registration">
        <span slot="label">
          <el-badge :value="badgeCounts.registration" :hidden="badgeCounts.registration === 0">
            报名审核
          </el-badge>
        </span>
        <div class="tab-content">
          <RegistrationAudit 
            :topicId="topicId" 
            @updateBadge="updateRegistrationBadge"
          />
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
        survey: 32,     // 待审核调查问卷数量
        registration: 8 // 待审核报名数量
      }
    }
  },
  methods: {
    handleTabClick(tab) {
      console.log('切换到tab:', tab.name)
      // tab切换时子组件会自动加载数据
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