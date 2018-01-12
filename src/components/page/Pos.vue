<template>
  <div class="pos">
    <el-row>
      <el-col :span='7' class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tabledata" border style="width:100%;">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="count" label="数量" width="50"></el-table-column>
              <el-table-column prop="price" label="金额" width="70"></el-table-column>
              <el-table-column label="操作" width="100" fixed="right">
                <template scope="scope">
                  <el-button type="text" size="small" @click="deleteSingleGood(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderlist(scope.row)">添加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="totaldiv">
             <small>数量</small> ：{{ totalCount }}   &nbsp;&nbsp;&nbsp;&nbsp;  <small>金额</small>：{{ totalMoney }}元
            </div>
            <div class="div-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="deleteAllGoods">删除</el-button>
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
      <el-col :span='17'>
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul>
              <li v-for="goods in oftenGoods" @click="addOrderlist(goods)">
                <span>{{ goods.goodsName }}</span>
                <span class="o-price">￥{{ goods.price }}元</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="goods-type">
          <el-tabs>
            <el-tab-pane label="汉堡">
              <div>
                <ul class='cookList'>
                    <li v-for="goods in type0Goods" @click="addOrderlist(goods)">
                        <span class="foodImg"><img :src="goods.goodsImg"  width="100%"></span>
                        <span class="foodName">{{ goods.goodsName }}</span>
                        <span class="foodPrice">￥{{ goods.price }}元</span>
                    </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="小食">
              <div>
                <ul class='cookList'>
                    <li v-for="goods in type1Goods" @click="addOrderlist(goods)">
                        <span class="foodImg"><img :src="goods.goodsImg"  width="100%"></span>
                        <span class="foodName">{{ goods.goodsName }}</span>
                        <span class="foodPrice">￥{{ goods.price }}元</span>
                    </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <div>
                <ul class='cookList'>
                    <li v-for="goods in type2Goods" @click="addOrderlist(goods)">
                        <span class="foodImg"><img :src="goods.goodsImg"  width="100%"></span>
                        <span class="foodName">{{ goods.goodsName }}</span>
                        <span class="foodPrice">￥{{ goods.price }}元</span>
                    </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <div>
                <ul class='cookList'>
                    <li v-for="goods in type3Goods" @click="addOrderlist(goods)">
                        <span class="foodImg"><img :src="goods.goodsImg"  width="100%"></span>
                        <span class="foodName">{{ goods.goodsName }}</span>
                        <span class="foodPrice">￥{{ goods.price }}元</span>
                    </li>
                </ul>
              </div>
            </el-tab-pane>
          </el-tabs>
        </div>

      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "pos",
  data() {
    return {
      tabledata: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0
    };
  },
  created: function() {
    axios
      .get("http://jspang.com/DemoApi/oftenGoods.php")
      .then(res => {
        if (res.status != 200) {
          //  err

          return;
        }
        this.oftenGoods = res.data;
      })
      .catch(err => {
        //  数据出错！
        console.log(err);
        alert("网络不通");
      });

    axios
      .get("http://jspang.com/DemoApi/typeGoods.php")
      .then(res => {
        if (res.status != 200) {
          //  err

          return;
        }
        this.type0Goods = res.data[0];
        this.type1Goods = res.data[1];
        this.type2Goods = res.data[2];
        this.type3Goods = res.data[3];
      })
      .catch(err => {
        alert("网络不通");
      });
  },
  mounted: function() {
    var orderHeight = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderHeight + "px";
  },
  methods: {
    // 添加商品
    addOrderlist(goods) {
      this.totalMoney = 0;
      this.totalCount = 0;

      //  商品是否已经存在订单列表中
      let isHave = false;
      for (let i = 0; i < this.tabledata.length; i++) {
        if (this.tabledata[i].goodsId == goods.goodsId) {
          isHave = true;
        }
      }

      //  根据判断的值编写业务逻辑
      if (isHave) {
        //  改变列表中商品数量
        let arr = this.tabledata.filter(o => o.goodsId == goods.goodsId);
        arr[0].count++;
      } else {
        let newGoods = {
          goodsId: goods.goodsId,
          goodsName: goods.goodsName,
          price: goods.price,
          count: 1
        };
        this.tabledata.push(newGoods);
      }
      // 计算汇总金额
       this.getAllMoney();
    },
    // 删除单个商品
    deleteSingleGood(goods) {
      this.tabledata = this.tabledata.filter(o => o.goodsId != goods.goodsId);
      this.getAllMoney();
    },
    // 删除全部
    deleteAllGoods(){
      this.tabledata = [];
      this.totalMoney = 0;
      this.totalCount = 0;
    },
    // 汇总数量和金额
    getAllMoney() {
      this.totalCount = 0;
      this.totalMoney = 0;

      if (this.tabledata) {
        this.tabledata.forEach(element => {
          this.totalCount += element.count;
          this.totalMoney = this.totalMoney + element.price * element.count;
        });
      }
    },
    // 模拟结账
    checkOut(){
      if(this.totalCount != 0){
        this.tabledata = [];
        this.totalCount = 0;
        this.totalCount = 0;
        this.$message({
          message:'结账成功！感谢你又为店里创造价值！',
          type:"success"
        })
      }else {
        const h = this.$createElement;
        this.$message({
          message:"没有选择商品，不要着急，我们明白你急切的心情！",
          type:"warning"
        })
      }
    }
  }
};
</script>

<style scoped>
.pos-order {
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
}
.div-btn {
  margin-top: 20px;
}
.title {
  height: 20px;
  border-bottom: 1px solid #d3dce6;
  background-color: #f9fafc;
  padding: 10px;
  text-align: left;
}
.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #e5e9f2;
  padding: 10px;
  margin: 10px;
  background-color: #ffffff;
  cursor: pointer;
}
.o-price {
  color: rgb(70, 136, 243);
}
.goods-type {
  clear: both;
}
.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #e5e9f2;
  height: auot;
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
  font-size: 12px;
  padding-left: 10px;
  color: brown;
}
.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}
.totaldiv {
  background-color: #fff;
  padding: 10px;
  border-bottom: 1px solid #d3dce6;
}
</style>
