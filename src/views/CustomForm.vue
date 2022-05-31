<template>
  <el-main class="container">
    <el-form ref="form" :model="form">
      <template v-for="(item, index) in form.itemList">
        <div :key="index">
          <el-col :span="18">
            <el-form-item
                :prop="'itemList.' + index + '.timeRangeArr'"
                :rules="timeRangeArrRules"
            >
              <el-date-picker
                  v-model="item.timeRangeArr"
                  type="daterange"
                  range-separator="至"
                  start-placeholder="开始日期"
                  end-placeholder="结束日期"
                  :picker-options="pickerOptionArr[index]"
              />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item
                :prop="'itemList.' + index + '.number'"
                :rules="[
          {
            pattern: /^-?\d{0,9}$/,
            message: '只能输入9位整数',
            trigger: 'blur',
          }
        ]">
              <el-input
                  placeholder="请输入数字"
                  v-model="item.number"
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item>
              <el-button v-if="index === form.itemList.length - 1" @click="addTimeRange(index)">新增</el-button>
            </el-form-item>
          </el-col>
        </div>
      </template>
    </el-form>
  </el-main>
</template>

<script>
export default {
  name: 'CustomForm',
  components: {},
  data() {
    // 检验规则
    let checkRangeDate = (rule, value, callback) => {
      let startDate = value[0].getTime()
      let endDate = value[1].getTime()
      let isInSelectedRange = false
      this.getAllInRange(startDate, endDate, 'validator').some(item => {
        if (this.selectedDateList.includes(item)) {
          isInSelectedRange = true
        }
      })
      if (isInSelectedRange) {
        return callback(new Error('选择的日期范围重合，请重新选择'))
      } else {
        callback()
      }
    }
    return {
      // 时间范围数据列表
      form: {
        itemList: [
          {
            timeRangeArr: [],
            number: ''
          }
        ]
      },
      // 选择范围选择器配置数组
      pickerOptionArr: [
        {}
      ],
      // 已选日期数组
      selectedDateList: [],
      timeRangeArrRules: [
        {required: true, message: '请选择日期范围', trigger: ['blur', 'change']},
        {validator: checkRangeDate, trigger: ['blur', 'change']}
      ]
    }
  },
  computed: {},
  watch: {},
  mounted() {
  },
  methods: {
    // 获取开始日期与结束日期之间的所有数组
    getAllInRange(startDate, endDate, type = "disabled") {
      if (startDate === endDate) {
        return [startDate]
      }
      // 判断处理类型
      // disabled：处理日期禁用
      // validator：处理校验
      if (type !== "disabled") {
        let tempArr = []
        let controlWhile = true
        while (controlWhile) {
          // 开始时间小于等于结束时间
          if (startDate <= endDate) {
            tempArr.push(startDate)
            // 递增一天的毫秒数
            startDate += 24 * 60 * 60 * 1000
          } else {
            controlWhile = false
          }
        }
        return tempArr
      } else {
        let controlWhile = true
        while (controlWhile) {
          // 开始时间小于等于结束时间
          if (startDate <= endDate) {
            // 如果已存在则不追加进数组
            if (!this.selectedDateList.includes(startDate)) {
              this.selectedDateList.push(startDate)
            }
            startDate += 24 * 60 * 60 * 1000
          } else {
            controlWhile = false
          }
        }
      }
    },
    addTimeRange(index) {
      // 单个校验规则
      this.$refs.form.validateField('itemList.' + index + '.timeRangeArr', (valid) => {
        // 如果valid有值就说明没有通过校验
        if (valid) {
          return false
        } else {
          let startDate = this.form.itemList[index].timeRangeArr[0].getTime()
          let endDate = this.form.itemList[index].timeRangeArr[1].getTime()
          // 判断本次选择的开始时间和结束时间的范围是否已被选择
          let isInSelectedRange = false
          this.getAllInRange(startDate, endDate, 'validator').some(item => {
            if (this.selectedDateList.includes(item)) {
              isInSelectedRange = true
            }
          })
          // 如果重复则返回错误提示
          if (isInSelectedRange) {
            return false
          }
          // 如果没有重复则将其范围内的日期数组全部列举出来
          this.getAllInRange(startDate, endDate)
          this.pickerOptionArr.push({
            disabledDate: (time) => {
              return this.selectedDateList.includes(time.getTime())
            }
          })
          this.form.itemList.push({
            timeRangeArr: '',
            number: ''
          })
        }
      })
    }
  }
}
</script>

<style scoped lang="scss">
.container {
  display: flex;
  justify-content: center;
  height: 100vh;
  background-color: #fafafa;
}
</style>
