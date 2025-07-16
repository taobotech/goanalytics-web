<template>
  <div class="app-container">
    <h3>自定义事件详情 -- {{ customizedCounter.displayName }}</h3>
    <el-date-picker
      v-model="datePickerValue"
      type="daterange"
      range-separator="~"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
      value-format="timestamp"
      @change="handleDateChange"
    />
    <div class="chart-wrapper">
      <simple-counter
        v-if="customizedCounter.type === 'simple'"
        :start="adjustedDateRange[0]"
        :end="adjustedDateRange[1]"
        :counterName="customizedCounter.name + '_customized'"
        :displayName="customizedCounter.displayName"
      />
      <slot-counter-table
        v-if="customizedCounter.type === 'slot'"
        :start="adjustedDateRange[0]"
        :end="adjustedDateRange[1]"
        :slotCounterName="customizedCounter.name + '_customized'"
        :slotsMapping="getSlotsMapping(customizedCounter.slots)"
      />
      <cpv-counter
        v-if="customizedCounter.type === 'cpv'"
        :start="adjustedDateRange[0]"
        :end="adjustedDateRange[1]"
        :counterName="customizedCounter.name + '_customized'"
        :displayName="customizedCounter.displayName"
        :channels="customizedCounter.channels"
        :versions="customizedCounter.versions"
      />
    </div>
  </div>
</template>

<script>
import dateUtils from '@/utils/date'
import SimpleCounter from '@/components/Counter/SimpleCounter'
import SlotCounterTable from '@/components/Counter/SlotCounterTable'
import CpvCounter from '@/components/Counter/CPVCounter'

export default {
  name: 'CustomizedEventDetail',
  components: { SimpleCounter, SlotCounterTable, CpvCounter },
  data() {
    return {
      datePickerValue: [(dateUtils.todayTimestamp() - 30 * 24 * 60 * 60) * 1000, dateUtils.todayTimestamp() * 1000],
      adjustedDateRange: [(dateUtils.todayTimestamp() - 30 * 24 * 60 * 60) * 1000, dateUtils.todayTimestamp() * 1000],

      localStorage: {
        customizedCounter: {
          type: Object
        }
      },
      customizedCounter: {}
    }
  },
  created() {
    this.customizedCounter = JSON.parse(this.$localStorage.get('customizedCounter'))
    this.adjustDateRange()
  },
  methods: {
    getSlotsMapping(slots) {
      const mapping = {}
      for (const slot of slots) {
        mapping[slot] = slot
      }
      return mapping
    },
    handleDateChange() {
      this.adjustDateRange()
    },
    adjustDateRange() {
      if (this.datePickerValue && this.datePickerValue.length === 2) {
        const startTime = this.datePickerValue[0]
        const endTime = this.datePickerValue[1]
        
        // 开始时间保持原样（当天0点）
        // 结束时间调整为当天的23:59:59
        const endDate = new Date(endTime)
        endDate.setHours(23, 59, 59, 999)
        
        // 确保时间戳是整数，去掉毫秒部分
        this.adjustedDateRange = [
          Math.floor(startTime / 1000) * 1000,  // 转换为整数
          Math.floor(endDate.getTime() / 1000) * 1000  // 转换为整数
        ]
      }
    }
  }
}

</script>
