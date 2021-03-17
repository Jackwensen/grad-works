<template>
  <div id="wrapper">
    <img id="logo" :src="logo" alt="electron-vue" />
    <main>
      <div class="left-side">
        <span class="title">基于electron和vue的研究生复试个人作品</span>
        <system-information></system-information>
        <div class="doc">
          <el-button type="primary" round @click="openNewWin"
            >打开新窗口</el-button>
        </div>
      </div>

      <div class="right-side">
        <span class="title-right">老师好！我是 <el-link class="title-right" type="primary" :underline="false">白景文</el-link></span>
        <about></about>
      </div>
    </main>
  </div>
</template>

<script>
import SystemInformation from "./LandingPage/SystemInformation";
import About from "./LandingPage/About"; 

export default {
  name: "landing-page",
  components: { SystemInformation, About },
  data: () => ({
    logo: require("@/assets/pku.png"),
  }),

  created() {
    console.log(__lib);
  },
  methods: {
    openNewWin() {
      let data = {
        url: "/dashboard",
      };
      this.$ipcApi.send("open-win", data);
    }

  },
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Source Sans Pro", sans-serif;
}

#wrapper {
  padding: 60px 80px;
}

#logo {
  height: auto;
  margin-bottom: 20px;
  width: 650px;
}

main {
  display: flex;
  justify-content: space-between;
}

main > div {
  flex-basis: 50%;
}

.left-side {
  display: flex;
  flex-direction: column;
}

.welcome {
  color: #555;
  font-size: 23px;
  margin-bottom: 10px;
}

.title {
  color: #2c3e50;
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 6px;
}
.title-right {
  color: #1a2938;
  font-size: 25px;
  /* font-weight: bold; */
  font: 'Hiragino Sans GB';
  margin-bottom: 6px;
}

.title.alt {
  font-size: 18px;
  margin-bottom: 10px;
}
.doc {
  margin-bottom: 10px;
}
.doc .el-button {
  margin-top: 10px;
  margin-right: 10px;
}
.doc .el-button + .el-button {
  margin-left: 0;
}
</style>