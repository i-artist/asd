<template>
  <div class="app-container">

    <div class="main-container">
      <div style="margin: 20px">
        <div>
          <el-button type="info" size="small" @click="changeState(0)">全部</el-button>
          <el-button type="info" size="small" @click="changeState(1)">待审</el-button>
          <el-button type="info" size="small" @click="changeState(2)">删除</el-button>
          <el-button type="info" size="small" @click="changeState(3)">通过</el-button>
          <el-button type="info" size="small" @click="changeState(4)">误删</el-button>
          <el-button type="info" size="small" @click="changeState(5)">漏删</el-button>
          <!-- <el-select v-model="state.timeHourpick" placeholder="请选择时间段" style="float:right; width:110px">
            <el-option v-for="item in timeSel" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
          <el-date-picker v-model="timeDayPick" type="date" placeholder="选择日期" style="float:right;margin-right:30px;width:120px" :clearable="false" format="yyyy-MM-dd" @change="dateChange">
          </el-date-picker> -->
          <div class="block" style="float:right;">
            <span class="demonstration" style="font-size: 14px;color: #48576a;font-weight:900;">请选择查询范围:</span>
             <el-date-picker
                v-model="timeDayPick"
                :picker-options="pickerOptions2"
                type="datetimerange"
                range-separator="至"
                start-placeholder="开始日期"
                end-placeholder="结束日期"
                value-farmat="yyyy\/MM\/dd"
                @change="dateChange"
                :clearable="false">
              </el-date-picker>
          </div>
        </div>
        <div style="margin: 15px 0;"></div>
        <el-form>
          <el-form-item label="当前状态:">
            <el-radio-group v-model="state.currentState" size="small">
              <el-radio-button label="0">不限</el-radio-button>
              <el-radio-button label="1">通过</el-radio-button>
              <el-radio-button label="2">待审</el-radio-button>
              <el-radio-button label="3">删除</el-radio-button>
              <el-radio-button label="4">通过+待审</el-radio-button>
              <el-radio-button label="5">通过+删除</el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="人工审核:">
            <el-radio-group v-model="state.humanReview" size="small">
              <el-radio-button label="0">不限</el-radio-button>
              <el-radio-button label="1">未确认</el-radio-button>
              <el-radio-button label="2">已确认</el-radio-button>
              <el-radio-button label="3">已忽略</el-radio-button>
            </el-radio-group>
            <el-button v-show="!listViewOpen" size="mini" type="warning" plain style="float:right" @click="listViewOpen=true">完整选项</el-button>
          </el-form-item>
          <div v-show="listViewOpen">
            <el-form-item label="内容类型:">
              <el-radio-group v-model="state.contentType" size="small">
                <el-radio-button label="0">不限</el-radio-button>
                <el-radio-button label="1">主题</el-radio-button>
                <el-radio-button label="2">回复</el-radio-button>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="识别类型:">
              <el-radio-group v-model="state.indentifyType" size="small">
                <el-radio-button label="0">不限</el-radio-button>
                <el-radio-button label="1">涉政</el-radio-button>
                <el-radio-button label="2">涉黄</el-radio-button>
                <el-radio-button label="3">涉恐</el-radio-button>
                <el-radio-button label="4">广告</el-radio-button>
                <el-radio-button label="5">低俗</el-radio-button>
                <el-radio-button label="6">敏感</el-radio-button>
                <el-radio-button label="7">灌水</el-radio-button>
                <el-radio-button label="8">个性</el-radio-button>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="彩数识别:">
              <el-radio-group v-model="state.colourdataType" size="small">
                <el-radio-button label="0">不限</el-radio-button>
                <el-radio-button label="1">通过</el-radio-button>
                <el-radio-button label="2">待审</el-radio-button>
                <el-radio-button label="3">删除</el-radio-button>
              </el-radio-group>
              <el-button size="mini" type="warning" plain style="float:right" @click="listViewOpen=false">简略选项</el-button>
            </el-form-item>
          </div>
        </el-form>
      </div>
      <sticky className="sub-navbar">
        <el-select v-model="listQuery.seachCondition" placeholder="请选择筛选条件" class="filter-item" style="float:left;margin-left:20px">
          <el-option v-for="item in seachSel" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
        <el-input style="width: 250px;float:left;margin-left:20px" class="filter-item" :placeholder="'请输入'+seachCondition" v-model="listQuery.seachContent">
        </el-input>
        <el-button class="filter-item" type="success" style="float:left;margin-left:20px;margin-top:8px" v-waves icon="search" @click="getList">搜索</el-button>
      </sticky>
      <div style="margin: 15px 0;"></div>
      <div style="margin: 15px; display: flex;justify-content:space-between;">
        <label style="float:left">总量：{{total}}</label>
      </div>

      <!-- <el-collapse v-for="(item, index) in list" :key="item.userid" v-loading="listLoading" accordion>
                    <el-collapse-item :title="item.userid+' ('+item.username+')'+' (共'+item.total+'条)'" name="index">
                      <workstationid :id="item.userid" :listQueryId="listQuery" :state="state"></workstationid>
                    </el-collapse-item>
                  </el-collapse> -->
      <div v-for="(item, index) in list" :key="item.userid" v-loading="listLoading">
        <el-row style="height:44px;border:1px solid #D3D3D3;display:flex;align-items:center;padding-left:10px;color:#48576a">
          <i @click="boxIsShow(item,$event)" ref="clickBtn" class="el-icon-arrow-right" style="cursor: pointer" :name="item.userid"></i>
          <p style="margin-left:10px">{{item.userid+' ('+item.username+')'+' (共'+item.total+'条)'}}</p>
        </el-row>
        <el-row style="background: white" v-if="item.boxshow">
          <workstationid :id="item.userid" :listQueryId="listQuery" :state="state"></workstationid>
        </el-row>
      </div>

      <div v-show="!listLoading" class="pagination-container" style="  display: flex;justify-content: center;align-items: center;">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.page" :page-sizes="[20, 50, 100]" :page-size="listQuery.limit" layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>
      </div>
      <back-to-top transitionName="fade" :customStyle="myBackToTopStyle" :visibilityHeight="300" :backPosition="50"></back-to-top>
    </div>
  </div>
