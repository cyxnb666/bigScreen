<template>
  <el-dialog 
    title="跟进记录" 
    :visible.sync="dialogVisible" 
    width="1200px" 
    :close-on-click-modal="false"
    @close="handleClose">
    
    <div class="follow-up-dialog">
      <!-- 左侧：新增跟进内容 -->
      <div class="left-section">
        <div class="section-title">新增跟进内容</div>
        
        <el-form :model="followUpForm" ref="followUpForm" :rules="rules" label-width="80px">
          <el-form-item label="跟进人">
            <el-input v-model="followUpForm.followUpPerson" disabled></el-input>
          </el-form-item>
          
          <el-form-item label="跟进内容" prop="content">
            <el-input 
              type="textarea" 
              v-model="followUpForm.content"
              placeholder="请输入跟进内容"
              :autosize="{ minRows: 8, maxRows: 12 }"
              style="width: 100%">
            </el-input>
          </el-form-item>
        </el-form>
        
        <div class="submit-section">
          <el-button type="primary" @click="handleSubmit" :loading="submitting">提交</el-button>
        </div>
      </div>
      
      <!-- 右侧：历史跟进内容 -->
      <div class="right-section">
        <div class="section-title">历史跟进内容</div>
        
        <div class="history-list" v-loading="loading">
          <div 
            class="history-item" 
            v-for="(item, index) in historyList" 
            :key="index">
            <div class="history-header">
              <span class="follow-time">跟进时间：{{ item.followTime }}</span>
              <span class="follow-person">跟进人：{{ item.followPerson }}</span>
            </div>
            <div class="history-content">
              <span class="content-label">跟进内容：</span>
              <span class="content-text">{{ item.content }}</span>
            </div>
          </div>
          
          <!-- 暂无数据提示 -->
          <div v-if="!loading && historyList.length === 0" class="no-data">
            暂无跟进记录
          </div>
        </div>
      </div>
    </div>
  </el-dialog>
</template>

