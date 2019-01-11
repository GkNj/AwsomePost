<template>
  <div class="post">
    <el-row>
      <el-col :span="7" class="pos-order" id="order-list">
        <el-tabs style="margin-left: 10px">
          <el-tab-pane label=" 点餐">
            <el-table :data="tableData" border style="width:100%">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column label="数量" width="120">
                <template slot-scope="scope">
                  <el-input-number style="width:100px" v-model="scope.row.count" controls-position="right"
                                   @change="handleChange" :min="1" :max="10"></el-input-number>
                </template>

              </el-table-column>
              <el-table-column prop="price" label="金额" width="70"></el-table-column>
              <el-table-column label="操作" width="100" fixed="right">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="count">
              <small>数量:</small>
              {{totalCount}} &nbsp;&nbsp;&nbsp;&nbsp;
              <small>总额:</small>
              {{totalMoney}}元
            </div>
            <div class="div-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods">删除</el-button>
              <el-button type="success" @click="checkout">结账</el-button>
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
        <div>
          <div class="title">
            常用商品
          </div>
          <div class="often-goodsList">
            <ul>
              <li v-for="goods in oftenGoods" :key="goods.goodsId" @click="addOrderList(goods)">
                <span>{{goods.goodsName}}</span>
                <span class="o-price">￥{{goods.price}}元</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="type">
          <el-tabs style="margin-left: 10px">
            <el-tab-pane label="汉堡">
              <ul class='cookList'>
                <li v-for="goods in type0Goods" @click="addOrderList(goods)" :key="goods.goodsId">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="小食">
              <ul class='cookList'>
                <li v-for="goods in type1Goods" @click="addOrderList(goods)" :key="goods.goodsId">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <ul class='cookList'>
                <li v-for="goods in type2Goods" @click="addOrderList(goods)" :key="goods.goodsId">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <ul class='cookList'>
                <li v-for="goods in type3Goods" @click="addOrderList(goods)" :key="goods.goodsId">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
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
  import axios from 'axios';
  export default {
    name: 'post',
    data () {
      return {
        tableData: [],
        oftenGoods: [],
        type0Goods: [],
        type1Goods: [],
        type2Goods: [],
        type3Goods: [],
        totalCount: 0,
        totalMoney: 0

      }
    },
    created: function () {
      axios.get('https://www.easy-mock.com/mock/5b8b30dbf032f03c5e71de7f/kuaican/oftenGoods').then(response => {
        console.log(response)
        this.oftenGoods = response.data;
      }).catch(error => {
        alert('网络错误，无法获取数据')
      })

      axios.get('https://www.easy-mock.com/mock/5b8b30dbf032f03c5e71de7f/kuaican/typeGoods').then(response => {
        console.log(response)
        this.type0Goods = response.data[0];
        this.type1Goods = response.data[1];
        this.type2Goods = response.data[2];
        this.type3Goods = response.data[3];
      }).catch(error => {
        alert('网络错误，无法获取数据')
      })

    },
    mounted: function () {
      var orderheight = document.body.clientHeight;

      document.getElementById('order-list').style.height = orderheight + 'px';
    },
    methods: {
      addOrderList(goods){
        //判断商品是否存在订单列表中
        this.totalCount = 0;
        this.totalMoney = 0;
        let ishave = false;
        for (let i = 0; i < this.tableData.length; i++)
          if (this.tableData[i].goodsId == goods.goodsId) {
            ishave = true;
          }
        //根据结果写业务逻辑
        if (ishave) {
          let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
          arr[0].count++;
        } else {
          let newGoods = {goodsId: goods.goodsId, goodsName: goods.goodsName, price: goods.price, count: 1};
          this.tableData.push(newGoods)

        }

        this.getAllMoney();

      },
      //删除单个商品
      delSingleGoods(goods, index){
//          console.log(goods,index)
//        this.tableData.splice(index,1);
        this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId);
        console.log('tableData');
        this.getAllMoney();
      },
      //删除全部
      delAllGoods(){
        this.tableData = [];
        this.totalCount = 0;
        this.totalMoney = 0;
      },
      //汇总数量和金钱
      getAllMoney() {
        this.totalCount = 0;
        this.totalMoney = 0;
        if (this.tableData) {
          this.tableData.forEach((element) => {
            this.totalCount += element.count;
            this.totalMoney = this.totalMoney + (element.price * element.count);
          });
        }
      },

      //模拟结账
      checkout(){
        if (this.totalCount != 0) {
          this.tableData = [];
          this.totalCount = 0;
          this.totalMoney = 0;
          this.$message({
            message: '结账成功',
            type: 'success'
          })
        } else {
          this.$message({
            message: '不能空结',
            type: 'error'
          })
        }
      },

      handleChange(value) {
        this.getAllMoney()
      },
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .pos-order {
    background-color: #f9fafc;
    border-right: 1px solid #c0ccda;
    height: 100%;
  }

  .div-btn {
    margin-top: 10px;
  }

  .title {
    height: 20px;
    border-bottom: 1px solid #D3DCE6;
    background-color: #F9FAFC;
    padding: 10px;
    text-align: left;
  }

  .often-goodsList ul li {
    list-style: none;
    float: left;
    border: 1px solid #E5E9F2;
    padding: 10px;
    margin: 5px;
    background-color: #fff;
    cursor: pointer;
  }

  .o-price {
    color: #58b7ff;
  }

  .type {
    clear: both;
  }

  .cookList li {
    list-style: none;
    width: 23%;
    border: 1px solid #E5E9F2;
    height: auto;
    overflow: hidden;
    background-color: #fff;
    padding: 2px;
    float: left;
    margin: 2px;
    cursor: pointer;
  }

  .cookList li span {
    display: block;
    float: left;
  }

  .foodImg {
    width: 40%;
  }

  .foodName {
    font-size: 18px;
    padding-left: 10px;
    color: brown;
  }

  .foodPrice {
    font-size: 16px;
    padding-left: 10px;
    padding-top: 10px;
  }

  .count {
    background-color: #fff;
    padding: 10px;
    border-bottom: 1px solid #E5E9F2;
  }
</style>
