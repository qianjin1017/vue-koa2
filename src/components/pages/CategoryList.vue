<template>
  <div>
    <div class="navbar-div">
      <van-nav-bar title="类别列表" />
    </div>
    <div>
      <van-row>
        <van-col span="6">
          <div id="leftNav">
            <ul>
              <li v-for="(item,index) in category" :key="index" @click="clickCategory(index,item.ID)" :class="categoryIndex==index?'categoryActive':''">
                {{item.MALL_CATEGORY_NAME}}
              </li>
            </ul>
          </div>
        </van-col>
        <van-col span="18">
          <div class="tabCategorySub">
            <van-tabs v-model="active">
              <van-tab v-for="(item, index) in categorySub" :key="index" :title="item.MALL_SUB_NAME">
              </van-tab>
            </van-tabs>
          </div>
          <div id="list-div">
            <van-pull-refresh v-model="isRefresh" @refresh="onRefresh">
              <van-list v-model="loading" :finished="finished" @load="onLoad">
                <div class="list-item" v-for="(item,index) in goodList" :key="index">
                  <div class="list-item-img"><img :src="item.IMAGE1" width="100%" :onerror="errorImg"/></div>
                  <div class="list-item-text">
                    <div>{{item.NAME}}</div>
                    <div class="">￥{{item.ORI_PRICE}}</div>
                  </div>
                </div>
              </van-list>
            </van-pull-refresh>
          </div>
        </van-col>
      </van-row>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import url from "@/serviceAPI.config.js";
import { Toast } from "vant";
export default {
  data() {
    return {
      category: [],
      categoryIndex: 0,
      categorySub: [],
      active: 0,
      list: [],
      loading: false, //上拉加载使用
      finished: false, //下拉加载是否没有数据了
      isRefresh: false, //下拉加载
      page: 1, //商品列表的页数
      goodList: [], //商品信息
      categorySubId: "" ,//商品子分类ID
      errorImg:'this.src="' + require('@/assets/images/errorimg.png') + '"',  //错误图片显示路径
    };
  },
  created() {
    this.getCategory();
  },
  mounted() {
    let winHeight = document.documentElement.clientHeight;
    document.getElementById("leftNav").style.height = winHeight - 46 + "px";
    document.getElementById("list-div").style.height = winHeight - 90 + "px";
  },
  methods: {
    //点击获取大类
    clickCategory(index, categoryId) {
      this.categoryIndex = index;
      this.page = 1;
      this.finished = false;
      this.goodList = [];
      this.getCategorySubByCategoryId(categoryId);
    },
    //点击子类获取商品信息
    onClickCategorySub(index, title) {
      //console.log( this.categorySub)
      this.categorySubId = this.categorySub[index].ID;
      console.log(this.categorySubId);
      this.goodList = [];
      this.finished = false;
      this.page = 1;
      this.onLoad();
    },
    onLoad() {
      console.log("xxxxxxxxxxxx")
      setTimeout(() => {
        this.categorySubId = this.categorySubId
          ? this.categorySubId
          : this.categorySub[0].ID;
        this.getGoodList();
      }, 500);
    },
    onRefresh() {
      setTimeout(() => {
        this.isRefresh = false;
        this.finished = false;
        this.goodList = [];
        this.page = 1;
        this.getGoodList();
      }, 500);
    },
    getCategory() {
      axios({
        url: url.getCateGoryList,
        method: "get"
      })
        .then(res => {
          if (res.data.code === 200 && res.data.message) {
            // console.log(res.data.message);
            this.category = res.data.message;
            this.getCategorySubByCategoryId(this.category[0].ID);
          } else {
            Toast("服务器错误，数据取得失败");
          }
        })
        .catch(error => {
          Toast("服务器错误，数据取得失败");
        });
    },
    getGoodList() {
      axios({
        url: url.getGoodsListByCategorySubID,
        method: "post",
        data: {
          categorySubId: this.categorySubId,
          page: this.page
        }
      })
        .then(response => {
          // console.log(response);
          if (response.data.code == 200 && response.data.message.length) {
            this.page++;
            this.goodList = this.goodList.concat(response.data.message);
          } else {
            this.finished = true;
          }
          this.loading = false;
          console.log(this.finished);
        })
        .catch(error => {
          console.log(error);
        });
    },
    //获取子类
    getCategorySubByCategoryId(categoryId) {
      axios({
        url: url.getCateGorySubList,
        method: "post",
        data: { categoryId: categoryId }
      })
        .then(res => {
          //console.log(res.data);
          if (res.data.code === 200 && res.data.message) {
            this.categorySub = res.data.message;
            this.active = 0;
          } else {
            Toast("服务器错误，数据取得失败");
          }
        })
        .catch(error => {
          Toast("服务器错误，数据取得失败");
        });
    }
  }
};
</script>

<style scoped>
#leftNav {
  background-color: aliceblue;
}
#leftNav ul li {
  line-height: 2rem;
  border-bottom: 1px solid #e4e7ed;
  padding: 3px;
  font-size: 0.8rem;
  text-align: center;
}
.categoryActice {
  background-color: #fff;
}

.categoryActive {
  background-color: #fff;
}
.list-item {
  display: flex;
  flex-direction: row;
  font-size: 0.8rem;
  border-bottom: 1px solid #f0f0f0;
  background-color: #fff;
  padding: 5px;
}
#list-div {
  overflow: scroll;
}
.list-item-img {
  flex: 8;
}
.list-item-text {
  flex: 16;
  margin-top: 10px;
  margin-left: 10px;
}
</style>