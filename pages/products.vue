<template>
  <el-row class="page-product">
    <el-col :span="19">
      <crumbs :keyword="keyword"/>
      <categroy :types="types" :areas="areas"/>
      <list :list="list"/>
    </el-col>
    <el-col :span="5">
      <amap
        v-if="point.length"
        :width="230"
        :height="290"
      :point="point"/>
    </el-col>
  </el-row>
</template>

<script>
import Crumbs from "@/components/products/crumbs.vue";
import Categroy from "@/components/products/categroy.vue";
import List from "@/components/products/list.vue";
import Amap from "@/components/public/map.vue";
const axios = require("../server/interface/utils/axios");
export default {
  layout: "default",
  components: {
    Crumbs,
    Categroy,
    List,
    Amap
  },
  computed: {
    position: state => state.position
  },
  data() {
    return {
      list: [],
      types: [],
      areas: [],
      keyword: "",
      point: []
    };
  },
  // 通过SSR获取异步数据,改方法在服务端调用，无法获取this,但是 vuex 是在服务端和客户端共享的
  async asyncData(ctx) {
    let keyword = ctx.query.keyword;
    let city = ctx.store.state.geo.position.city;
    let {
      status: status1,
      data: { count, pois }
    } = await axios.get("/search/resultsByKeywords", {
      params: {
        keyword,
        city
      }
    });
    
    let {
      status: status2,
      data: { areas, types }
    } = await axios.get("/categroy/crumbs", {
      params: {
        city
      }
    });
    if (status1 === 200 && count > 0 && status2 === 200) {
      return {
        list: pois
          .filter(item => item.photos.length)
          .map(item => {
            return {
              type: item.type,
              img: item.photos[0].url,
              name: item.name,
              comment: Math.floor(Math.random() * 10000),
              rate: Number(item.biz_ext.rating),
              price: Number(item.biz_ext.cost),
              scene: item.tag,
              tel: item.tel,
              status: "可订明日",
              location: item.location,
              module: item.type.split(";")[0]
            };
          }),
        keyword,
        areas: areas.filter(item => item.type !== "").slice(0, 5),
        types: types.filter(item => item.type !== "").slice(0, 5),
        point: (pois.find(item => item.location).location || "").split(",")
      };
    }
  }
};
</script>

<style lang="scss">
@import "@/assets/css/products/index.scss";
</style>
