我是一名前端开发工程师，需要根据后端的接口写一个界面 

我已经写好的web端的前端代码，需要基于rouyi-app-vue3进行二开，增加界面，现在需要增加如下功能

基础房贷测算功能

- 支持贷款类型选择：商贷/公积金/组合贷
- 支持还款方式：等额本息/等额本金
- 输入参数：贷款总额、LPR+基点年利率、贷款年限
- 输出汇总数据：还款总额、总利息、首月还款额
- 输出完整还款计划表：期数、还款日、当月利息、当月本金、剩余本金

其中web的代码为：

```
<template>
  <div class="app-container">

    <!-- 1. 房贷还款计算模块 -->
    <div v-if="mainTab === 'repay-calc'">
      <!-- 计算类型选择 -->
      <div class="calc-type-selector mb-4">
        <span class="me-2 fw-medium">计算类型：</span>
        <el-select
            v-model="calcType"
            style="width: 200px;"
            @change="handleCalcTypeChange">
          <el-option label="普通计算" value="normal" />
          <el-option label="提前还款计算" value="prepayment" />
        </el-select>
      </div>
      <!-- 还款方式 Tab -->
      <el-tabs
          v-model="activeTab"
          type="card"
          class="mb-5"
          @tab-change="handleTabChange"
      >
        <el-tab-pane label="等额本金" name="equal-principal">
          <CalculatorForm
              @calculate="handleCalculate"
              @reset="handleReset"
              :visible-prepayment="showPrepaymentForm"
          />
        </el-tab-pane>
        <el-tab-pane label="等额本息" name="equal-interest">
          <CalculatorForm
              @calculate="handleCalculate"
              @reset="handleReset"
              :visible-prepayment="showPrepaymentForm"
          />
        </el-tab-pane>
      </el-tabs>

      <!-- 计算结果展示 -->
      <CalculatorResult
          :summary="summary"
          :monthlyDetails="monthlyDetails"
          :yearlySummaries="yearlySummaries"
          :loading="loading"
          :showResult="showResult"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
// 导入RuoYi内置的request工具
import request from '@/utils/request'
// 组件导入
import CalculatorForm from '@/components/calculate/CalculatorForm.vue'
import CalculatorResult from '@/components/calculate/CalculatorResult.vue'
import {getEqualPrincipal, getEqualInterest} from '@/api/calculate/getRepayments/api.js'

// ========== 核心变量定义 ==========
const mainTab = ref('repay-calc')
const calcType = ref('normal')
const activeTab = ref('equal-principal')
const showPrepaymentForm = computed(() => calcType.value === 'prepayment')

// 计算结果存储
const summary = reactive({
  loanTotal: '-',
  years: '-',
  totalMonths: '-',
  totalAllPrincipal: '-',
  totalAllInterest: '-',
  totalAllRepay: '-'
})
const monthlyDetails = ref([])
const yearlySummaries = ref([])
const loading = ref(false)
const showResult = ref(false)

// ========== 事件处理函数 ==========
const handleMainTabChange = (tabName) => {
  if (tabName === 'repay-calc') {
    handleReset()
  }
}

const handleCalcTypeChange = () => {
  handleReset()
}

const handleTabChange = () => {
  handleReset()
}

const handleReset = () => {
  showResult.value = false
  Object.assign(summary, {
    loanTotal: '-',
    years: '-',
    totalMonths: '-',
    totalAllPrincipal: '-',
    totalAllInterest: '-',
    totalAllRepay: '-'
  })
  monthlyDetails.value = []
  yearlySummaries.value = []
}

/**
 * 处理计算请求（修复参数校验逻辑）
 * @param {Object} params 从表单传递的完整参数
 */
const handleCalculate = async (params) => {
  // 1. 空参数校验
  if (!params) {
    ElMessage.warning('请填写完整的贷款信息！')
    return
  }

  // 2. 根据贷款类型（纯商贷/纯公积金/组合贷）校验核心参数
  let hasValidLoan = false
  let totalLoan = 0
  let loanYears = 0

  // 纯商贷校验
  if (params.loanType === 'single') {
    if (!params.businessLoanTotal || !params.businessYears) {
      ElMessage.warning('请填写完整的商贷信息（总额+年限）！')
      return
    }
    totalLoan = params.businessLoanTotal
    loanYears = params.businessYears
    hasValidLoan = true
  }
  // 纯公积金贷校验
  else if (params.loanType === 'fund') {
    if (!params.fundLoanTotal || !params.fundYears) {
      ElMessage.warning('请填写完整的公积金贷信息（总额+年限）！')
      return
    }
    totalLoan = params.fundLoanTotal
    loanYears = params.fundYears
    hasValidLoan = true
  }
  // 组合贷校验
  else if (params.loanType === 'combination') {
    if (!params.businessLoanTotal || !params.businessYears || !params.fundLoanTotal || !params.fundYears) {
      ElMessage.warning('请填写完整的组合贷信息（商贷+公积金贷的总额+年限）！')
      return
    }
    totalLoan = params.businessLoanTotal + params.fundLoanTotal
    // 组合贷取商贷年限（也可根据业务逻辑调整为取最大值/最小值）
    loanYears = params.businessYears
    hasValidLoan = true
  }

  // 3. 最终校验兜底
  if (!hasValidLoan || totalLoan <= 0 || loanYears <= 0) {
    ElMessage.warning('请填写有效的贷款金额和年限（金额>0，年限>0）！')
    return
  }

  loading.value = true
  try {
    // 根据还款方式调用对应API
    let res = null
    if (activeTab.value === 'equal-principal') {
      res = await getEqualPrincipal(params)
    } else {
      res = await getEqualInterest(params)
    }
    debugger;
    // 接口返回处理
    if (res) {
      loading.value = false
      Object.assign(summary, res)
      monthlyDetails.value = res.monthlyDetails || []
      yearlySummaries.value = res.fundYearSummaries || []
      showResult.value = true
      ElMessage.success('计算成功！')
    } else {
      loading.value = false
      console.error('计算失败：', errMsg)
    }
  } catch (error) {
    console.error('房贷计算请求失败：', error)
    ElMessage.error('计算请求失败，请检查网络或联系管理员！')
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  console.log('房贷计算页面初始化完成')
})
</script>

<style scoped>
/* 适配RuoYi默认的app-container样式 */
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

/* 计算类型选择器样式 */
.calc-type-selector {
  display: flex;
  align-items: center;
}

/* 样式优化：使用RuoYi的margin类名，统一风格 */
.mb-4 {
  margin-bottom: 16px !important;
}

.mb-5 {
  margin-bottom: 20px !important;
}

.me-2 {
  margin-right: 8px !important;
}

.fw-medium {
  font-weight: 500 !important;
}

/* Tab样式优化（适配RuoYi后台风格） */
:deep(.el-tabs--card) {
  --el-tabs-card-border-color: var(--el-border-color-lighter);
}

:deep(.el-tabs--card .el-tabs__header) {
  margin-bottom: 20px;
}

:deep(.el-tabs--card .el-tabs__item) {
  padding: 0 20px;
  margin-right: 8px;
}

/* 响应式适配 */
@media (max-width: 768px) {
  :deep(.el-tabs--card .el-tabs__nav) {
    flex-wrap: wrap;
  }

  :deep(.el-tabs--card .el-tabs__item) {
    margin-bottom: 8px;
    flex: 1 1 auto;
    text-align: center;
  }

  .app-container {
    padding: 10px;
  }
}
</style>

```

