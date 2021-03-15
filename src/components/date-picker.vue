<template>
  <div class="cc-calendar">
    <calendarHeader
      :headOptions="headOptions"
      @handlePrevMonth="handlePrevMonth"
      @handleNextMonth="handleNextMonth"
      @handleToday="handleToday"
    />
    <ul class="calendar-week clear">
      <li
        v-for="(item, index) in calendarTitleArr"
        :key="index"
        class="week-item"
      >
        {{ item }}
      </li>
    </ul>
    <ul class="calendar-view clear">
      <li
        v-for="(item, index) in visibleCalendar"
        :key="index"
        class="date-view"
        :class="[
          { 'month-class': !isCurrentMonth(item.date) },
          { todayBg: isCurrentDay(item.date) },
          { handleDay: item.clickDay },
        ]"
        @click="handleClickDay(item)"
      >
        <span
          class="date-day"
          :class="[{ 'opacity-class': !isCurrentMonth(item.date) }]"
        >
          {{ item.day }}
        </span>
        <span class="rate-num"> 56% </span>
      </li>
    </ul>
  </div>
</template>

<script>
import "../assets/css/reset.min.css";
import calendarHeader from "./canlendar-head.vue";
import * as utils from "../assets/js/utils.js";

export default {
  name: "cc-calendar",
  componentName: "cc-calendar",
  props: {
    options: Object,
  },
  components: {
    calendarHeader,
  },
  data() {
    let { year, month, day } = utils.getNewDate(new Date());
    return {
      headOptions: {
        type: this.options.type,
        style: this.options.headStyle,
        date: "",
      },
      calendarTitleArr: ["日", "一", "二", "三", "四", "五", "六"],
      time: { year, month, day },
      calendarList: [],
    };
  },
  computed: {
    dayStyle: function () {
      return {
        textAlign: this.options.viewStyle.day,
      };
    },
    visibleCalendar() {
      let calendatArr = [];
      let { year, month, day } = utils.getNewDate(
        utils.getDate(this.time.year, this.time.month, this.time.day)
      );
      console.log(`date is ${year} ${month} ${day}`);
      //根据`一部手机管
      let currentFirstDay = utils.getDate(year, month, 1);
      // 获取当前月第一天星期几
      let weekDay = currentFirstDay.getDay();
      let startTime = null;
      if (weekDay == 0) {
        /*
        这里需要做运算 后面是基于startTime为number类型计算时间
        如果这里直接是currentFirstDay的话startTime会是一个Date对象而不是Number
        */
        startTime = currentFirstDay - 0;
      } else {
        startTime = currentFirstDay - weekDay * 24 * 60 * 60 * 1000;
      }
      console.log(`startTime is ${startTime}`);
      let monthDayNum;
      console.log(`weekDay is ${weekDay}`);
      if (weekDay == 5 || weekDay == 6) {
        monthDayNum = 42;
      } else {
        monthDayNum = 35;
      }
      let dateDay = new Date(startTime + 0 * 24 * 60 * 60 * 1000).getDate();
      let date = new Date(currentFirstDay);
      date.setDate(date.getDate() + 30);
      console.log(`dateDay is ${dateDay}`);
      console.log(`date is ${date}`);
      for (let i = 0; i < monthDayNum; i++) {
        let itemMonth = month + 1;
        console.log(`startTime is ${startTime}`);
        let itemDay = new Date(startTime + i * 24 * 60 * 60 * 1000).getDate();
        let itemDate = new Date(startTime + i * 24 * 60 * 60 * 1000);
        let show = true;
        console.log(`itemDay is ${itemDay}`);
        if (!(itemDate > date)) {
          calendatArr.push({
            date: new Date(startTime + i * 24 * 60 * 60 * 1000),
            year: year,
            month: itemMonth,
            day: itemDay,
            clickDay: false,
            show,
          });
        } else {
          show = false;
          // monthDayNum -= 1;
          console.log(
            `itemDate > date itemDate is ${itemDate} date is ${date} itemDay is ${itemDay}`
          );
        }
      }

      this.headOptions.date = `${utils.englishMonth(month)} ${year}`;
      return calendatArr;
    },
  },
  methods: {
    // 是否是当前月
    isCurrentMonth(date) {
      let { year: currentYear, month: currentMonth } = utils.getNewDate(
        utils.getDate(this.time.year, this.time.month, 1)
      );
      let { year, month } = utils.getNewDate(date);
      return currentYear == year && currentMonth == month;
    },
    // 是否是今天
    isCurrentDay(date) {
      let {
        year: currentYear,
        month: currentMonth,
        day: currentDay,
      } = utils.getNewDate(new Date());
      let { year, month, day } = utils.getNewDate(date);
      return currentYear == year && currentMonth == month && currentDay == day;
    },
    // 上一个月
    handlePrevMonth() {
      let prevMonth = utils.getDate(this.time.year, this.time.month, 1);
      prevMonth.setMonth(prevMonth.getMonth() - 1);
      this.time = utils.getNewDate(prevMonth);
      this.headOptions.date = `${utils.englishMonth(this.time.month)} ${
        this.time.year
      }`;
      this.$emit("handlePrevMonth");
    },
    // 下一个月
    handleNextMonth() {
      let nextMonth = utils.getDate(this.time.year, this.time.month, 1);
      nextMonth.setMonth(nextMonth.getMonth() + 1);
      this.time = utils.getNewDate(nextMonth);
      this.headOptions.date = `${utils.englishMonth(this.time.month)} ${
        this.time.year
      }`;
      this.$emit("handleNextMonth");
    },
    // 点击回到今天
    handleToday() {
      this.time = utils.getNewDate(new Date());
      this.returnDate();
      this.$emit("handleToday");
    },
    // 点击某一天
    handleClickDay(item) {
      this.$forceUpdate();
      this.$emit("handleClickDay", item);
      this.calendarList.map((x) => {
        x.clickDay = false;
      });
      this.$set(item, "clickDay", true);
    },
  },
  created() {
    this.calendarList = this.visibleCalendar;
    this.calendarType = this.options.calendarType;
  },
};
</script>

