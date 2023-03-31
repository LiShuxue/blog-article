> 基于vue@2.7.14源码

## Vue 源码构建

1. 下载 vue2.7.14 源码或者 `git clone https://github.com/vuejs/vue.git`
2. 源码 .github/CONTRIBUTING.md 文件中有描述怎么安装依赖，使用 `pnpm install`，yarn 不行
3. 在 package.json 中的 scripts 中修改 dev 和添加 prod，主要是添加 `--sourcemap` 来输出 map 文件。另外 dev 和 prod 的区别就是打包的 vue 是否在控制台输出一些信息。
   ```bash
   "dev": "rollup -w -c scripts/config.js --environment TARGET:full-dev --sourcemap",
   "prod": "rollup -w -c scripts/config.js --environment TARGET:full-prod --sourcemap",
   ```
4. 在入口文件 src/platforms/web/entry-runtime-with-compiler.ts 中注释掉 v3 相关的东西，保留第一行和最后一行。
5. 执行 `pnpm run prod`，会在 dist 文件夹下生成 `vue.min.js` 和 `vue.min.js.map`，接下来就可以直接使用了。

## Vue 使用和调试

1. 在 examples/classic 文件夹下创建 debug 文件夹，并且在 debug 文件夹下创建 index.html
2. 将下面的代码复制到 index.html 中
3. 浏览器运行这个 html
4. 开始打断点调试

   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <title>debug</title>
       <script src="../../../dist/vue.min.js"></script>
     </head>
     <body>
       <div id="debug"></div>

       <script type="text/x-template" id="debug-template">
         <div>
           <p @click="changeData">Selected: {{ msg }} + {{msg2}}</p>
           <p v-for="(item, index) in arr" :key="index"> {{item}} </p>
           <child-comp />
         </div>
       </script>

       <script>
         const MyComp = Vue.component("child-comp", {
           template: "<p>{{text}}</p>",
           data: function () {
             return {
               text: "我是子组件",
             };
           },
           beforeCreate() {
             console.log("child beforeCreate");
           },
         });

         new Vue({
           el: "#debug",
           template: "#debug-template",
           data: {
             msg: "messasge",
             obj: {
               a: "111",
               test: {
                 test2: 222,
               },
             },
             arr: [1, 2, 3, 4],
           },
           computed: {
             msg2() {
               return this.msg + 2;
             },
           },
           watch: {
             msg(newVal, val) {
               console.log(newVal);
             },
           },
           beforeCreate() {
             console.log("beforeCreate");
           },

           created() {
             console.log("created");
           },

           beforeMount() {
             console.log("beforeMount");
           },

           mounted() {
             console.log("mounted");
           },

           methods: {
             changeData: function () {
               this.msg = Math.random();
               this.arr.splice(0, 1, this.msg);
             },
           },

           beforeUpdate() {
             console.log("beforeUpdate");
           },

           updated() {
             console.log("updated");
           },
         });
       </script>
     </body>
   </html>
   ```
