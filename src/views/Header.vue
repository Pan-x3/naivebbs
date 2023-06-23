<template>
  <transition name="Header">
    <div class="header" v-show="showHeader" :class="{ header_mob_show: showMobNav }">
      <div class="header-content">
        <div class="mob_nav">
          <router-link to="/">
            <div class="logo">NaiveBBS</div>
          </router-link>
          <div class="mob_button" @click="showMobNav = !showMobNav">
            <div></div>
            <div></div>
            <div></div>
          </div>
        </div>
        <!-- 板块信息 -->
        <div class="menu">
          <router-link to="/" class="menu-item" style="color: black;">首页</router-link>
          <template v-for="board in boardList">
            <el-popover placement="bottom-start" :width="300" trigger="hover" v-if="board.children.length > 0" :disabled="disablePopover">
              <template #reference>
                <span :class="[
                  'menu-item',
                  board.boardId == activePboardId ? 'active' : '',
                ]" @click="boardClickHandler(board)">{{ board.boardName }}</span>
              </template>
              <div class="sub-board-list">
                <span v-for="subBoard in board.children" @click="subBoardClickHandler(subBoard)" :class="[
                  'sub-board',
                  subBoard.boardId == activeboardId ? 'active' : '',
                ]">{{ subBoard.boardName }}</span>
              </div>
            </el-popover>
            <span class="menu-item" v-else @click="boardClickHandler(board)" :class="[
              'menu-item',
              board.boardId == activePboardId ? 'active' : '',
            ]">{{ board.boardName }}</span>
          </template>
        </div>
        <!-- 登录 注册 用户信息 -->
        <div class="user-info-panel">
          <el-button type="primary" class="op-btn" @click="newPost">
            发帖<span class="iconfont icon-add"></span>
          </el-button>
          <el-button type="primary" class="op-btn" @click="toSearch">
            搜索<span class="iconfont icon-search"></span>
          </el-button>
          <!-- 显示用户信息 -->
          <template v-if="userInfo?.userId">
            <div class="message-info">
              <el-dropdown>
                <el-badge :value="messageCountInfo.total" class="item" :hidden="messageCountInfo.total == null || messageCountInfo.total == 0
                  ">
                  <div class="iconfont icon-message" style="transform: scale(0.8);"></div>
                </el-badge>
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item @click="gotoMessage('reply')" class="message-item"><span class="text">回复我的</span>
                      <span class="count-tag" v-if="messageCountInfo.reply > 0">{{
                        messageCountInfo.reply > 99
                        ? "99+"
                        : messageCountInfo.reply
                      }}</span>
                    </el-dropdown-item>
                    <el-dropdown-item @click="gotoMessage('likePost')" class="message-item"><span
                        class="text">赞了我的文章</span>
                      <span class="count-tag" v-if="messageCountInfo.likePost > 0">{{
                        messageCountInfo.likePost > 99
                        ? "99+"
                        : messageCountInfo.likePost
                      }}</span></el-dropdown-item>
                    <el-dropdown-item @click="gotoMessage('likeComment')" class="message-item"><span
                        class="text">赞了我的评论</span>
                      <span class="count-tag" v-if="messageCountInfo.likeComment > 0">{{
                        messageCountInfo.likeComment > 99
                        ? "99+"
                        : messageCountInfo.likeComment
                      }}</span>
                    </el-dropdown-item>
                    <el-dropdown-item @click="gotoMessage('downloadAttachment')" class="message-item"><span
                        class="text">下载了我的附件</span>
                      <span class="count-tag" v-if="messageCountInfo.downloadAttachment > 0">{{
                        messageCountInfo.downloadAttachment > 99
                        ? "99+"
                        : messageCountInfo.downloadAttachment
                      }}</span></el-dropdown-item>
                    <el-dropdown-item @click="gotoMessage('sys')" class="message-item"><span class="text">系统消息</span>
                      <span class="count-tag" v-if="messageCountInfo.sys > 0">{{
                        messageCountInfo.sys > 99 ? "99+" : messageCountInfo.sys
                      }}</span></el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </div>
            <div class="user-info">
              <el-dropdown>
                <Avatar :userId="userInfo?.userId" :width="40" :addLink="false" />
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item @click="gotoUcenter(userInfo.userId)">个人主页</el-dropdown-item>
                    <el-dropdown-item @click="logout">退出</el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </div>
          </template>

          <el-button-group :style="{ 'margin-left': '12px' }" v-else>
            <el-button type="primary" plain @click="loginAndRegister(1)">登录</el-button>
            <el-button type="primary" plain @click="loginAndRegister(0)">注册</el-button>
          </el-button-group>
        </div>
      </div>
      <LoginAndRegister ref="loginAndRegisterRef" />
    </div>
  </transition>
