<template>

  <el-dialog custom-class="dia-class" :visible.sync="visibleP" :before-close="handleClose" center="">
    <header slot="title">
      <span class="title">
        {{getTitle}}
      </span>
    </header>
    <section class="layout loan-contract-form">
      <el-form ref="form" :model="detailsP" status-icon :rules="rules" label-width="130px">
        <el-row>
          <el-col :span="11" class="flex">
            <el-form-item label="贴现金额: " prop="billBookAmt">
              <span>{{detailsP.billBookAmt | regexNum}}</span>
            </el-form-item>
          </el-col>
          <el-col :span="11" :offset="1" class="flex">
            <el-form-item label="放款比例: " prop="loanPer">
             <el-input v-model="detailsP.loanPer"  placeholder="放款比例">
               <template slot="append">%</template>
             </el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11"  class="flex">
            <el-form-item label="实放金额: " prop="actualDiscountAmt">
             <el-input v-model="actualDiscountAmt" placeholder="实放金额"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11" class="flex">
            <el-form-item label="年利率: " prop="interestRate">
             <el-input v-model="detailsP.interestRate" placeholder="年利率">
               <template slot="append">%</template>
             </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="11" :offset="1" class="flex">
             <el-form-item label="服务费率: " prop="serviceFeeRate">
              <el-input v-model="detailsP.serviceFeeRate" placeholder="服务费率">
                <template slot="append">%</template>
              </el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11" class="flex">
             <el-form-item label="逾期利率: " prop="overdueRate">
             <el-input v-model="detailsP.overdueRate" placeholder="逾期利率">
               <template slot="append">%</template>
             </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="11" :offset="1" class="flex">
            <el-form-item label="提前还款手续费: " prop="prepaymentDeductInterest">
              <el-input v-model="detailsP.prepaymentDeductInterest"  placeholder="提前还款手续费"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11" class="flex">
             <el-form-item label="还款方式: " prop="repaymentType">
             <el-select v-model="detailsP.repaymentType" clearable placeholder="还款方式">
              <el-option v-for="(item,index) in moneyTypes" :key="index" :label="item.RepaymentTypeName" :value="item.RepaymentType"></el-option>
            </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="11" :offset="1" class="flex">
             <el-form-item label="宽容天数: " prop="fineGraceDays">
             <el-input v-model="detailsP.fineGraceDays"  placeholder="宽容天数"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11" class="flex">
             <el-form-item label="预计回款日期: " prop="billDueDate">
              <span>{{detailsP.billPayDate|dateFormat}}</span>
            </el-form-item>
          </el-col>
          <el-col :span="11" :offset="1" class="flex">
            <el-form-item label="预计还款日期: " prop="billPayDate"
            :rules="[
          { required: !detailsP.billDueDate, message: '请输入预计还款日期', trigger: 'blur' }
        ]">
             <el-date-picker :editable="false" :disabled="!!detailsP.billPayDate" v-model="detailsP.billDueDate" type="date" placeholder="选择日期">
            </el-date-picker>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-alert
            title="声明:在预计回款日期存在的情况下将以预计回款日期为准,在预计回款日期没有的情况下,需和供应商商议合理的预计还款日期填写,此处平台不承担责任!"
            type="warning"
            :closable="false"
            show-icon>
          </el-alert>
        </el-row>
      </el-form>
    </section>
    <footer slot="footer" :style="'clear:both'">
      <el-button type="primary" @click="handleSubmit">确认</el-button>
    </footer>
  </el-dialog>
</template>
<style lang="scss">
.layout.loan-contract-form {
  margin-top: 10px;
  .el-row {
    margin-top: 10px;
  }
}

.layout.loan-contract-form .flex {
  display: flex;
  label {
    width: 150px;
    height: 40px;
    line-height: 40px;
  }
  span {
    height: 40px;
    line-height: 40px;
  }
  .el-select,
  .el-date-editor.el-input,
  .el-date-editor.el-input__inner {
    width: 180px;
  }
  .el-input-group__append {
    padding: 0 5px;
  }
}
</style>

<script>
import DialogClose from '@/mixins/suplier/Ar/DialogClose'
import Common from '@/mixins/common'
import { debounce } from '@/util/util' // 防抖函数
import { loadingConf } from '@/config/common' // 获取加载配置

