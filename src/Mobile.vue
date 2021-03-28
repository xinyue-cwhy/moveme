<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
import StyleEditor from "./components/StyleEditor";
import ResumeEditor from "./components/ResumeEditor";
import "./assets/reset.css";
export default {
  name: "app",
  components: {
    StyleEditor,
    ResumeEditor
  },
  data() {
    return {
      interval: 3,
      currentStyle: "",
      enableHtml: false,
      fullStyle: [
        `

/* 来点过渡 */
* {
  transition: all .1s;
}
/* 来点背景 */
html {
  color: rgb(222,222,222);
  background: linear-gradient(right, #d4fc78 , #99ESA2);
}
/* 来点内边距 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  overflow: auto;
  width: 90vw;
  margin: 2.5vh 5vw;
  height: 90vh;
}
/* 太高了 */
.styleEditor {
  height: 45vh;
}
/* 代码高亮 */
.token.selector{
  color: rgb(133,153,0);
}
.token.property{
  color: rgb(187,137,0);
}
.token.punctuation{
  color: yellow;
}
.token.function{
  color: rgb(42,161,152);
}
/* 来点3D */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  transform: rotateX(-10deg) translateZ(-50px) ;
}
/* 接下来准备一个编辑器 */
.resumeEditor{
  position: fixed;
  top: 50%; left: 0;
  padding: .5em;  margin: 2.5vh;
  width: 95vw; height: 45vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 开始写简历 */
`,
        `
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`
      ],
      currentMarkdown: "",
      fullMarkdown: `
辛跃
----
22岁，意向前端工程师。
  独立完成过小型项目，前端操作数据库；
	教务系统项目，从开始到完成，极大地加强了我的逻辑思考能力；
	具备艺术家的灵感创作能力；
	专业知识的学习以及多年的实践经验，使我积累了丰的项目经验，并取得了优秀的专业课成绩

技能
----
* 全日制本科学历
* 熟练使用HTML5 , Javascript , CSS3 ，ES6
* 熟练使用photoshop 切图
* 了解node.js Java Andriod c#
* 了解Git 分布式版本控制系统的使用
* 熟悉npm包管理，了解mysql数据库
* 以上提到过的均在Github中有例子，所遇到的问题解决方法均在CSDN上有记录

技术及语言
----
  - **前端**: VueJs、Bootstrap、ElementUI
  - **数据库**: MySQL
  - **web 服务器**: Nginx、Tomcat
  - **OS**: Windows
  - **Others**: Git、webpack、IDEA


开源项目
----
1. [基于vue.js 的电商移动端项目](https://github.com/xinyue-cwhy/supermall)
2. [基于vue.js 的教务管理系统](https://github.com/xinyue-cwhy/educational)
3. [基于vue.js + ElementUI的电商移动端项目]
4. [基于c#对数据库的增删改查]
5. [一个会动的简历](https://github.com/xinyue-cwhy/moveme)
6. [辛跃的技术博客网站](https://github.com/xinyue-cwhy/xinyue-cwhy.github.io)
链接
----
* [技术博客](https://github.com/xinyue-cwhy/xinyue-cwhy.github.io)
* [GitHub](https://github.com/xinyue-cwhy/)
* [CSDN](https://blog.csdn.net/qq_47735503)

联系我
----
* 联系QQ：**2644965101** | 微信：**15526645761**
* 主要涉及技术：**前端开发****开源爱好者**

`
    };
  },
  created() {
    this.makeResume();
  },
  methods: {
    makeResume: async function() {
      await this.progressivelyShowStyle(0);
      await this.progressivelyShowResume();
      await this.progressivelyShowStyle(1);
      await this.showHtml();
      await this.progressivelyShowStyle(2);
    },
    showHtml: function() {
      return new Promise((resolve, reject) => {
        this.enableHtml = true;
        this.$nextTick(() => {
          this.$refs.resumeEditor.goTop();
        });
        resolve();
      });
    },
    progressivelyShowStyle(n) {
      return new Promise((resolve, reject) => {
        let interval = this.interval;
        let showStyle = async function() {
          let style = this.fullStyle[n];
          if (!style) {
            return;
          }
          // 计算前 n 个 style 的字符总数
          let length = this.fullStyle
            .filter((_, index) => index <= n)
            .map(item => item.length)
            .reduce((p, c) => p + c, 0);
          let prefixLength = length - style.length;
          if (this.currentStyle.length < length) {
            let l = this.currentStyle.length - prefixLength;
            let char = style.substring(l, l + 1) || " ";
            this.currentStyle += char;
            if (style.substring(l - 1, l) === "\n" && this.$refs.styleEditor) {
              this.$nextTick(() => {
                this.$refs.styleEditor.goBottom();
              });
            }
            setTimeout(showStyle, interval);
          } else {
            resolve();
          }
        }.bind(this);
        showStyle();
      });
    },
    progressivelyShowResume() {
      return new Promise((resolve, reject) => {
        let length = this.fullMarkdown.length;
        let interval = this.interval;
        let showResume = () => {
          if (this.currentMarkdown.length < length) {
            this.currentMarkdown = this.fullMarkdown.substring(
              0,
              this.currentMarkdown.length + 1
            );
            let lastChar = this.currentMarkdown[
              this.currentMarkdown.length - 1
            ];
            let prevChar = this.currentMarkdown[
              this.currentMarkdown.length - 2
            ];
            if (prevChar === "\n" && this.$refs.resumeEditor) {
              this.$nextTick(() => this.$refs.resumeEditor.goBottom());
            }
            setTimeout(showResume, interval);
          } else {
            resolve();
          }
        };
        showResume();
      });
    }
  }
};
</script>

<style scoped>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  min-height: 100vh;
  position: relative;
}
html {
  min-height: 100vh;
}
* {
  box-sizing: border-box;
}
</style>
