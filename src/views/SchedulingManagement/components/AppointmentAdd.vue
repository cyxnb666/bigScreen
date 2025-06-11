<template>
  <el-dialog title="新增预约" :visible.sync="dialogVisible" width="600px" :close-on-click-modal="false"
    @close="handleClose">
    <el-form :model="formData" ref="formRef" :rules="rules" label-width="100px">
      <el-form-item label="预约课题" prop="topicId">
        <el-select v-model="formData.topicId" placeholder="请选择预约课题" style="width: 100%">
          <el-option v-for="item in topicOptions" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      
      <el-form-item label="患者姓名" prop="customerId">
        <el-select v-model="formData.customerId" placeholder="请选择患者" style="width: 100%" filterable>
          <el-option v-for="item in customerOptions" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      
      <el-form-item label="预约时间" prop="appointTime">
        <el-date-picker
          v-model="formData.appointTime"
          placeholder="请选择预约时间"
          style="width: 100%"
          value-format="yyyy-MM-dd"
          :picker-options="pickerOptions">
        </el-date-picker>
      </el-form-item>
    </el-form>
    
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="handleSubmit" :loading="loading">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script>
export default {
  name: 'AppointmentAdd',
  data() {
    return {
      dialogVisible: false,
      loading: false,
      formData: {
        topicId: '',
        customerId: '',
        appointTime: ''
      },
      rules: {
        topicId: [
          { required: true, message: '请选择预约课题', trigger: 'change' }
        ],
        customerId: [
          { required: true, message: '请选择患者', trigger: 'change' }
        ],
        appointTime: [
          { required: true, message: '请选择预约时间', trigger: 'change' }
        ]
      },
      // 模拟课题数据
      topicOptions: [
        { value: '1', label: '心血管疾病研究课题' },
        { value: '2', label: '糖尿病康复治疗课题' },
        { value: '3', label: '神经系统疾病研究课题' },
        { value: '4', label: '呼吸系统疾病治疗课题' },
        { value: '5', label: '消化系统疾病研究课题' }
      ],
      // 模拟患者数据
      customerOptions: [
        { value: '1', label: '张三 (13812345678)' },
        { value: '2', label: '李四 (13987654321)' },
        { value: '3', label: '王五 (13611112222)' },
        { value: '4', label: '赵六 (13733334444)' },
        { value: '5', label: '钱七 (13855556666)' },
        { value: '6', label: '孙八 (13977778888)' },
        { value: '7', label: '周九 (13099990000)' }
      ],
      // 时间选择器配置
      pickerOptions: {
        disabledDate(time) {
          // 禁用今天之前的日期
          return time.getTime() < Date.now() - 8.64e7
        },
        disabledTime(time) {
          const now = new Date()
          const selectedDate = new Date(time)
          
          // 如果选择的是今天，则禁用当前时间之前的时间
          if (selectedDate.toDateString() === now.toDateString()) {
            const currentHour = now.getHours()
            const currentMinute = now.getMinutes()
            
            return {
              disabledHours() {
                const hours = []
                for (let i = 0; i < currentHour; i++) {
                  hours.push(i)
                }
                return hours
              },
              disabledMinutes(hour) {
                if (hour === currentHour) {
                  const minutes = []
                  for (let i = 0; i <= currentMinute; i++) {
                    minutes.push(i)
                  }
                  return minutes
                }
                return []
              }
            }
          }
          return {}
        }
      }
    }
  },
  methods: {
    // 打开弹窗
    open() {
      this.dialogVisible = true
      this.resetForm()
    },
    
    // 关闭弹窗
    handleClose() {
      this.dialogVisible = false
      this.resetForm()
      this.$nextTick(() => {
        this.$refs.formRef && this.$refs.formRef.clearValidate()
      })
    },
    
    // 重置表单
    resetForm() {
      this.formData = {
        topicId: '',
        customerId: '',
        appointTime: ''
      }
    },
    
    // 提交表单
    handleSubmit() {
      this.$refs.formRef.validate(valid => {
        if (valid) {
          this.loading = true
          
          // 构造提交数据
          const submitData = {
            topicId: this.formData.topicId,
            customerId: this.formData.customerId,
            appointTime: this.formData.appointTime
          }
          
          // 模拟API调用
          // 实际使用时替换为真实的API调用
          setTimeout(() => {
            console.log('提交预约数据:', submitData)
            this.$message.success('预约添加成功')
            this.dialogVisible = false
            
            // 通知父组件刷新数据
            this.$emit('refresh')
            
            this.loading = false
          }, 1000)
          
          // 实际API调用示例：
          // addAppointment(submitData)
          //   .then(() => {
          //     this.$message.success('预约添加成功')
          //     this.dialogVisible = false
          //     this.$emit('refresh')
          //   })
          //   .catch(() => {
          //     this.$message.error('预约添加失败')
          //   })
          //   .finally(() => {
          //     this.loading = false
          //   })
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.el-form {
  .el-form-item {
    margin-bottom: 20px;
  }
}
</style>