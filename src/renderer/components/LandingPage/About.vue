<template>
  <div>
    <div class="items">
      <div class="title">学生 & web开发</div>
      <div class="item" v-for="(item, index) in tips" :key="index">
        <div class="name" v-text="item.name" />
        <div class="value" v-text="item.value" />
      </div>
    </div>
  </div>
</template>

<script>
import { platform, release, arch } from "os";
export default {
  data() {
    return {
      tips: [
        { name: "GitHub：", value: "https://github.com/Jackwensen" },
        { name: "技能：", value: "JavaScript/Vue/node.js/JAVA/Git/Python/SQL" },
        { name: "生日：", value: "1997年4月30日" },
        { name: "地区：", value: "河北石家庄" },
        { name: "邮箱：", value: "744915067@qq.com" },
        { name: "实习经历：", value: "水岩科技 & 美团·大众点评" },
      ],
    };
  },
  mounted() {
    console.log(this.$route);
  },
};
</script>

<style scoped>
.title {
  color: #888;
  font-size: 18px;
  font-weight: initial;
  letter-spacing: 0.25px;
  margin-top: 10px;
  margin-bottom: 10px;
}

.items {
  margin-top: 8px;
}

.item {
  display: flex;
  align-items: center;
  margin-bottom: 6px;
  line-height: 24px;
}

.item .name {
  color: #151a5f;
  margin-right: 6px;
  font-family: "Hiragino Sans GB";
  /* font-weight: bold; */
}

.item .value {
  color: #35495e;
  font-family: "Comic Sans MS",PingFangSC-Light;
}
</style>
