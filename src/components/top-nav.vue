<!--
  @name: 顶部菜单栏
  @author: fengyanlong
  @date:2021-07-29
-->

<template>
  <div class="top-nav">
    <ul>
      <li
        v-for="item in currentTopMenuList"
        :key="item.id"
        @click="jumpTo(item)"
        :class="item.id === currentTopMenuId ? 'active' : ''"
      >
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  name: "top-nav",
  data() {
    return {};
  },
  computed: {
    ...mapGetters("common", ["currentTopMenuList", "currentTopMenuId"]),
  },
  watch: {
    "$route.path": {
      handler(value) {
        //匹配高亮菜单
        this.findActiveMenu(value);
      },
      immediate: true,
    },
  },
  created() {},
  methods: {
    jumpTo(row) {
      if (row.url === "/") {
        this.$router.push("/");
      } else {
        this.$router.push(row.children[0].children[0].url);
      }
    },
    async findActiveMenu(path) {
      //由于第一次访问时，顶部菜单列表是异步的，需要等待结果才能进行计算
      let topMenuList = await this.checkCurrentTopMenuList();
      //首页特殊处理
      if (path === "/") {
        let target = topMenuList.filter((item) => {
          return item.url === "/";
        });
        if (target && target.length) {
          this.$store.commit("common/setCurrentRouterTreeLink", [target[0].id]);
          this.$store.commit("common/setCurrentTopMenuId", target[0].id);
        }
        return;
      }
      console.log(topMenuList);
      //记录当前路由在整个树节点里的链路
      let treeLink = [];
      /**
       * @param tree 树结构菜单
       * @param func 判断找到节点的条件
       * @description 在树结构里找节点
       */
      function treeFind(tree, func) {
        for (const data of tree) {
          if (func(data)) {
            treeLink.push(data); //把最终节点加入链路
            return data;
          }
          if (data.children && data.children.length) {
            const res = treeFind(data.children, func);
            if (res) {
              treeLink.push(data); //把每个结果的父节点加入链路
              return res;
            }
          }
        }
        return null;
      }
      for (let k = 0; k < topMenuList.length; k++) {
        let output = treeFind(topMenuList[k].children, function (data) {
          return data.url === path;
        });
        if (output) {
          //如果找到了这个树下拥有这个节点，则拿到顶级节点的路由id
          this.$store.commit("common/setCurrentTopMenuId", topMenuList[k].id);
          //把顶级也加到链路
          treeLink.push(topMenuList[k]);
          treeLink = treeLink.map(({ id, name, level, url }) => {
            return { id, name, level, url };
          });
          //console.log(treeLink);
          this.$store.commit("common/setCurrentRouterTreeLink", treeLink);
          break;
        }
      }

      //处理从登录页面进来的首次跳转
      if (this.$route.query.from === "selectPage") {
        let firstGoPage = "/";
        let level2 = this.currentTopMenuList[0]?.children;
        if (level2 && level2.length) {
          let level3 = level2[0]?.children;
          //console.log(level3);
          if (level3 && level3.length) {
            //找到第一个3级菜单的第一个rul
            if (level3[0]?.url) firstGoPage = level3[0].url;
          }
        }
        this.$router.push(firstGoPage);
      }
    },
    checkCurrentTopMenuList() {
      let that = this;
      return new Promise(function (resolve) {
        let timeOut = 0;
        const loopFind = () => {
          setTimeout(() => {
            timeOut++;
            if (timeOut > 10) {
              resolve([]);
            }
            if (that.currentTopMenuList.length === 0) {
              loopFind();
            } else {
              //等找到了顶部列表，才结束循环，返回出去
              resolve(that.currentTopMenuList);
            }
          }, 100);
        };
        loopFind();
      });
    },
  },
};
</script>

<style scoped>
.top-nav {
  padding: 10px 0;
  border-bottom: 1px solid #ccc;
}

.top-nav ul li {
  list-style: none;
  display: inline-block;
  margin: 5px 15px;
  cursor: pointer;
}
.top-nav ul li:hover {
  color: #14b314;
}
.top-nav ul li.active {
  color: #14b314;
}
</style>
