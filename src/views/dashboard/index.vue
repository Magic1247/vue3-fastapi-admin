<template>
  <div>
    <div class="pageHeaderContent">
      <div class="avatar">
        <a-avatar size="large" :src="currentUser.avatar"/>
      </div>
      <div class="content">
        <div class="contentTitle">
          欢迎，
          {{ currentUser.name }}
          ，祝你开心每一天！
        </div>
        <div>{{ currentUser.title }} |{{ currentUser.group }}</div>
      </div>
      <div class="extraContent">
        <div class="statItem">
          <a-statistic title="项目数" :value="projectNotice.length"/>
        </div>
        <div class="statItem">
          <a-statistic title="团队内排名" :value="1" suffix="/ 3"/>
        </div>
        <div class="statItem">
          <a-statistic title="项目访问" :value="1247"/>
        </div>
      </div>
    </div>

    <div style="padding: 10px">
      <a-row :gutter="24">
        <a-col :xl="16" :lg="24" :md="24" :sm="24" :xs="24">
          <a-card
              class="projectList"
              :style="{ marginBottom: '24px' }"
              title="进行中的项目"
              :bordered="false"
              :loading="false"
              :body-style="{ padding: 0 }"
          >
            <template #extra>
              <div @click="goToProjectPage"><span style="color: #1890ff">全部项目</span></div>
            </template>
            <a-card-grid
                v-for="item in projectNotice"
                :key="item.projectId"
                class="projectGrid"
            >
              <a-card
                  :body-style="{ padding: 0 }"
                  style="box-shadow: none"
                  :bordered="false"
              >
                <a-card-meta :description="item.projectDescribe" class="w-full">
                  <template #title>
                    <div class="cardTitle">
                      <a-avatar size="large" :src="currentUser.avatar"/>
                      <a :href="item.href">
                        {{ item.projectName }}
                      </a>
                    </div>
                  </template>
                </a-card-meta>
                <div class="projectItemContent">
                  <a :href="item.memberLink">
                    {{ item.createBy || "" }}
                  </a>
                  <span class="datetime" ml-2 :title="item.updateTime">
                    {{ parseTime(item.updateTime) }}
                  </span>
                </div>
              </a-card>
            </a-card-grid>
          </a-card>
          <a-card
              :body-style="{ padding: 0 }"
              :bordered="false"
              class="activeCard"
              title="动态"
              :loading="false"
          >
            <a-list :data-source="activities" class="activitiesList">
              <template #renderItem="{ item }">
                <a-list-item :key="item.operId">
                  <a-list-item-meta>
                    <template #title>
                      <span style="color: #F51928">{{ item.operName }}</span>&nbsp;
                      <span>在</span>&nbsp;

                      <span style="color: #F51928">{{ item.title }}</span>&nbsp;
                      <span>模块</span>&nbsp;

                      <span style="color: #F51928" v-if="item.businessType===1">新增</span>
                      <span style="color: #67C23A" v-if="item.businessType===2">修改</span>
                      <span style="color: #F51928" v-if="item.businessType===3">刪除</span>


                      &nbsp;<span>了数据</span>&nbsp;&nbsp;&nbsp;&nbsp;

                      <!--                      <span>相关方法: {{item.method}}</span>-->

                      <!--                      <span><el-button>详情</el-button></span>-->

                    </template>

                    <template #description>
                      <span class="datetime" :title="item.operTime">
                        {{ item.operTime }}
                      </span>
                    </template>
                  </a-list-item-meta>
                </a-list-item>
              </template>
            </a-list>
          </a-card>
        </a-col>
        <a-col :xl="8" :lg="24" :md="24" :sm="24" :xs="24">
          <a-card
              :style="{ marginBottom: '24px' }"
              title="Coming Soon..."
              :bordered="false"
              :body-style="{ padding: 0 }"
          >
            <EditableLinkGroup/>
          </a-card>

          <a-card
              :body-style="{ paddingTop: '12px', paddingBottom: '12px' }"
              :bordered="false"
              title="团队"
          >
            <div class="members">
              <a-row :gutter="48">
                <a-col
                    v-for="item in userList"
                    :key="`members-item-${item.id}`"
                    :span="12"
                >
                  <a :href="item.href">
                    <a-avatar size="small" :src="base_url + item.avatar"/>
                    <span class="member">{{ item.nickName }}</span>
                  </a>
                </a-col>
              </a-row>
            </div>
          </a-card>

          <a-card
              :style="{ marginBottom: '24px' }"
              :bordered="false"
              title="登录日志"
          >
            <div v-for="info in LoginInfoList" class="LoginList">
              {{ info.userName }} 于 {{ parseTime(info.loginTime) }} 登录了系统
            </div>
          </a-card>
        </a-col>
      </a-row>
    </div>
  </div>