</template>

<script>
import Sticky from '@/components/Sticky' // 粘性header组件
import waves from '@/directive/waves.js'// 水波纹指令
import BackToTop from '@/components/BackToTop'
import { getUserIDWorkStation } from '@/api/content'
import workstationid from './workstationid'

export default {
  components: { Sticky, BackToTop, workstationid },
  directives: {
    waves
  },
  data() {
    return {
      pickerOptions2: {
          shortcuts: [{
            text: '最近一天',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 1);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三天',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 3);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }]
        },
      list: [],
       defaultDay: '',
       timeDayPick: this.getNowDay(),
      total: null,
      listLoading: true,
      listViewOpen: false,
      massList: [],
      listQuery: {
        page: 1,
        limit: 100,
        seachCondition: null,  //  查询种类 默认全部
        seachContent: null //  查询详情 默认全部
      },
      state: {
        currentState: 2,
        humanReview: 1,
        contentType: 0,
        indentifyType: 0,
        recognitionType: 0,
        colourdataType: 0,
        startTime: this.getNowDay() + ' 00:00:00',
        endTime: this.getNowDay() + ' 23:59:59'
      },
      seachCondition: '',
      seachContent: '',
      seachSel: [{
          value: 'PID',
          label: '帖子ID'
        },
        {
          value: 'SID',
          label: '主题ID'
        },
        {
          value: 'UID',
          label: '用户ID'
        }, {
          value: 'author',
          label: '用户名'
        },
        {
          value: 'PIP',
          label: '用户IP'
        },
        {
          value: 'content',
          label: '内容'
        }],
      timeSel: [{
        value: '0000',
        label: '近2小时'
      },
      {
        value: '0024',
        label: '全天'
      }, {
        value: '0002',
        label: '0点-2点'
      }, {
        value: '0204',
        label: '2点-4点'
      }, {
        value: '0406',
        label: '4点-6点'
      }, {
        value: '0608',
        label: '6点-8点'
      }, {
        value: '0810',
        label: '8点-10点'
      }, {
        value: '1012',
        label: '10点-12点'
      }, {
        value: '1214',
        label: '12点-14点'
      }, {
        value: '1416',
        label: '14点-16点'
      }, {
        value: '1618',
        label: '16点-18点'
      }, {
        value: '1820',
        label: '18点-20点'
      }, {
        value: '2022',
        label: '20点-22点'
      }, {
        value: '2224',
        label: '22点-24点'
      }],
      locationSel: [{
        label: '论坛、评论',
        options: [{
          value: '论坛',
          label: '论坛'
        }, {
          value: '回帖',
          label: '回帖'
        }, {
          value: '历史数据清洗',
          label: '历史数据清洗'
        }]
      }, {
        label: '问答',
        options: [{
          value: '提问',
          label: '提问'
        }, {
          value: '答案',
          label: '答案'
        }]
      }, {
        label: '精华帖',
        options: [{
          value: '精华帖',
          label: '精华贴'
        }]
      }],
      myBackToTopStyle: {
        right: '50px',
        bottom: '50px',
        width: '40px',
        height: '40px',
        'border-radius': '4px',
        'line-height': '45px', // 请保持与高度一致以垂直居中
        background: '#e7eaf1'// 按钮的背景颜色
      }
    };
  },
  computed: {
    checkAll: {
      get() {
        return this.checkedCount === this.list.length;
      },
      set(value) {
        this.lists = this.list.map(item => {
          item.checked = value;
          return item;
        });
      }
    },
    checkedCount: {
      get() {
        return this.list.filter(item => item.checked).length;
      }
    }
  },
  created() {
    this.getList();
     const date = new Date();
     
      let month = date.getMonth();
      let strDate = date.getDate();
      if (month >= 1 && month <= 9) {
        month = '0' + month;
      }
      if (strDate >= 0 && strDate <= 9) {
        strDate = '0' + strDate;
      }
      // console.log(date.getFullYear(), month, strDate)
      this.timeDayPick = [new Date(date.getFullYear(), month, strDate, '00', '00'), new Date(date.getFullYear(), month, strDate, 23, 59, 59)]
      this.defaultDay = this.timeDayPick
 },
  methods: {
    getList() {
      this.listLoading = true
      getUserIDWorkStation(this.listQuery, this.state).then(response => {
        this.list = response.data.items.map(v => {
          this.$set(v, 'boxshow', false);
          return v
        });
        console.log(response.data);
        this.total = response.data.total
        this.listLoading = false
      })
    },
    handleSizeChange(val) {
      this.listQuery.limit = val
      this.getList()
    },
    handleCurrentChange(val) {
      this.listQuery.page = val
      this.getList()
    },
    changeState(state) {
      if (state === 0) {
        this.state.currentState = 0;
        this.state.humanReview = 1;
        this.state.contentType = 0;
        this.state.indentifyType = 0;
        this.state.recognitionType = 0;
        this.state.colourdataType = 0;
      }
      if (state === 1) {
        this.state.currentState = 4;
        this.state.humanReview = 1;
        this.state.contentType = 0;
        this.state.indentifyType = 0;
        this.state.recognitionType = 0;
        this.state.colourdataType = 0;
      }
      if (state === 2) {
        this.state.currentState = 3;
        this.state.humanReview = 1;
        this.state.contentType = 0;
        this.state.indentifyType = 0;
        this.state.recognitionType = 0;
        this.state.colourdataType = 0;
      }
      if (state === 3) {
        this.state.currentState = 1;
        this.state.humanReview = 1;
        this.state.contentType = 0;
        this.state.indentifyType = 0;
        this.state.recognitionType = 0;
        this.state.colourdataType = 0;
      }
      if (state === 4) {
        this.state.currentState = 1;
        this.state.humanReview = 2;
        this.state.contentType = 0;
        this.state.indentifyType = 0;
        this.state.recognitionType = 0;
        this.state.colourdataType = 3;
      }
      if (state === 5) {
        this.state.currentState = 3;
        this.state.humanReview = 2;
        this.state.contentType = 0;
        this.state.indentifyType = 0;
        this.state.recognitionType = 0;
        this.state.colourdataType = 1;
      }
    },
    getNowDay() {
      const date = new Date();
      const seperator1 = '-';
      let month = date.getMonth() + 1;
      let strDate = date.getDate();
      if (month >= 1 && month <= 9) {
        month = '0' + month;
      }
      if (strDate >= 0 && strDate <= 9) {
        strDate = '0' + strDate;
      }
      return date.getFullYear() + seperator1 + month + seperator1 + strDate;
    },
    dateChange(val) {
          const oTime = new Date().getTime()
      if (this.timeDayPick[1].getTime() - this.timeDayPick[0].getTime() > 604900000) {
        this.$message({
          type: 'info',
          message: '选择时间要在一周之内'
        })
        this.timeDayPick = this.defaultDay
        this.state.startTime = this.getNowDay() + ' 00:00:00'
        this.state.endTime = this.getNowDay() + ' 23:59:59'
      } else {
        const dateArr = val.split('至')
        this.state.startTime = dateArr[0];
        this.state.endTime = dateArr[1];
      }
    },
    boxIsShow(row, $event) {
      console.log(row, $event);
      console.log(event.currentTarget)
      row.boxshow = !row.boxshow
      const clickTarget = $event.currentTarget || $event.currentTarget;
      if (!row.boxshow) {
        clickTarget.style.transform = 'rotate(0deg)';
      } else {
        clickTarget.style.transform = 'rotate(90deg)';
      }
    }
  },
  watch: {
    state: {
      handler() {
        this.mainLoading = true;
        this.listQuery.page = 1;
        this.getList()
      },
      deep: true
    }
  }
}
</script>


<style rel="stylesheet/scss" lang="scss" scoped>
@import "src/styles/mixin.scss";
.main-container {
  border: 1px solid #d3dce6;
  margin: 80px;
  margin-top: 10px;
  background: #f3f3f3
}

.detail-box {
  border: 1px solid #d3dce6; // background: #F0FFFF
}

.aTitle {
  color: #4682B4;
}

.aHref {
  color: #00BFFF;
  text-decoration: underline;
}

.infoGrey {
  color: #808080;
}
</style>