CalculatorForm.vue

```
<template>
  <el-card shadow="hover" header="房贷还款计算器" class="calc-card">
    <!-- 表单容器：绑定校验规则 + 统一尺寸 -->
    <el-form
        :model="form"
        :rules="formRules"
        ref="formRef"
        size="default"
    >
      <!-- 贷款类型选择 -->
      <el-form-item label="贷款类型" prop="loanType">
        <el-radio-group v-model="form.loanType">
          <el-radio label="combination">组合贷（商贷+公积金）</el-radio>
          <el-radio label="single">纯商贷</el-radio>
          <el-radio label="fund">纯公积金贷</el-radio>
        </el-radio-group>
      </el-form-item>

      <!-- 商业贷款参数：修复v-model绑定 -->
      <el-collapse v-model="activeBusinessPanel" :disabled="form.loanType === 'fund'">
        <el-collapse-item title="商业贷款参数" name="business">
          <el-form-item label="商贷总额（元）" prop="businessLoanTotal" >
            <el-input
                v-model.number="form.businessLoanTotal"
                type="number"
                step="0.01"
                placeholder="请输入商贷总额，如 1000000"
                clearable
            />
          </el-form-item>
          <el-form-item label="商贷年利率（%）" prop="businessAnnualRate">
            <el-input
                v-model.number="form.businessAnnualRate"
                type="number"
                step="0.01"
                placeholder="请输入商贷年利率，如 4.9"
                clearable
            />
          </el-form-item>
          <el-form-item label="商贷还款年限" prop="businessYears">
            <el-input
                v-model.number="form.businessYears"
                type="number"
                min="1"
                placeholder="请输入商贷还款年限，如 30"
                clearable
            />
          </el-form-item>
        </el-collapse-item>
      </el-collapse>

      <!-- 公积金贷款参数：修复v-model绑定 -->
      <el-collapse v-model="activeFundPanel" :disabled="form.loanType === 'single'">
        <el-collapse-item title="公积金贷款参数" name="fund">
          <el-form-item label="公积金贷总额（元）" prop="fundLoanTotal">
            <el-input
                v-model.number="form.fundLoanTotal"
                type="number"
                step="0.01"
                placeholder="请输入公积金贷总额，如 500000"
                clearable
            />
          </el-form-item>
          <el-form-item label="公积金贷年利率（%）" prop="fundAnnualRate">
            <el-input
                v-model.number="form.fundAnnualRate"
                type="number"
                step="0.01"
                placeholder="请输入公积金贷年利率，如 3.1"
                clearable
            />
          </el-form-item>
          <el-form-item label="公积金贷还款年限" prop="fundYears">
            <el-input
                v-model.number="form.fundYears"
                type="number"
                min="1"
                placeholder="请输入公积金贷还款年限，如 30"
                clearable
            />
          </el-form-item>
        </el-collapse-item>
      </el-collapse>

      <!-- 公共参数：保留本金 -->
      <el-form-item label="保留本金（元）" prop="reservedPrincipal">
        <el-input
            v-model.number="form.reservedPrincipal"
            type="number"
            step="0.01"
            min="0"
            placeholder="请输入保留不还的本金金额，如 10000（填 0 则不保留）"
            clearable
        />
      </el-form-item>

      <!-- 提前还款子组件 -->
      <PrepaymentSection
          v-if="visiblePrepayment"
          v-model:prepayments="form.prepayments"
          v-model:periodicRepayList="form.periodicRepayList"
          @reset="handlePrepaymentReset"
          ref="prepaymentSectionRef"
      />

      <!-- 操作按钮区域 -->
      <el-form-item class="form-actions">
        <el-button type="primary" @click="handleCalculate">
          计算还款明细
        </el-button>
        <el-button @click="handleReset">
          重置
        </el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>

<script setup>
import { reactive, ref, defineEmits, defineProps, watch } from 'vue'
import { ElMessage } from 'element-plus'
// 请确保 PrepaymentSection 组件路径正确
import PrepaymentSection from './PrepaymentSection.vue'

// 新增：折叠面板激活状态（修复v-model语法错误）
const activeBusinessPanel = ref(['business'])
const activeFundPanel = ref(['fund'])

// Props 定义
const props = defineProps({
  visiblePrepayment: {
    type: Boolean,
    default: true
  }
})

// 事件定义
const emit = defineEmits(['calculate', 'reset'])

// 表单 Ref + 校验规则
const formRef = ref()
const prepaymentSectionRef = ref()

// 表单校验规则
const formRules = reactive({
  loanType: [
    { required: true, message: '请选择贷款类型', trigger: 'change' }
  ],
  // 商贷校验规则（动态校验）
  businessLoanTotal: [
    {
      required: true,
      message: '请输入商贷总额',
      trigger: 'blur',
      validator: (rule, value) => {
        return form.loanType !== 'single' && form.loanType !== 'combination' || (value && value > 0)
      }
    },
    { type: 'number', min: 0.01, message: '商贷总额必须大于 0', trigger: 'blur' }
  ],
  businessAnnualRate: [
    {
      required: true,
      message: '请输入商贷年利率',
      trigger: 'blur',
      validator: (rule, value) => {
        return form.loanType !== 'single' && form.loanType !== 'combination' || (value && value > 0)
      }
    },
    { type: 'number', min: 0.01, message: '商贷年利率必须大于 0', trigger: 'blur' }
  ],
  businessYears: [
    {
      required: true,
      message: '请输入商贷还款年限',
      trigger: 'blur',
      validator: (rule, value) => {
        return form.loanType !== 'single' && form.loanType !== 'combination' || (value && value >= 1)
      }
    },
    { type: 'number', min: 1, message: '商贷还款年限必须大于等于 1', trigger: 'blur' }
  ],
  // 公积金贷校验规则（动态校验）
  fundLoanTotal: [
    {
      required: true,
      message: '请输入公积金贷总额',
      trigger: 'blur',
      validator: (rule, value) => {
        return form.loanType !== 'fund' && form.loanType !== 'combination' || (value && value > 0)
      }
    },
    { type: 'number', min: 0.01, message: '公积金贷总额必须大于 0', trigger: 'blur' }
  ],
  fundAnnualRate: [
    {
      required: true,
      message: '请输入公积金贷年利率',
      trigger: 'blur',
      validator: (rule, value) => {
        return form.loanType !== 'fund' && form.loanType !== 'combination' || (value && value > 0)
      }
    },
    { type: 'number', min: 0.01, message: '公积金贷年利率必须大于 0', trigger: 'blur' }
  ],
  fundYears: [
    {
      required: true,
      message: '请输入公积金贷还款年限',
      trigger: 'blur',
      validator: (rule, value) => {
        return form.loanType !== 'fund' && form.loanType !== 'combination' || (value && value >= 1)
      }
    },
    { type: 'number', min: 1, message: '公积金贷还款年限必须大于等于 1', trigger: 'blur' }
  ],
  reservedPrincipal: [
    { type: 'number', min: 0, message: '保留本金必须大于等于 0', trigger: 'blur' }
  ]
})

// 表单数据
const form = reactive({
  loanType: 'combination', // 贷款类型：single(纯商贷)、fund(纯公积金)、combination(组合贷)
  // 商贷参数
  businessLoanTotal: '',
  businessAnnualRate: '',
  businessYears: '',
  // 公积金贷参数
  fundLoanTotal: '',
  fundAnnualRate: '',
  fundYears: '',
  // 公共参数
  reservedPrincipal: 0,
  prepayments: [],
  periodicRepayList: []
})

// 监听贷款类型变化：自动切换面板状态 + 清空无关参数
watch(() => form.loanType, (newType) => {
  if (newType === 'single') {
    // 纯商贷：展开商贷面板，收起公积金面板
    activeBusinessPanel.value = ['business']
    activeFundPanel.value = []
    // 清空公积金参数
    form.fundLoanTotal = ''
    form.fundAnnualRate = ''
    form.fundYears = ''
  } else if (newType === 'fund') {
    // 纯公积金：展开公积金面板，收起商贷面板
    activeFundPanel.value = ['fund']
    activeBusinessPanel.value = []
    // 清空商贷参数
    form.businessLoanTotal = ''
    form.businessAnnualRate = ''
    form.businessYears = ''
  } else if (newType === 'combination') {
    // 组合贷：展开两个面板
    activeBusinessPanel.value = ['business']
    activeFundPanel.value = ['fund']
  }
})

// 重置表单
const handleReset = () => {
  if (formRef.value) {
    formRef.value.resetFields()
  }

  // 重置提前还款子组件
  prepaymentSectionRef.value?.resetPrepayment()

  // 重置面板状态
  activeBusinessPanel.value = ['business']
  activeFundPanel.value = ['fund']

  // 重置默认值
  form.loanType = 'single'
  form.reservedPrincipal = 0
  form.prepayments = []
  form.periodicRepayList = []

  emit('reset')
  ElMessage.success('表单已重置')
}

// 提前还款子组件重置回调
const handlePrepaymentReset = () => {
  form.prepayments = []
  form.periodicRepayList = []
}

// 计算逻辑
const handleCalculate = async () => {
  // 表单校验
  try {
    await formRef.value.validate()
  } catch (error) {
    ElMessage.error('表单校验失败，请检查输入内容')
    return
  }

  // 组装请求参数
  const params = {
    loanType: form.loanType,
    // 商贷参数
    businessLoanTotal: Number(form.businessLoanTotal) || 0,
    businessAnnualRate: Number(form.businessAnnualRate) || 0,
    businessYears: Number(form.businessYears) || 0,
    // 公积金贷参数
    fundLoanTotal: Number(form.fundLoanTotal) || 0,
    fundAnnualRate: Number(form.fundAnnualRate) || 0,
    fundYears: Number(form.fundYears) || 0,
    // 公共参数
    reservedPrincipal: Number(form.reservedPrincipal),
    prepayments: form.prepayments
        .filter(item => item.month && item.amount)
        .map(item => ({
          month: Number(item.month),
          amount: Number(item.amount),
          year: 0
        })),
    periodicRepayList: form.periodicRepayList
  }

  // 保留本金校验（组合贷时校验总额）
  const totalLoan = params.businessLoanTotal + params.fundLoanTotal
  if (params.reservedPrincipal >= totalLoan && totalLoan > 0) {
    ElMessage.error(`保留本金需小于贷款总额（${totalLoan} 元）`)
    return
  }

  // 通知父组件计算（父组件需调用后端 /api/repay/combination-principal 接口）
  emit('calculate', params)
  ElMessage.info('正在计算，请稍候...')
}
</script>

<style scoped>
/* 核心容器：基础样式 + 响应式最大宽度 */
.calc-card {
  max-width: 1000px;
  min-width: 320px;
  margin: 0 auto;
  padding: 0 16px;
  margin-top: 20px;
  margin-bottom: 40px;
}

/* Element Plus 卡片内边距：响应式调整 */
:deep(.el-card__body) {
  padding: 10px;
  /* 小屏（手机）时减小内边距，节省空间 */
  @media (max-width: 768px) {
    padding: 16px;
  }
}

/* 表单项间距：响应式调整 */
:deep(.el-form-item) {
  margin-bottom: 40px;

  /* 小屏时减小间距，节省垂直空间 */
  @media (max-width: 768px) {
    margin-bottom: 16px;
  }
}

/* 折叠面板样式优化 */
:deep(.el-collapse-item__header) {
  font-weight: 500;
  padding: 12px 16px;
  background-color: #f8f9fa;
}

:deep(.el-collapse-item__content) {
  padding: 16px;
  border-top: 1px solid #ebeef5;
}


/* 按钮区域：核心响应式布局 */
.form-actions {
  display: flex;
  justify-content: flex-start;
  padding-left: 120px;
  margin-top: 8px;
  gap: 12px;

  /* 断点1：平板/小屏电脑（≤992px） */
  @media (max-width: 992px) {
    padding-left: 80px;
  }

  /* 断点2：手机（≤768px）- 核心适配 */
  @media (max-width: 768px) {
    /* 按钮从横向排列改为纵向排列 */
    flex-direction: column;
    /* 取消左内边距，避免内容溢出 */
    padding-left: 0;
    /* 按钮宽度占满容器，更易点击 */
    gap: 8px;
  }

  /* 断点3：超小屏手机（≤480px） */
  @media (max-width: 480px) {
    /* 进一步减小间距 */
    gap: 6px;
    margin-top: 4px;
  }
}

/* 表单标签宽度：响应式调整（解决小屏标签换行问题） */
:deep(.el-form-item__label) {
  /* 小屏时减小标签宽度，避免内容挤压 */
  @media (max-width: 768px) {
    width: 80px !important;
  }
  @media (max-width: 480px) {
    width: 60px !important;
    font-size: 14px; /* 减小字体，节省空间 */
  }
}

/* 输入框宽度：小屏占满容器 */
:deep(.el-input) {
  @media (max-width: 768px) {
    width: 100% !important;
  }
}

/* 单选框组样式优化 */
:deep(.el-radio-group) {
  display: flex;
  gap: 20px;
  margin-top: 5px;
  @media (max-width: 480px) {
    flex-direction: column;
    gap: 10px;
  }
}
</style>

```