<style lang="less">
.cc-calendar {
  // padding: 23px 30px 30px;
  width: 100%;
  height: 100%;
  background: #f9fafc;
  box-sizing: border-box;
  .calendar-week {
    width: 100%;
    height: 46px;
    line-height: 46px;
    border: 1px solid #e4e7ea;
    border-right: none;
    .week-item {
      float: left;
      width: 14.285%;
      text-align: center;
      font-size: 14px;
      color: #424953;
      border-right: 1px solid #e4e7ea;
      font-weight: 600;
    }
  }
  .calendar-view {
    width: 100%;
    border-left: 0px solid #e4e7ea;
    .date-view {
      float: left;
      width: 14.285%;
      height: 60px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      border-right: 0px solid #e4e7ea;
      border-bottom: 0px solid #e4e7ea;
      text-align: center;
      cursor: pointer;
      .date-day {
        display: block;
        width: 40px;
        height: 40px;
        font-size: 14px;
        line-height: 40px;
        color: #2d3457;
      }
      .calendar-num {
        margin-top: 6px;
        display: block;
        width: 100%;
        text-align: center;
        font-size: 30px;
        color: #424953;
      }
      .rate-num {
        font-size: 11px;
        color: #333;
      }
    }
    .date-view.todayBg {
      .date-day {
        display: block;
        width: 40px;
        height: 40px;
        font-size: 14px;
        line-height: 40px;
        color: #2d3457;
        border-radius: 50%;
        background: linear-gradient(0deg, #3abce9, #4385f3);
      }
    }
    .opacity-class {
      opacity: 0.5;
    }
    .month-class {
      background-image: linear-gradient(
        45deg,
        rgba(000, 000, 000, 0.03) 25%,
        transparent 25%,
        transparent 50%,
        rgba(000, 000, 000, 0.03) 50%,
        rgba(000, 000, 000, 0.03) 75%,
        transparent 75%,
        transparent
      );
      background-size: 20px 20px;
    }
    .todayBg {
      background: #fffdef;
    }
    .handleDay {
      background: #2061ff !important;
      .date-day {
        color: #bccfff !important;
      }
      .calendar-num {
        color: #fff !important;
      }
    }
  }
}
</style>