</template>

<script setup>
import { useStore } from "vuex";
import {
  ref,
  getCurrentInstance,
  onMounted,
  watch,
  defineEmits,
  onUnmounted
} from "vue";
import { useRouter } from "vue-router";
import LoginAndRegister from "@/views/LoginAndRegister.vue";
const { proxy } = getCurrentInstance();
const router = useRouter();
const store = useStore();
const emit = defineEmits(["loginAndRegister"]);

const api = {
  getUserInfo: "/getUserInfo",
  loadBoard: "/board/loadBoard",
  loadMessageCount: "/ucenter/getMessageCount",
  logout: "/logout",
  getSysSetting: "/getSysSetting",
  loadMessageCount: "/ucenter/getMessageCount",
};

//处理头部和滚动条的位置
const showHeader = ref(true);
//获取滚动条的高度
const getScrollTop = () => {
  let scrollTop =
    document.documentElement.scrollTop ||
    window.pageYOffset ||
    document.body.scrollTop; //兼容问题
  return scrollTop;
};

function handleScroll() {
  let oldScrollTop = 0;
  return function () {
    let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop;
    if (scrollTop < 100) {
      showHeader.value = true;
    }
    else {
      showHeader.value = false;
      if (oldScrollTop > scrollTop) {
        showHeader.value = true;
      }
    }
    oldScrollTop = scrollTop;
  };
};
window.addEventListener("scroll", handleScroll());
onUnmounted(() => {
  window.removeEventListener("scroll", handleScroll());
  handleScroll = null;
});
//登陆注册
const loginAndRegisterRef = ref();
const loginAndRegister = (type) => {
  loginAndRegisterRef.value.showPanel(type);
};
onMounted(() => {
  getUserInfo();
});
//获取用户信息
const getUserInfo = async () => {
  let result = await proxy.Request({
    url: api.getUserInfo,
  });
  if (!result) {
    return;
  }
  store.commit("updateLoginUserInfo", result.data);
};
//获取板块信息
const boardList = ref([]);
const loadBoard = async () => {
  let result = await proxy.Request({
    url: api.loadBoard,
  });
  if (!result) {
    return;
  }
  boardList.value = result.data;
  store.commit("saveBoardList", result.data);
};
loadBoard();

//监听登录用户信息
const userInfo = ref({});
watch(
  () => store.state.loginUserInfo,
  (newVal, oldVal) => {
    if (newVal != undefined && newVal != null) {
      userInfo.value = newVal;
    } else {
      userInfo.value = {};
    }
  },
  { immediate: true, deep: true }
);
//监听是否展示登录框
watch(
  () => store.state.showLogin,
  (newVal, oldVal) => {
    if (newVal) {
      loginAndRegister(1);
    }
  },
  { immediate: true, deep: true }
);

//板块切换
//一级板块
const boardClickHandler = (board) => {
  router.push(`/forum/${board.boardId}`);
};
//二级板块
const subBoardClickHandler = (subBoard) => {
  router.push(`/forum/${subBoard.pBoardId}/${subBoard.boardId}`);
};
//当前选中的版块
const activePboardId = ref(0);
watch(
  () => store.state.activePboardId,
  (newVal, oldVal) => {
    if (newVal !== 0) {
      activePboardId.value = newVal;
    }
  },
  { immediate: true, deep: true }
);

const activeboardId = ref(0);
watch(
  () => store.state.activeBoardId,
  (newVal, oldVal) => {
    activeboardId.value = newVal;
  },
  {
    immediate: true,
    deep: true,
  }
);

//发帖
const newPost = () => {
  if (!store.getters.getLoginUserInfo) {
    loginAndRegister(1);
  } else {
    router.push("/newPost");
  }
};

const gotoUcenter = (userId) => {
  router.push(`/user/${userId}`);
};
//消息相关
const gotoMessage = (type) => {
  router.push(`/user/message/${type}`);
};

const messageCountInfo = ref({});
const loadMessageCount = async () => {
  let result = await proxy.Request({
    url: api.loadMessageCount,
  });
  if (!result) {
    return;
  }
  messageCountInfo.value = result.data;
  store.commit("updateMessageCountInfo", result.data);
};

