<template>
  <view class="content">
    <z-calendar
      :firstDayOfWeek="1"
      v-model="checkDate"
      :dotDates="ondutys"
      @month-change="monthChange"
      @on-selected="dateSelected"
    ></z-calendar>
  </view>
</template>

<script>
import ZCalendar from "../../components/z-calendar/z-calendar.vue";
export default {
  components: {
    ZCalendar,
  },
  data() {
    return {
      title: "Hello",
      checkDate: new Date(),
      ondutys: [{ date: "2021-10-11" }],
    };
  },
  onLoad() {},
  methods: {
    monthChange(e) {
      console.log(e);
    },
    dateSelected(e) {
      console.log(e);
    },
  },
};
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 12rpx;
}
</style>