<script>
export default {
  name: 'FollowUpDialog',
  data() {
    return {
      dialogVisible: false,
      loading: false,
      submitting: false,
      
      // 患者信息
      patientInfo: null,
      
      // 新增跟进表单
      followUpForm: {
        followUpPerson: '',
        content: ''
      },
      
      // 表单验证规则
      rules: {
        content: [
          { required: true, message: '请输入跟进内容', trigger: 'blur' }
        ]
      },
      
      // 历史跟进记录列表
      historyList: []
    }
  },
  methods: {
    // 打开对话框
    open(patientData) {
      this.patientInfo = patientData
      this.dialogVisible = true
      
      // 设置当前登录用户为跟进人
      const userInfo = JSON.parse(sessionStorage.getItem('userInfo') || '{}')
      this.followUpForm.followUpPerson = userInfo.userName || '王丽'
      
      // 加载历史跟进记录
      this.loadHistoryList()
    },
    
    // 关闭对话框
    handleClose() {
      this.dialogVisible = false
      this.resetForm()
    },
    
    // 重置表单
    resetForm() {
      this.followUpForm = {
        followUpPerson: '',
        content: ''
      }
      this.$nextTick(() => {
        this.$refs.followUpForm && this.$refs.followUpForm.clearValidate()
      })
    },
    
    // 提交跟进记录
    handleSubmit() {
      this.$refs.followUpForm.validate(valid => {
        if (valid) {
          this.submitting = true
          
          // 构造提交数据
          const submitData = {
            patientId: this.patientInfo?.id,
            followUpPerson: this.followUpForm.followUpPerson,
            content: this.followUpForm.content,
            followTime: new Date().toLocaleString('zh-CN', {
              year: 'numeric',
              month: '2-digit',
              day: '2-digit',
              hour: '2-digit',
              minute: '2-digit',
              second: '2-digit'
            }).replace(/\//g, '-')
          }
          
          console.log('提交跟进记录:', submitData)
          
          // 模拟API调用
          setTimeout(() => {
            this.$message.success('跟进记录添加成功')
            
            // 添加到历史记录列表顶部
            this.historyList.unshift({
              followTime: submitData.followTime,
              followPerson: submitData.followUpPerson,
              content: submitData.content
            })
            
            // 重置表单
            this.followUpForm.content = ''
            this.$refs.followUpForm.clearValidate()
            
            this.submitting = false
            
            // 实际项目中这里应该调用真实的API
            // addFollowUpRecord(submitData)
            //   .then(() => {
            //     this.$message.success('跟进记录添加成功')
            //     this.loadHistoryList() // 重新加载历史记录
            //     this.followUpForm.content = ''
            //     this.$refs.followUpForm.clearValidate()
            //   })
            //   .catch(() => {
            //     this.$message.error('跟进记录添加失败')
            //   })
            //   .finally(() => {
            //     this.submitting = false
            //   })
          }, 1000)
        }
      })
    },
    
    // 加载历史跟进记录
    loadHistoryList() {
      if (!this.patientInfo) return
      
      this.loading = true
      
      console.log('加载跟进历史记录:', this.patientInfo)
      
      // 模拟API调用
      setTimeout(() => {
        // 模拟历史数据
        this.historyList = [
          {
            followTime: '2025-2-13 14:23:23',
            followPerson: '王丽',
            content: '主动联系患者，确认患者情况，符合课题。后续协助患者填写报告'
          },
          {
            followTime: '2025-2-12 13:22:32',
            followPerson: '王丽',
            content: '已经电话联系患者，患者目前在第一人民医院治疗，先不做审核，后续跟进进一步了解情况，根据患者报告决定'
          }
        ]
        
        this.loading = false
        
        // 实际项目中这里应该调用真实的API
        // getFollowUpHistory({ patientId: this.patientInfo.id })
        //   .then(res => {
        //     this.historyList = res || []
        //   })
        //   .catch(() => {
        //     this.$message.error('加载跟进记录失败')
        //     this.historyList = []
        //   })
        //   .finally(() => {
        //     this.loading = false
        //   })
      }, 500)
    }
  }
}
</script>

<style lang="scss" scoped>
.follow-up-dialog {
  display: flex;
  height: 600px;
  gap: 20px;
  
  .left-section,
  .right-section {
    flex: 1;
    display: flex;
    flex-direction: column;
  }
  
  .left-section {
    border-right: 1px solid #EBEEF5;
    padding-right: 20px;
  }
  
  .right-section {
    padding-left: 20px;
  }
  
  .section-title {
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 20px;
    color: #303133;
  }
  
  // 左侧样式
  .submit-section {
    display: flex;
    justify-content: center;
    margin-top: 20px;
    
    .el-button {
      width: 120px;
    }
  }
  
  // 右侧样式
  .history-list {
    flex: 1;
    overflow-y: auto;
    background-color: #F5F7FA;
    padding: 15px;
    border-radius: 4px;
    
    .history-item {
      background: white;
      border-radius: 4px;
      padding: 15px;
      margin-bottom: 15px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      
      &:last-child {
        margin-bottom: 0;
      }
      
      .history-header {
        display: flex;
        justify-content: space-between;
        margin-bottom: 10px;
        font-size: 13px;
        color: #909399;
        
        .follow-time,
        .follow-person {
          font-weight: 500;
        }
      }
      
      .history-content {
        line-height: 1.6;
        
        .content-label {
          color: #606266;
          font-weight: 500;
        }
        
        .content-text {
          color: #303133;
        }
      }
    }
    
    .no-data {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 200px;
      color: #909399;
      font-size: 14px;
    }
  }
}

// 表单样式调整
:deep(.el-form) {
  .el-form-item {
    margin-bottom: 20px;
    
    .el-form-item__label {
      font-weight: 500;
    }
  }
  
  .el-input.is-disabled .el-input__inner {
    background-color: #F5F7FA;
    border-color: #E4E7ED;
    color: #606266;
  }
}
</style>