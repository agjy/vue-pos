<template>
  <div class="pos">
    <el-row>
      <el-col :span="7" class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border style="width:=100%">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="price" label="金额" width="50"></el-table-column>
              <el-table-column prop="count" label="数量" width="70"></el-table-column>
              <el-table-column prop="operation" label="操作" width="100" fixed="right">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                </template>
              </el-table-column>

            </el-table>
            <div class="goods-total">
              <small>数量：</small>{{totalCount}}
              <small>金额：</small>{{totalMoney}}元
            </div>
            <div class="div-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods">删除</el-button>
              <el-button type="success" @click="checkOut">结账</el-button>
            </div>
            </el-tab-pane>
            <el-tab-pane label="挂单">
              挂单
            </el-tab-pane>
            <el-tab-pane label="外卖">
              外卖
            </el-tab-pane>

        </el-tabs>
      </el-col>
      <el-col :span="17">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul>
              <li v-for="(goods, key) in oftenGoods" :key="key" @click="addOrderList(goods)">
                <span>{{goods.goodsName}}</span>
                <span class="o-price">￥{{goods.price}}元</span>
              </li>
            </ul>
          </div>
        </div>
        <div class="goods-type">
          <el-tabs>
            <el-tab-pane label="汉堡">
              <ul class='cookList'>
                  <li v-for="(goods, key) in type0Goods" :key="key" @click="addOrderList(goods)">
                      <span class="foodImg">
                          <img :src="goods.goodsImg" width="100%">
                      </span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="小吃">
              <ul class='cookList'>
                  <li v-for="(goods, key) in type1Goods" :key="key" @click="addOrderList(goods)">
                      <span class="foodImg">
                          <img :src="goods.goodsImg" width="100%">
                      </span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <ul class='cookList'>
                  <li v-for="(goods, key) in type2Goods" :key="key" @click="addOrderList(goods)">
                      <span class="foodImg">
                          <img :src="goods.goodsImg" width="100%">
                      </span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <ul class='cookList'>
                  <li v-for="(goods, key) in type3Goods" :key="key" @click="addOrderList(goods)">
                      <span class="foodImg">
                          <img :src="goods.goodsImg" width="100%">
                      </span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
              </ul>
            </el-tab-pane>
          </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>

import axios from 'axios'

export default {
  name: 'Pos',
  mounted: () => {
    var orderHeight = document.body.clientHeight;
    document.getElementById('order-list').style.height = orderHeight + 'px';
  },
  created() {
    //读取常用商品列表
    axios.get('https://www.easy-mock.com/mock/5b8b30dbf032f03c5e71de7f/kuaican/oftenGoods')
      .then(res => {
        console.log(res)
        this.oftenGoods = res.data;
      })
      .catch(error => {
         console.log(error);
         alert('网络错误，不能访问');
      })
      axios.get('https://www.easy-mock.com/mock/5b8b30dbf032f03c5e71de7f/kuaican/typeGoods')
      .then(res => {
        console.log(res)
        this.type0Goods = res.data[0];
        this.type1Goods = res.data[1];
        this.type2Goods = res.data[2];
        this.type3Goods = res.data[3];
      })
      .catch(error => {
         console.log(error);
         alert('网络错误，不能访问');
      })
  },
  data() {
    return {
      tableData: [],
      oftenGoods:[],
      type0Goods:[],
      type1Goods:[],
      type2Goods:[],
      type3Goods:[],
      totalCount: 0,
      totalMoney: 0
    }
  },
  methods: {
    addOrderList(goods) {
      // 商品是否已经存在
      let isExist = false;
      for (let i = 0; i < this.tableData.length; i++) {
        if (this.tableData[i].goodsId == goods.goodsId) {
          isExist = true;
        }
      }

      // 根据判断的值编写业务逻辑
      if (isExist) {
        let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
        arr[0].count++;
      } else {
        let newGoods = {
          goodsName: goods.goodsName,
          goodsId: goods.goodsId,
          price: goods.price,
          count: 1
        }
        this.tableData.push(newGoods);
      }
      this.getTotal();
    },

    // 计算汇总数量和金额
    getTotal() {
      this.totalCount = 0;
      this.totalMoney = 0;
      if (this.tableData) {
        this.tableData.forEach((element) => {
          this.totalCount += element.count;
          this.totalMoney = this.totalMoney + (element.price * element.count);
        });
      }

    },
    delSingleGoods(goods) {
      if (goods.count > 1) {
        goods.count --;
      } else {
        this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId);
      }
      this.getTotal();
    },
    delAllGoods() {
      this.tableData = [];
      this.totalCount = 0;
      this.totalMoney = 0;
    },
    // 模拟结账
    checkOut() {
      if (this.totalCount != 0) {
        this.tableData = [];
        this.totalCount = 0;
        this.totalMoney = 0;
        this.$message({
          message: '结账成功！',
          type: 'success'
        })
      } else {
        this.$message.error('结账失败，暂无商品！')
      }

    }
  },
}
</script>

<style scoped>
  .pos-order {
    background-color: #F9FAFC;
    border-right: 1px solid #C0CCDA;
    height: 100%;
  }
  .div-btn {
    margin-top: 10px;
  }
  .goods-total {
    padding: 10px;
    background-color: #fff;
    border-bottom: 1px solid #E5E9F2;
  }
  .goods-total small {
    margin: 0 5px;
  }
  .title {
    height: 20px;
    border-bottom: 1px solid #D3DCE6;
    background-color: #F9FAFC;
    padding: 10px;
  }
  .often-goods-list ul li{
    list-style: none;
    float:left;
    border:1px solid #E5E9F2;
    padding:10px;
    margin:5px;
    background-color:#fff;
    cursor: pointer;
  }
  .goods-type {
    clear: both;
  }
  .o-price{
      color:#58B7FF;
  }
  .cookList li {
    list-style: none;
    width: 23%;
    float: left;
    border: 1px solid #E5E9F2;
    padding: 2px;
    margin: 2px;
    background-color: #fff;
    overflow: hidden;
    position: relative;
    cursor: pointer;
  }
  .cookList li span {
    display: block;
    float:left;
  }
  .foodImg{
    width: 40%;
  }
  .foodName{
    font-size: 18px;
    padding-left: 10px;
    color:brown;
  }
  .foodPrice{
    position: absolute;
    color: #58B7FF;
    font-size: 18px;
    top: 30px;
    left: 105px;
  }
</style>