export default {
  props: ['visibleP', 'detailsP'],
  mixins: [DialogClose, Common],
  data () {
    return {
      transAmt: 0,
      checkList: [],
      moneyTypes: [{
        RepaymentType: 2,
        RepaymentTypeName: '一期还息，期末清偿法'
      },
      {
        RepaymentType: 1,
        RepaymentTypeName: '一次还款本息'
      }
      ],
      rules: {
        loanPer: [
          { required: true, message: '请输入放款比例', trigger: 'blur' },
          { validator: checkRate, trigger: 'blur' }
        ],
        actualDiscountAmt: [
          { required: true, message: '实放金额不能为空', trigger: 'blur' },
          { validator: checkNumber, trigger: 'blur' }
        ],
        interestRate: [
          { required: true, message: '请输入贴现利率', trigger: 'blur' },
          { validator: checkRate, trigger: 'blur' }
        ],
        serviceFeeRate: [
          { required: true, message: '请输入服务费率', trigger: 'blur' },
          { validator: checkRate, trigger: 'blur' }
        ],
        overdueRate: [
          { required: true, message: '请输入逾期利率', trigger: 'blur' },
          { validator: checkRate, trigger: 'blur' }
        ],
        prepaymentDeductInterest: [
          { required: true, message: '请输入提前还款手续费', trigger: 'blur' },
          { validator: checkNumber, trigger: 'blur' }
        ],
        fineGraceDays: [
          { required: true, message: '请输入宽容天数', trigger: 'blur' },
          { validator: checkNumber, trigger: 'blur' }
        ]
      }
    }
  },
  watch: {
    /** 修复只根据放款比例计算得到结果 输入的值无法获取  */
    loanPer: debounce(function (val) {
      this.detailsP.actualDiscountAmt = this.detailsP.billBookAmt * val / 100
    }, 1000)
  },
  computed: {
    loanPer () {
      return this.detailsP.loanPer
    },
    actualDiscountAmt: {
      // getter
      get: function () {
        return (this.detailsP.billBookAmt * (this.detailsP.loanPer * 100) / 10000).toFixed(2)
      },
      // setter
      set: debounce(function (newValue) {
        this.detailsP.actualDiscountAmt = newValue
        let val = Number((newValue / this.detailsP.billBookAmt * 100).toFixed(2))
        console.log(typeof (val))
        this.detailsP.loanPer = val
        console.log(this.detailsP.loanPer)
      }, 1000)
    },
    getTitle () {
      return this.detailsP.masterChainId + '合同利益确认'
    }
  },
  methods: {
    handleSubmit: debounce(submit, 1000, true)
  }
}
// 提交操作
function submit () {
  console.log(this.detailsP)
  this.$refs.form.validate((valid) => {
    if (valid) {
      const param = {
        masterChainId: this.detailsP.masterChainId,
        supplierCustId: this.detailsP.supplierCustId,
        billBookAmt: this.detailsP.billBookAmt, // 贴现金额
        loanPer: this.loanPer, // 放款比例
        actualDiscountAmt: this.detailsP.actualDiscountAmt || '', // 实放金額 修复只根据放款比例计算得到结果
        interestRate: this.detailsP.interestRate || '', // 贴现利率
        serviceFeeRate: this.detailsP.serviceFeeRate || '', // 服务费率
        overdueRate: this.detailsP.overdueRate || '', // 逾期利率
        prepaymentDeductInterest: this.detailsP.prepaymentDeductInterest || '', // 提前还款手续费
        repaymentType: this.detailsP.repaymentType || '', // 还款方式
        fineGraceDays: this.detailsP.fineGraceDays || '', // 宽容天数
        billPayDate: this.detailsP.billPayDate, // 预计回款日期
        billDueDate: this.detailsP.billDueDate // 预计还款日期
      }
      console.log(param)
      // 显示加载图标
      const loading = this.$loading(loadingConf.sub())
      // 发送数据
      this.axios.post('/loan2/generateContract.do', param).then(res => {
        let type = res.data.status ? 'success' : 'error'
        this.$message({
          message: res.data.data ? res.data.data : '返回结果错误，请联系管理员',
          type: type
        })
        // 关闭加载图标
        loading.close()
        // 操作成功关闭弹窗刷新数据
        if (res.data.status) {
          this.$parent.fresh()
          this.handleClose()
        }
      }).catch((err) => {
        console.log(err)
        this.$message({
          type: 'info',
          message: '操作失败'
        })
        // 关闭加载图标
        loading.close()
      })
    }
  })
}
// async 校验规则
// 利率规则
var checkRate = (rule, value, callback) => {
  if (!value) {
    return callback(new Error('不能为空'))
  }
  let re = /^([1-9]\d*\.\d*|0\.\d+|[1-9]\d*|0)$/
  setTimeout(() => {
    if (!re.test(value)) {
      callback(new Error('请输入大于等于0的数字'))
    } else {
      if (value < 0 || value > 100) {
        callback(new Error('必须为0-100之间'))
      } else {
        callback()
      }
    }
  }, 1000)
}
// 数字规则
var checkNumber = (rule, value, callback) => {
  if (!value) {
    return callback(new Error('不能为空'))
  }
  let re = /^([1-9]\d*\.\d*|0\.\d+|[1-9]\d*|0)$/
  setTimeout(() => {
    if (!re.test(value)) {
      callback(new Error('请输入大于等于0的数字'))
    } else {
      if (value < 0) {
        callback(new Error('必须大于等于0'))
      } else {
        callback()
      }
    }
  }, 1000)
}
</script>