</template>

<script>
import {
  Statistic,
  Row,
  Col,
  Card,
  CardGrid,
  CardMeta,
  List,
  ListItem,
  ListItemMeta,
  Avatar,
} from "ant-design-vue";
import 'ant-design-vue/dist/reset.css';

export default {
  components: {
    AStatistic: Statistic,
    ARow: Row,
    ACol: Col,
    ACard: Card,
    ACardGrid: CardGrid,
    ACardMeta: CardMeta,
    AList: List,
    AListItem: ListItem,
    AListItemMeta: ListItemMeta,
    AAvatar: Avatar,
  },
};
</script>


<script setup>
// import {Radar} from "@antv/g2plot";
import EditableLinkGroup from "./editable-link-group.vue";
import useUserStore from "@/store/modules/user.js";
// import {pageListProject} from "@/api/apiTest/project.js";
import {useRouter} from "vue-router";
import {parseTime} from "@/utils/common.js";
import {listUser} from "@/api/system/user.js";
import {list} from "@/api/monitor/operlog.js";
import {list as getLoginInfoList} from '@/api/monitor/logininfor.js'

const userStore = useUserStore();
const base_url = import.meta.env.VITE_APP_BASE_API


defineOptions({
  name: "DashBoard",
});


const currentUser = ref({
  avatar: "",
  name: "",
  userid: "",
  email: "antdesign@alipay.com",
  signature: "Less is more",
  title: "点工",
  group: "人生无限公司",
});
currentUser.value.avatar = userStore.avatar
currentUser.value.name = userStore.name
currentUser.value.userid = userStore.id


const projectNotice = ref([])
const router = useRouter()

function goToProjectPage() {
  router.push('/projectMange/project')
}


const userList = ref([])

function getUserList() {
  listUser().then(
      res => {
        userList.value = res.rows
      }
  )
}

const activities = ref([])

function getActivities() {
  list().then(
      res => {
        activities.value = res.rows
      }
  )
}


const LoginInfoList = ref([
  {
    "infoId": 180,
    "userName": "Administrator",
    "ipaddr": "127.0.0.1",
    "loginLocation": "内网IP",
    "browser": "Chrome 124",
    "os": "Windows 10",
    "status": "0",
    "msg": "登录成功",
    "loginTime": "2024-04-29T05:25:46"
  },])

function getLoginInfo() {
  getLoginInfoList().then(
      res => {
        LoginInfoList.value = res.rows
      }
  )
}

// let radar;
onMounted(() => {


  // getProjectNotice()
  getUserList()
  // getActivities()
  // getLoginInfo()

});
//
// onBeforeUnmount(() => {
//   // radar?.destroy?.();
// });
</script>

<style scoped lang="less">
.textOverflow() {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  word-break: break-all;
}

// mixins for clearfix
// ------------------------
.clearfix() {
  zoom: 1;
  &::before,
  &::after {
    display: table;
    content: " ";
  }
  &::after {
    clear: both;
    height: 0;
    font-size: 0;
    visibility: hidden;
  }
}

.activitiesList {
  padding: 0 24px 8px 24px;

  .username {
    color: rgba(0, 0, 0, 0.65);
  }

  .event {
    font-weight: normal;
  }
}