CalculatorResult.vue

```
<template>
  <el-card
      shadow="hover"
      header="还款计算结果"
      v-if="showResult"
      style="margin-top: 20px;"
  >
    <!-- 总计信息：Element Plus 栅格布局 -->
    <el-row :gutter="20" style="margin-bottom: 20px;">
      <!-- 还款总年限 -->
      <el-col :span="6">
        <div class="summary-item">
          <label>还款总年限</label>
          <div class="value">{{ summary.years || '-' }} 年（{{ summary.totalMonths || '-' }} 个月）</div>
        </div>
      </el-col>
      <!-- 总还款本金 -->
      <el-col :span="6">
        <div class="summary-item">
          <label>总还款本金</label>
          <div class="value">{{ formatMoney(summary.totalAllPrincipal) }} 元</div>
        </div>
      </el-col>
      <!-- 总还款利息 -->
      <el-col :span="6">
        <div class="summary-item">
          <label>总还款利息</label>
          <div class="value">{{ formatMoney(summary.totalAllInterest) }} 元</div>
        </div>
      </el-col>
      <!-- 总还款金额 -->
      <el-col :span="24">
        <div class="summary-item total">
          <label>总还款金额</label>
          <div class="value">{{ formatMoney(summary.totalAllRepay) }} 元</div>
        </div>
      </el-col>
    </el-row>

    <!-- 标签页：Element Plus 自带 -->
    <el-tabs v-model="activeTab" type="card">
      <el-tab-pane label="月度还款明细" name="monthly">
        <!-- 空数据提示 -->
        <div v-if="!loading && (monthlyDetails === null || monthlyDetails.length === 0)" class="empty-tip">
          <el-empty description="暂无月度还款数据" />
        </div>
        <el-table
            :data="monthlyDetails"
            border
            v-loading="loading"
            style="width: 100%;"
            height="400px"
            v-else
            :empty-text="''"
        >
          <el-table-column prop="month" label="还款月份" align="center" />
          <el-table-column prop="monthlyPrincipal" label="当月本金（元)" align="right">
            <template #default="scope">{{ formatMoney(scope.row.monthlyPrincipal) }}</template>
          </el-table-column>
          <el-table-column prop="monthlyInterest" label="当月利息（元)" align="right">
            <template #default="scope">{{ formatMoney(scope.row.monthlyInterest) }}</template>
          </el-table-column>
          <el-table-column prop="monthlyRepay" label="当月还款总额（元)" align="right">
            <template #default="scope">{{ formatMoney(scope.row.monthlyRepay) }}</template>
          </el-table-column>
          <el-table-column prop="remainingPrincipal" label="剩余本金（元)" align="right">
            <template #default="scope">{{ formatMoney(scope.row.remainingPrincipal) }}</template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="年度还款汇总" name="yearly">
        <!-- 调试：打印数据到控制台（方便排查） -->
        <div style="display: none;">{{ yearlySummaries }}</div>
        <!-- 空数据提示：优化判断逻辑 -->
        <div v-if="!loading && (yearlySummaries === null || yearlySummaries.length === 0)" class="empty-tip">
          <el-empty description="暂无年度还款数据" />
        </div>
        <el-table
            :data="yearlySummaries"
            border
            v-loading="loading"
            style="width: 100%;"
            height="400px"
            v-else
            :empty-text="''"
        >
          <!-- 修复：移除重复的year列，年份不使用金额格式化 -->
          <el-table-column prop="year" label="还款年度" align="center" />
          <el-table-column prop="yearPrincipal" label="当年本金（元)" align="right">
            <template #default="scope">{{ formatMoney(scope.row.yearPrincipal) }}</template>
          </el-table-column>
          <el-table-column prop="yearInterest" label="当年利息（元)" align="right">
            <template #default="scope">{{ formatMoney(scope.row.yearInterest) }}</template>
          </el-table-column>
          <el-table-column prop="yearTotalRepay" label="当年还款总额（元)" align="right">
            <template #default="scope">{{ formatMoney(scope.row.yearTotalRepay) }}</template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
    </el-tabs>
  </el-card>
</template>

<script setup>
import { ref, defineProps, watch, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'

// 接收父组件传递的参数
const props = defineProps({
  summary: {
    type: Object,
    default: () => ({
      loanTotal: '-',
      years: '-',
      totalMonths: '-',
      totalAllPrincipal: '-',
      totalAllInterest: '-',
      totalAllRepay: '-'
    }),
    required: true
  },
  monthlyDetails: {
    type: Array,
    default: () => [],
    required: true
  },
  yearlySummaries: {
    type: Array,
    default: () => [],
    required: true
  },
  loading: {
    type: Boolean,
    default: false
  },
  showResult: {
    type: Boolean,
    default: false
  }
})

// 激活的标签页
const activeTab = ref('monthly')

// 调试：组件挂载时检查年度数据
onMounted(() => {
  console.log('年度还款数据：', props.yearlySummaries)
  // 如果数据为空，给出调试提示
  if (props.showResult && props.yearlySummaries.length === 0) {
    console.warn('年度还款数据为空，请检查：1.后端是否返回yearlySummaries 2.字段名是否匹配 3.父组件是否正确传递')
  }
})

// 监听数据变化，给用户友好提示
watch([() => props.monthlyDetails, () => props.yearlySummaries], () => {
  if (props.showResult && !props.loading) {
    // 区分提示类型
    if (props.yearlySummaries.length > 0) {
      ElMessage.success('还款数据已更新（包含年度汇总）')
    } else {
      ElMessage.success('月度还款数据已更新（暂无年度数据）')
    }
  }
}, { deep: true })

// 金额格式化 - 增强版（移除debugger，优化逻辑）
const formatMoney = (num) => {
  // 处理空值、字符串、非数字情况
  if (num === null || num === undefined || num === '-' || isNaN(Number(num))) {
    return '0.00'
  }

  // 转换为数字并保留两位小数
  const number = Number(num)
  return number.toLocaleString('zh-CN', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  })
}
</script>

<style scoped>
/* 仅保留少量自定义样式，大部分由 Element Plus 提供 */
.summary-item {
  padding: 12px 15px;
  border: 1px solid #e6e6e6;
  border-radius: 6px;
  background-color: #fafafa;
  transition: all 0.3s ease;
}

.summary-item:hover {
  border-color: #409eff;
  background-color: #f5f7fa;
}

.summary-item label {
  color: #606266;
  font-size: 14px;
  margin-bottom: 8px;
  display: block;
}

.summary-item .value {
  color: #303133;
  font-size: 16px;
  font-family: "Microsoft YaHei", sans-serif;
}

.summary-item.total {
  background-color: #fff8e6;
  border-color: #ffd591;
}

.summary-item.total .value {
  color: #e6a23c;
  font-weight: bold;
  font-size: 18px;
}

/* 空数据提示样式 */
.empty-tip {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 400px;
}

/* 表格单元格样式优化 */
:deep(.el-table td),
:deep(.el-table th) {
  text-align: center;
}

:deep(.el-table .cell) {
  font-family: "Microsoft YaHei", sans-serif;
}

/* 标签页样式优化 */
:deep(.el-tabs__header) {
  margin-bottom: 15px;
}
</style>

```

