<!-- strategy 策略选择 -->
<template>
    <div class='strategy'>
        <div class="wrapper">
            <!-- 基本信息 -->
            <div class="wrapper-item">
                <h4>基本信息：</h4>
                <div class="wrapper-item-content basic">
                    <div class="wrapper-item-content-list"><span>初始资金：</span><Input readonly size="small" v-model="initialFunding" placeholder="初始资金" style="width: 120px" /></div>
                    <div class="wrapper-item-content-list"><span>开始时间：</span>
                        <Date-picker size="small" v-model="startTime" type="date" placeholder="开始时间" :options="startDataOption" style="width: 120px"></Date-picker>
                    </div>
                    <div class="wrapper-item-content-list"><span>结束时间：</span>
                        <Date-picker size="small" v-model="endTime" type="date" placeholder="开始时间" :options="startDataOption" style="width: 120px"></Date-picker>
                    </div>
                </div>
            </div>
            <!-- 股池 -->
            <div class="wrapper-item">
                <h4>股池：</h4>
                <div class="wrapper-item-content">
                    <p class="notice">注：股池仅展示我收藏的公司</p>
                    <div class="wrapper-item-content-select">
                        <span>股池：</span>
                        <Select v-model="selectedCompany" size="small" style="width:500px">
                            <Option v-for="(item, i) in stockData" 
                                :key="i" :value="item.ts_code" 
                                :label="item.company_name">
                                <span>{{item.company_name}}</span>
                                <span style="float:right; margin-right:20px; color:#ccc">{{item.ts_code}}</span>
                            </Option>
                        </Select>
                    </div>
                </div>
            </div>
            <!-- 策略选择 -->
            <div class="wrapper-item">
                <h4>策略选择：</h4>
                <div class="wrapper-item-content factors-wrapper">
                    <p class="notice">注：买入卖出策略暂无法更改</p>
                    <div class="wrapper-item-content-select">
                        <span>买入策略：</span>
                        <Select v-model="buySelectFactors" multiple disabled size="small" style="width:500px">
                            <Option v-for="(item, i) in buyFactors" 
                                :key="i" 
                                :value="item.alias"
                                :label="item.text">
                                <span>{{item.text}}</span>
                            </Option>
                        </Select>
                    </div>
                    <div class="wrapper-item-content-select">
                        <span>卖出策略：</span>
                        <Select v-model="sellSelectFactors" multiple disabled size="small" style="width:500px">
                            <Option v-for="(item, i) in sellFactors" 
                                :key="i" 
                                :value="item.alias"
                                :label="item.text">
                                <span>{{item.text}}</span>
                            </Option>
                        </Select>
                    </div>
                </div>
            </div>
            <div class="wrapper-button">
                <Button type="primary" :disabled="sureButtonDisable" @click="submitStrategy">确定</Button>
                <Button type="warning" @click="cancle">取消</Button>
            </div>
        </div>
    </div>
</template>

<script>
    const DAY = 86400000;
    import { FACTORS,BUY_FACTORS, SELL_FACTROS } from '../../common/utils.js';
    import _ from 'lodash';

    export default {
        name: 'strategy',
        data() {
            return {
                initialFunding: 100000, // 初始资金
                startTime: new Date(Date.now() - DAY * 365 * 1 - DAY * 30), // 默认开始时间
                endTime: new Date(Date.now() - DAY * 30), //默认结束时间
                factors: FACTORS.toArray(), // 策略
                buyFactors: BUY_FACTORS.toArray(),
                buySelectFactors: _.map(BUY_FACTORS.toArray(), 'alias'),
                sellFactors: SELL_FACTROS.toArray(),
                sellSelectFactors: _.map(SELL_FACTROS.toArray(), 'alias'),
                stockData: [],
                selectedCompany: '', // 股池
                sureButtonDisable: true
            };
        },

        components: {},

        created () {
            this.userData = this.$getUserInfo();
            this.userData && this.loadCollectedList();
        },
        watch: {
            selectedCompany(com) {
                com.length === 0 ? (this.sureButtonDisable = true) : (this.sureButtonDisable = false);
            }
        },
        methods: {
            loadCollectedList() {
                const payload = {
                    user_id: this.userData.id
                }
                return this.$api.post('/api/getUserColledCompany', payload).then(res => {
                    this.stockData = res.list;
                    return res;
                })
            },
            submitStrategy() {
                let strategData = {
                    company_name: this.selectedCompany,
                    read_cash: this.initialFunding,
                    start: this.$getLocalTime(this.startTime, true),
                    end: this.$getLocalTime(this.endTime, true),
                    selectedCompany: this.selectedCompany
                }
                this.$emit("closeStrategyDrawer", strategData)
            },
            cancle() {
                this.$emit("closeStrategyDrawer")
            },
            onChangeCompany(item){
                console.log(item);
            }
        },

    }
</script>

<style scoped lang='less'>
    .strategy {
        .wrapper {
            .wrapper-item {
                padding: 5px 20px;
                border-bottom: 1px solid #eee;

                h4 {
                    font-size: 15px;
                }

                .wrapper-item-content {
                    padding: 10px 20px;
                    display: flex;
                    flex-wrap: wrap;
                    // &.factors-wrapper {
                    //     min-height: 200px;
                    // }
                    .wrapper-item-content-list {
                        padding: 10px;
                        width: calc(100%/3);

                        span {
                            vertical-align: middle;
                        }
                    }

                    .wrapper-item-content-select {
                        padding: 10px;
                        width: 100%;
                        span {
                            display:  inline-block;
                            width: 70px;
                        }
                    }

                    .notice {
                        padding-left: 10px;
                        color: #bbb;
                    }
                }
            }
            .wrapper-button {
                padding: 20px 30px;
                button {
                    width: 80px;
                    margin: 0 10px;
                }
            }
        }
    }
</style>