<template>
  <div class="question">
    <img class="bg" src="/static/bg.png" alt="">
    <div class="qs_content" v-if="questions.length > 0">
      <div v-if="!startqs">
        <p class="title">温馨提示:</p>
        <div class="warn_tag">请先回答问题！！</div>
      </div>
      <div v-else>
        <p class="title">{{questions[currentIndex].title}}</p>
        <div
          class="response"
          v-for="(item,index) in questions[currentIndex].option"
          :key="index"
          @click="selectAnswer(index)"
        >
          <img v-if="item.select" src="/static/selected.jpg" alt>
          <img v-else src="/static/unselect.jpg" alt>
          <span>{{item.label}}</span>
        </div>
      </div>
    </div>
    <button :disabled="disabled" @click="btn_click" class="qs_btn">{{btn_title}}</button>
  </div>
</template>
<script>
export default {
   data() {
    return {
      questions: [],
      startqs: false, // 是否显示问题列表
      currentIndex: 0, // 记录当前回答到第几道题
      btn_title: "ACTION", // 按钮的title
      lesson: "" // 记录提交的问题
    };
  },
  computed: {
    disabled() {
      if (!this.startqs) return false;
      else {
        const option = this.questions[this.currentIndex].option;
        let select = false;

        option.forEach(item => {
          if (item.select) select = true;
        });

        return !select;
      }
    }
  },
  onLoad() {
    this.getData();
  },
  methods: {
    getData() {
      this.$https
        .request({
          url: this.$interfaces.getQuestions,
          method: "get"
        })
        .then(res => {
          console.log(res);
          this.questions = res.questions;
        });
    },
    btn_click() {
      // 如果没有开始测试(this.startqs为false) 按钮标题默认显示’开始答题’如果已经开始测试 判断是否为最后一题（this.currentIndex == this.questions.length - 1）如果不是最后一题，按钮标题为’下一题’ 否则显示’完成’

      if (!this.startqs) {
        this.startqs = true;
        this.btn_title = "下一题";
      } else {
        // 是,vue,node
        this.lesson += this.getSelectOption();
        if (this.currentIndex < this.questions.length - 1) {
          this.btn_title = "下一题";
          this.currentIndex++;
          // 拼接逗号
          this.lesson += ",";
          if (this.currentIndex == this.questions.length - 1) {
            this.btn_title = "完成";
          }
        } else {
          // 完成
          console.log(this.lesson);
          this.sendQuestions();
        }
      }
    },
    selectAnswer(index) {
      const option = this.questions[this.currentIndex].option;
      // 将列表中的是否选项都置为未选中
      option.forEach(item => {
        item.select = false;
      });
      // 将点击的选项选中
      option[index].select = !option[index].select;
    },
    getSelectOption() {
      const option = this.questions[this.currentIndex].option;
      let label = "";
      option.forEach(item => {
        if (item.select) label = item.label;
      });
      return label;
    },
    sendQuestions() {
      // 定义默认等级为初级
      wx.setStorage({
        key: "grade",
        data: "初级"
      });
      this.$https
        .request({
          url: this.$interfaces.myLesson,
          method: "post",
          data: {
            // lesson拼接的问题答案
            lesson: this.lesson,
            // openid
            userId: this.$store.getters.openId
          }
        })
        .then(res => {
          // console.log(res);
          // 存储课程信息
          this.$store.dispatch("setLessonInfo", res);

          // 跳转到学习页面
          wx.switchTab({
            url: "../news/main"
          });
        });
    }
  }
}
</script>
<style scoped>
.question {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
}
 .bg {
  width: 100%;
  height:600px;
}
.qs_content {
  position: absolute;
  width: 80%;
  height: 50%;
  background: #fff;
  top: 36%;
  left: 10%;
  border-radius: 5px;
  box-sizing: border-box;
  padding: 10px;
  color: rgb(13, 28, 51);
}
.qs_btn {
  position: absolute;
  top: 90%;
  left: 10%;
  width: 80%;
  background-color: #009eef;
  color: white;
}
.title {
  font-weight: bold;
  margin-bottom: 20px;
  word-break: break-all;
}
.warn_tag {
  padding: 40px 10px;
}
.response {
  padding: 10px;
}
.response img {
  width: 20px;
  height: 20px;
  margin-right: 10px;
}
</style>





