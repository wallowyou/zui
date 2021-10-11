<template>
  <view class="z-calendar">
    <view class="z-calendar-header">
      <view class="z-calendar-left">
        <text
          class="pre-year iconfont icon-a-doubleleft2x"
          @click="previousYear"
          v-show="showYearControl"
        ></text>
        <text
          class="pre-month iconfont icon-a-left2x"
          @click="previousMonth"
          v-show="showMonthControl"
        ></text>
      </view>
      <view class="z-calendar-center">
        {{ timeStr }}
      </view>
      <view class="z-calendar-right">
        <text
          class="next-month iconfont icon-a-right2x"
          @click="nextMonth"
          v-show="showMonthControl"
        ></text>
        <text
          class="next-year iconfont icon-a-doubleright2x"
          @click="nextYear"
          v-show="showYearControl"
        ></text>
      </view>
    </view>
    <view class="z-calendar-week">
      <view class="z-calendar-day" v-for="day in weeks" :key="day">{{
        day
      }}</view>
    </view>
    <view class="z-calenda-wrap">
      <swiper
        class="swiper"
        :indicator-dots="false"
        :autoplay="false"
        @change="change"
        :current="current"
        :circular="true"
        :disable-touch="!touchSwitch"
      >
        <swiper-item v-for="i in [0, 1, 2]" :key="i">
          <view class="swiper-item">
            <view class="z-calendar-dates">
              <view
                class="z-calendar-date"
                v-for="date in current === i
                  ? daysMap.curs
                  : current - i === 1 || current - i === -2
                  ? daysMap.pres
                  : daysMap.nexts"
                :key="date.timestamp"
              >
                <view
                  class="z-calendar-date-inner"
                  @click="dateClick(date)"
                  :class="{
                    istoday: date.isToday,
                    ishighlight: date.isHighlight,
                    notmonth: date.isPreMonth || date.isNextMonth,
                  }"
                >
                  {{ date.date }}
                  <view
                    class="isdot"
                    v-show="date.isDot"
                    :style="{
                      top: dotPosition.top + 'rpx',
                      right: dotPosition.right + 'rpx',
                      background: dotColor,
                    }"
                  ></view>
                </view>
              </view>
            </view>
          </view>
        </swiper-item>
      </swiper>
    </view>
  </view>
</template>

