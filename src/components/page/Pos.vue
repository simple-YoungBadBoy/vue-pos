<template>
    <div class="pos">
        <el-row>
            <el-col :span="7" class="pos-order" id="order-list">
                <el-tabs>
                    <el-tab-pane label="点餐">
                        <el-table border :data="tableData" style="width:100%;">
                            <el-table-column prop="goodsName" label="商品名称" width="100">

                            </el-table-column>
                            <el-table-column prop="count" label="数量" width="80">

                            </el-table-column>
                            <el-table-column prop="price" label="金额" width="90">

                            </el-table-column>
                            <el-table-column fixed="right" label="操作" width="105">
                                <template scope="scope">
                                    <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                                    <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                    </el-tab-pane>
                    <el-tab-pane label="挂单">
                        挂单
                    </el-tab-pane>
                    <el-tab-pane label="外卖">
                        外卖
                    </el-tab-pane>
                </el-tabs>
                <div class="o-priceCount">
                    <small>数量：</small>{{totalCount}}&emsp;
                    <small>金额：</small>{{totalMoney}}
                </div>
                <!-- 按钮 -->
                <div class="button">
                    <el-button type="warning">挂单</el-button>
                    <el-button type="danger" @click="delAllGoods">删除</el-button>
                    <el-button type="success " @click="checkout">结账</el-button>
                </div>

            </el-col>
            <!-- 右边 -->
            <el-col :span="17">
                <div class="often-goods">
                    <div class="title">常用商品</div>
                    <div class="ofen-goodList">
                        <ul>
                            <li v-for="items in oftenGoods" @click="addOrderList(items)">
                                <span>{{items.goodsName}}</span>
                                <span class="o-price">￥{{items.price}}</span>
                            </li>
                        </ul>
                    </div>
                </div>

                <div class="goods-type">
                    <el-tabs>
                        <el-tab-pane label="汉堡">
                            <div>
                                <ul class="cookList">
                                    <li v-for="items in type0Goods" @click="addOrderList(items)">
                                        <span class="foodImg"><img :src="items.goodsImg" width="100%" /></span>
                                        <span class="foodName">{{items.goodsName}}</span>
                                        <span class="foodPrice">￥{{items.price}}</span>
                                    </li>
                                </ul>
                            </div>
                        </el-tab-pane>
                        <el-tab-pane label="小食">
                            <ul class="cookList">
                                <li v-for="items in type1Goods" @click="addOrderList(items)">
                                    <span class="foodImg"><img :src="items.goodsImg" width="100%" /></span>
                                    <span class="foodName">{{items.goodsName}}</span>
                                    <span class="foodPrice">￥{{items.price}}</span>
                                </li>
                            </ul>
                        </el-tab-pane>
                        <el-tab-pane label="饮料">
                            <ul class="cookList">
                                <li v-for="items in type2Goods" @click="addOrderList(items)">
                                    <span class="foodImg"><img :src="items.goodsImg" width="100%" /></span>
                                    <span class="foodName">{{items.goodsName}}</span>
                                    <span class="foodPrice">￥{{items.price}}</span>
                                </li>
                            </ul>
                        </el-tab-pane>
                        <el-tab-pane label="套餐">
                            <ul class="cookList">
                                <li v-for="items in type3Goods" @click="addOrderList(items)">
                                    <span class="foodImg"><img :src="items.goodsImg" width="100%" /></span>
                                    <span class="foodName">{{items.goodsName}}</span>
                                    <span class="foodPrice">￥{{items.price}}</span>
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
    data() {
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
    mounted: function() { //为order-list添加高度
        var orderHeight = document.body.clientHeight
        document.getElementById('order-list').style.height = orderHeight + 'px';
    },
    created: function() {
        axios.get('http://jspang.com/DemoApi/oftenGoods.php')
            //成功
            .then(reponse => {
                //console.log(reponse)
                this.oftenGoods = reponse.data;
            })
            //失败
            .catch(error => {
                console.log(error)
                alert('网络错误，不能访问.')
            })

        axios.get('http://jspang.com/DemoApi/typeGoods.php')
            .then(reponse => {
                console.log(reponse.data)
                this.type0Goods = reponse.data[0];
                this.type1Goods = reponse.data[1];
                this.type2Goods = reponse.data[2];
                this.type3Goods = reponse.data[3];
            })
            .catch()
    },
    methods: {
        addOrderList(goods) {
            //1.商品是否已经存在于订单列表中
            let isHave = false;
            for (let i = 0; i < this.tableData.length; i++) {
                if (this.tableData[i].goodsId == goods.goodsId) {
                    isHave = true
                }
            }
            //如果有就加1
            if (isHave) {
                //存在就数量添加
                let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
                arr[0].count++;
            } else {
                //如果没有就添加商品
                let newGoods = {
                    goodsId: goods.goodsId,
                    goodsName: goods.goodsName,
                    price: goods.price,
                    count: 1
                }
                this.tableData.push(newGoods);
            }
            this.getAllMoney();

        },
        // 删除单个商品
        delSingleGoods(goods) {
            this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId)
            this.getAllMoney();
        },
        //删除全部商品
        delAllGoods(goods){
            this.tableData = this.tableData.filter(o => o.goodsId != o.goodsId)
            this.getAllMoney();
        },
        //模拟结账
        checkout(){
            if(this.totalCount != 0){
                this.tableData=[];
                this.$message({
                    message:'结账成功！本次消费'+this.totalMoney+'元，欢迎下次光临！',
                    type:'success',
                })
                this.totalMoney = 0;
                this.totalCount = 0;
            }else{
                this.$message.error('请先添加购物车！')
            }
        },
        //计算汇总金额和数量
        getAllMoney() {
            this.totalMoney = 0;
            this.totalCount = 0;
            if (this.tableData) {
                this.tableData.forEach((element) => {
                    this.totalCount += element.count;
                    this.totalMoney = this.totalMoney + (element.price * element.count)
                })
            }
        }

    }
}
</script>

<style scoped>
.pos-order {
    background: #f9fafc;
    border-right: 1px solid #c0ccda;
}

.button {
    margin-top: 10px;
}

.title {
    text-align: left;
    height: 41px;
    line-height: 41px;
    padding-left: 10px;
    border-bottom: 1px solid #d3dce6;
    background: #F9FAFC;
}

.often-goods {
    overflow: hidden;
}

.ofen-goodList ul {
    padding: 0;
    margin: 0;
}

.ofen-goodList ul li {
    list-style: none;
    float: left;
    border: 1px solid #e5e9f2;
    padding: 10px;
    margin: 10px;
    background: white;
}

.o-price {
    color: #5887ff;
}

.cookList li {
    list-style: none;
    width: 23%;
    border: 1px solid #e5e9f2;
    height: auto;
    overflow: hidden;
    float: left;
    margin: 2px;
    padding: 2px;
}

.cookList li span {
    float: left;
    display: block;
}

.foodImg {
    width: 40%;
}

.cookList .foodName {
    color: brown;
    padding: 0 10px;
    font-size: 16px;
}

.cookList .foodPrice {
    padding: 10px;
}

.o-priceCount {
    margin-top: 10px;
}

@media screen and (min-width:1628px) {
    .cookList .foodPrice {
        padding: 10px 10%;
    }
    .cookList .foodName {
        padding: 0 10%;
    }
}
</style>