.pageHeaderContent {
  display: flex;
  padding: 12px;
  margin-bottom: 24px;
  box-shadow: rgba(0, 0, 0, 0.1) 0px 4px 12px;

  .avatar {
    flex: 0 1 72px;

    & > span {
      display: block;
      width: 72px;
      height: 72px;
      border-radius: 72px;
    }
  }

  .content {
    position: relative;
    top: 4px;
    flex: 1 1 auto;
    margin-left: 24px;
    color: rgba(0, 0, 0, 0.45);
    line-height: 22px;

    .contentTitle {
      margin-bottom: 12px;
      color: rgba(0, 0, 0, 0.85);
      font-weight: 500;
      font-size: 20px;
      line-height: 28px;
    }
  }
}

.extraContent {
  .clearfix();

  float: right;
  white-space: nowrap;

  .statItem {
    position: relative;
    display: inline-block;
    padding: 0 32px;

    > p:first-child {
      margin-bottom: 4px;
      color: rgba(0, 0, 0, 0.45);
      font-size: 14px;
      line-height: 22px;
    }

    > p {
      margin: 0;
      color: rgba(0, 0, 0, 0.85);
      font-size: 30px;
      line-height: 38px;

      > span {
        color: rgba(0, 0, 0, 0.45);
        font-size: 20px;
      }
    }

    &::after {
      position: absolute;
      top: 8px;
      right: 0;
      width: 1px;
      height: 40px;
      background-color: #e8e8e8;
      content: "";
    }

    &:last-child {
      padding-right: 0;

      &::after {
        display: none;
      }
    }
  }
}

.members {
  a {
    display: block;
    height: 24px;
    margin: 12px 0;
    color: rgba(0, 0, 0, 0.65);
    transition: all 0.3s;
    .textOverflow();

    .member {
      margin-left: 12px;
      font-size: 14px;
      line-height: 24px;
      vertical-align: top;
    }

    &:hover {
      color: #1890ff;
    }
  }
}

.projectList {
  :deep(.ant-card-meta-description) {
    height: 44px;
    overflow: hidden;
    color: rgba(0, 0, 0, 0.45);
    line-height: 22px;
  }

  .cardTitle {
    font-size: 0;

    a {
      display: inline-block;
      height: 24px;
      margin-left: 12px;
      color: rgba(0, 0, 0, 0.85);
      font-size: 14px;
      line-height: 24px;
      vertical-align: top;

      &:hover {
        color: #1890ff;
      }
    }
  }

  .projectGrid {
    width: 33.33%;
  }

  .projectItemContent {
    display: flex;
    flex-basis: 100%;
    height: 20px;
    margin-top: 8px;
    overflow: hidden;
    font-size: 12px;
    line-height: 20px;
    .textOverflow();

    a {
      display: inline-block;
      flex: 1 1 0;
      color: rgba(0, 0, 0, 0.45);
      .textOverflow();

      &:hover {
        color: #1890ff;
      }
    }

    .datetime {
      flex: 0 0 auto;
      float: right;
      color: rgba(0, 0, 0, 0.25);
    }
  }
}

.datetime {
  color: rgba(0, 0, 0, 0.25);
}

@media screen and (max-width: 1200px) and (min-width: 992px) {
  .activeCard {
    margin-bottom: 24px;
  }

  .members {
    margin-bottom: 0;
  }

  .extraContent {
    margin-left: -44px;

    .statItem {
      padding: 0 16px;
    }
  }
}

@media screen and (max-width: 992px) {
  .activeCard {
    margin-bottom: 24px;
  }

  .members {
    margin-bottom: 0;
  }

  .extraContent {
    float: none;
    margin-right: 0;

    .statItem {
      padding: 0 16px;
      text-align: left;

      &::after {
        display: none;
      }
    }
  }
}

@media screen and (max-width: 768px) {
  .extraContent {
    margin-left: -16px;
  }

  .projectList {
    .projectGrid {
      width: 50%;
    }
  }
}

@media screen and (max-width: 576px) {
  .pageHeaderContent {
    display: block;

    .content {
      margin-left: 0;
    }
  }

  .extraContent {
    .statItem {
      float: none;
    }
  }
}

@media screen and (max-width: 480px) {
  .projectList {
    .projectGrid {
      width: 100%;
    }
  }
}

.LoginList {
  padding: 10px;
  color: #1c84c6;
}
</style>