<script>
const WEEKS = ["日", "一", "二", "三", "四", "五", "六"];
import { parseTime, getDateList, compareDates } from "./date-util";
export default {
  name: "ZCalendar",
  props: {
    type: {
      type: String,
      default: "date", // 模式
    },
    timeFormatter: {
      type: String,
      default: "yyyy年MM月", // 头部日期月份格式化
    },
    touchSwitch: {
      type: Boolean,
      default: true, // 是否可以滑动切换
    },
    showYearControl: {
      type: Boolean,
      default: true, // 是否显示上一年下一年控制按钮
    },
    showMonthControl: {
      type: Boolean,
      default: true, // 是否显示上一月下一月控制按钮
    },
    firstDayOfWeek: {
      default: 1,
      type: Number,
      validator: (val) => val === 1 || val === 7, //是从周一开始还是周日开始
    },
    value: {
      type: Date,
      default() {
        return new Date();
      },
    },
    dotDates: {
      type: Array,
      default() {
        return [];
      },
    },
    dotPosition: {
      type: Object, // 打点小圆点的位置
      default() {
        return {
          top: 10,
          right: 16,
        };
      },
    },
    dotColor: {
      type: String,
      default: "#faad14", // 打卡圆点的颜色
    },
  },
  data() {
    return {
      pageDate: new Date(),
      currentDate: this.value,
      current: 1,
    };
  },
  watch: {
    value(val) {
      this.currentDate = val;
    },
  },
  computed: {
    timeStr() {
      return parseTime(this.pageDate, this.timeFormatter);
    },
    weeks() {
      if (this.firstDayOfWeek === 7) {
        return WEEKS;
      } else {
        return WEEKS.slice(1).concat(["日"]);
      }
    },
    daysMap() {
      let d = this.pageDate;
      let preD = new Date(
        d.getFullYear(),
        d.getMonth() - 1,
        1,
        d.getHours(),
        d.getMinutes()
      );
      let nextD = new Date(
        d.getFullYear(),
        d.getMonth() + 1,
        1,
        d.getHours(),
        d.getMinutes()
      );
      let pres = getDateList(preD, this.firstDayOfWeek);
      let curs = getDateList(d, this.firstDayOfWeek);
      let nexts = getDateList(nextD, this.firstDayOfWeek);

      curs.forEach((item) => {
        item.isHighlight = compareDates(item.dateObj, this.currentDate);
        item.isDot = !!this.dotDates.find((i) => {
          return compareDates(item.dateObj, new Date(i.date));
        });
      });

      return {
        pres,
        curs,
        nexts,
      };
    },
  },

  mounted() {},
  methods: {
    changeMonth(incrementBy) {
      const date = this.pageDate;
      this.pageDate = new Date(
        date.getFullYear(),
        date.getMonth() + incrementBy,
        date.getDate()
      );
      this.setCurrentDate();
      this.emitMonthChange();
    },
    changeYear(incrementBy) {
      const date = this.pageDate;
      this.pageDate = new Date(
        date.getFullYear() + incrementBy,
        date.getMonth(),
        date.getDate()
      );
      this.setCurrentDate();
      this.emitMonthChange();
    },
    dateClick(e) {
      const { dateObj, isPreMonth, isNextMonth } = e;
      if (isPreMonth || isNextMonth) {
        return;
      }

      // 复制副本
      this.currentDate = new Date(
        dateObj.getFullYear(),
        dateObj.getMonth(),
        dateObj.getDate()
      );
      this.$emit("input", this.currentDate);
      this.$emit("on-selected", this.currentDate);
    },
    previousMonth() {
      this.changeMonth(-1);
    },
    nextMonth() {
      this.changeMonth(+1);
    },
    previousYear() {
      this.changeYear(-1);
    },
    nextYear() {
      this.changeYear(+1);
    },
    setCurrentDate() {
      // 切换月份或者年份之后，默认选中当前月的1号
      const y = this.pageDate.getFullYear();
      const m = this.pageDate.getMonth();
      this.currentDate = new Date(y, m, 1);
      this.$emit("input", this.currentDate);
      this.$emit("on-selected", this.currentDate);
    },
    emitMonthChange() {
      this.$emit("month-change", this.currentDate);
    },
    change(e) {
      let preCurrent = this.current;
      let current = e.detail.current;
      this.current = current;
      if (current - preCurrent === 1 || current - preCurrent === -2) {
        this.nextMonth();
      } else if (current - preCurrent === -1 || current - preCurrent === 2) {
        this.previousMonth();
      }
    },
  },
};
</script>

<style lang="scss" scoped>
@import "./fonts/iconfont.css";
.z-calendar {
  width: 100%;
  &-header {
    display: flex;
    flex-flow: row nowrap;
    justify-content: flex-start;
    align-content: center;
    font-size: 34rpx;
    font-weight: 500;
    color: #000000;
    height: 60rpx;
    line-height: 60rpx;
  }
  &-left,
  &-right {
    width: 160rpx;
    font-size: 32rpx;
    color: rgba(0, 0, 0, 0.45);
  }
  &-center {
    flex: 1;
    text-align: center;
  }
  &-left {
    text-align: left;
  }
  &-right {
    text-align: right;
  }
  &-week {
    display: flex;
    flex-flow: row nowrap;
    justify-content: flex-start;
    align-items: center;
    height: 60rpx;
    line-height: 60rpx;
    font-size: 34rpx;
    font-weight: 500;
    color: #bfbfbf;
    margin-top: 16rpx;
  }
  &-day {
    flex: 1;
    text-align: center;
    height: 100%;
  }
  &-dates {
    display: flex;
    flex-flow: row wrap;
    justify-content: flex-start;
    font-weight: 400;
    color: rgba(0, 0, 0, 0.65);
    font-size: 30rpx;
  }
  &-date {
    height: 75rpx;
    width: calc(100% / 7);
    text-align: center;
    margin-top: 12rpx;
    line-height: 75rpx;
  }
  &-date-inner {
    width: 75rpx;
    height: 75rpx;
    margin: 0 auto;
    position: relative;
  }
  .swiper {
    height: 516rpx;
  }
}
.pre-month {
  margin-left: 40rpx;
}
.next-month {
  margin-right: 40rpx;
}

.istoday {
  color: #203dbc;
}
.notmonth {
  color: rgba(0, 0, 0, 0.25);
}
.ishighlight {
  background: #203dbc;
  color: #ffffff;
  border-radius: 50%;
}
.isdot {
  display: block;
  content: "";
  position: absolute;
  width: 8rpx;
  height: 8rpx;
  border-radius: 50%;
}
</style>