watch(
  () => store.state.messageCountInfo,
  (newVal, oldVal) => {
    messageCountInfo.value = newVal || {};
  },
  { immediate: true, deep: true }
);

watch(
  () => store.state.loginUserInfo,
  (newVal, oldVal) => {
    if (newVal) {
      loadMessageCount();
    }
  },
  { immediate: true, deep: true }
);
//退出
const logout = () => {
  proxy.Confirm("确定要退出吗?", async () => {
    let result = await proxy.Request({
      url: api.logout,
    });
    if (!result) {
      return;
    }
    store.commit("updateLoginUserInfo", null);
  });
};
const toSearch = () => {
  router.push("/search");
};
const showMobNav = ref(false);
router.afterEach(() => {
  showMobNav.value = false;
});
const disablePopover = navigator.userAgent.match(/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone|HarmonyOS)/i);
</script>

<style lang="scss" scoped>
.header {
  width: 100vw;
  box-shadow: 0 1px 5px 0 rgba(0, 0, 0, 0.1);
  height: 60px;
  position: fixed;
  top: 0;
  background-color: rgba(255, 255, 255, 0.5);
  backdrop-filter: blur(6px);
  z-index: 1000;

  .header-content {
    width: 1300px;
    margin: 0px auto;
    align-items: center;
    height: 60px;
    display: flex;
    justify-content: space-around;

    a {
      text-decoration: none;
    }

    .logo {
      font-family: 'Times New Roman', Times, serif;
      font-size: 26px;
      font-weight: 500;
      color: #333;
      cursor: pointer;
    }

    .menu {
      width: 400px;
      height: 60px;
      line-height: 60px;
      display: flex;

      .home {
        text-decoration: none;
      }

      .menu-item {
        margin-left: 20px;
        cursor: pointer;
      }

      a {
        text-decoration: none;
      }
    }

    .user-info-panel {
      width: 350px;
      display: flex;
      align-items: center;

      .op-btn {
        .iconfont {
          margin-left: 5px;
        }

        .el-button+.el-button {
          margin-left: 5px;
        }
      }

      .message-info {
        margin-left: 20px;
        margin-right: 20px;
        cursor: pointer;

        .icon-message {
          font-size: 25px;
          color: rgb(147, 147, 147);
        }
      }
    }
  }
}

.sub-board-list {
  display: flex;
  flex-wrap: wrap;

  .sub-board {
    padding: 4px 12px;
    border-radius: 15px;
    margin-right: 10px;
    background-color: rgb(230, 230, 230);
    color: rgb(50, 50, 50);
    margin-top: 10px;
    cursor: pointer;
  }

  .sub-board:hover {
    color: var(--link);
  }

  .active {
    background-color: var(--link);
    color: #fff;
  }

  .active:hover {
    color: #fff;
  }
}

.message-item {
  display: flex;
  justify-content: space-around;

  .text {
    flex: 1;
  }

  .count-tag {
    height: 15px;
    line-height: 15px;
    min-width: 20px;
    display: inline-block;
    background: #f56c6c;
    border-radius: 10px;
    font-size: 13px;
    text-align: center;
    color: #fff;
    margin-left: 10px;
  }
}

.Header-enter-active,
.Header-leave-active {
  transition: all 0.5s;
}

.Header-enter-from,
.Header-leave-to {
  opacity: 0;
  transform: translateY(-100%);
}

.Header-enter-to,
.Header-leave-from {
  opacity: 1;
  transform: translateY(0);
}

.el-tooltip__trigger:focus {
  outline: none;
}

@media screen and (max-width: 768px) {
  .header_mob_show {
    height: 180px;
  }

  .header {
    transition: all 0.5s ease-in-out;
    // height: 210px;
    overflow-y: hidden;

    .header-content {
      height: 100%;
      width: 100vw;
      flex-wrap: wrap;
      justify-content: center;
      align-items: center;

      .mob_nav {
        display: flex;

        .mob_button {
          cursor: pointer;
          width: 30px;
          position: absolute;
          top: 22px;
          right: 20px;

          div {
            width: 20px;
            height: 3px;
            transform: scale(1, 0.5);
            background-color: black;
            margin-bottom: 4px;
          }
        }
      }

      .menu,
      .user-info-panel {
        // display: none;
        height: 60px;
        width: 100vw;
        justify-content: center;
      }

      .logo {
        display: block;
        width: 100px;
        line-height: 60px;
        text-align: center;
        font-size: 22px;
      }
      .menu a{
        margin-left:0 !important;
      }
    }
  }
}</style>