<template>
    <div class="framework">
      <div class="header">
        <div class="logo">
            <span class="iconfont icon-pan"></span>
            <div class="name">个人云盘</div>
        </div>
        <div class="right-panel">
            <el-popover
              :width="800"
              trigger="click"
              v-model:visible="showUploader"
              :offset="20"
              transition="none"
              :hide-after="0"
              :popper-style="{ padding: '0px' }"
            >
            <template #reference>
                <span class="iconfont icon-transfer"></span>
            </template>
            <template #default>
              <Uploader
                ref="uploaderRef"
                @uploadCallback="uploadCallbackHandler"
            ></Uploader>
            </template>
          </el-popover>

          <el-dropdown>
            <div class="user-info">
                <div class="avatar">
                    <Avatar
                      :userId="userInfo.userId"
                      :avatar="userInfo.avatar"
                      :timestamp="timestamp"
                      :width="46"
                      ></Avatar>
                </div>
                <span class="nick-name">{{ userInfo.nickName }}</span>
            </div>
            <template #dropdown>
                <el-dropdown-menu>
                    <el-dropdown-item @click="updateAvatar"
                    >修改头像</el-dropdown-item
                    >
                    <el-dropdown-item @click="updatePassword"
                    >修改密码</el-dropdown-item
                    >
                    <el-dropdown-item @click="logout">退出</el-dropdown-item>
                </el-dropdown-menu>
            </template>
          </el-dropdown>
        </div>
      </div>
      <div class="body">
        <div class="left-sider">
            <div class="menu-list">
                <template v-for="item in menus">
                <div
                v-if="item.allShow || (!item.allShow && userInfo.admin)"
                @click="jump(item)"
                :class="[
                    'menu-item',
                    item.menuCode == currentMenu.menuCode ? 'active' : '',
                    ]"
                >
                    <div :class="['iconfont', 'icon-' + item.icon]"></div>
                    <div class="text">{{ item.name }}</div>
                </div>
            </template>
              </div>
            <div class="menu-sub-list">
                <div
                  @click="jump(sub)"
                  :class="['menu-item-sub', currentPath == sub.path ? 'active' : '']"
                  v-for="sub in currentMenu.children"
                >
                  <span
                  :class="['iconfont', 'icon-' + sub.icon]"
                  v-if="sub.icon"
                  ></span>
                  <span class="text">{{ sub.name }}</span>
              </div>
              <div class="tips" v-if="currentMenu && currentMenu.tips">
                {{ currentMenu.tips }}
              </div>
              <div class="space-info">
                <div>空间使用</div>
                <div class="percent">
                    <el-progress
                      :percentage="
                        Math.floor(
                            (useSpaceInfo.useSpace / useSpaceInfo.totalSpace) * 10000
                        ) / 100
                      "
                      color="#4F4F4F"
                    ></el-progress>
<!--                  color="#f701ff"-->
                </div>
                <div class="space-use">
                    <div class="use">
                        {{ proxy.Utils.size2Str(useSpaceInfo.useSpace) }} / {{
                           proxy.Utils.size2Str(useSpaceInfo.totalSpace)
                        }}
                    </div>
                    <div class="iconfont icon-refresh" @click="getUseSpace"></div>
                </div>
              </div>
            </div>
        </div>
        <div class="body-content">
            <router-view v-slot="{ Component }">
                <component
                  ref="routerViewRef"
                  :is="Component"
                  @addFile="addFile"
                  @reload="getUseSpace"
                ></component>
            </router-view>
        </div>
      </div>
      <!-- 修改头像 -->
      <UpdateAvatar
        ref="updateAvatarRef"
        @updateAvatar="reloadAvatar"
      ></UpdateAvatar>
      <!-- 修改密码 -->
      <UpdatePassword ref="updatePasswordRef"></UpdatePassword>
  </div>
</template>

<script setup>
import Uploader from "@/views/main/Uploader.vue";
import UpdateAvatar from "./UpdateAvatar.vue";
import UpdatePassword from "./UpdatePassword.vue";

import { ref, reactive, getCurrentInstance, nextTick, watch } from "vue";
import { useRouter, useRoute } from "vue-router";
const { proxy } = getCurrentInstance();
const router = useRouter();
const route = useRoute();

const api = {
    logout: "/logout",
    getUseSpace: "/getUseSpace",
};

//显示上传窗口
const showUploader = ref(false);

// 添加文件
const uploaderRef = ref();
const addFile = (data) => {
  const { file, filePid } = data;
  showUploader.value = true;
  uploaderRef.value.addFile(file, filePid);
};

// 上传文件回调
const routerViewRef = ref();
const uploadCallbackHandler = () => {
    nextTick(() => {
      routerViewRef.value.reload();
      getUseSpace();
    });
};
const timestamp = ref(0);
//获取用户信息
const userInfo = ref(proxy.VueCookies.get("userInfo"));

const menus = [
    {
        icon: "cloude",
        name: "首页",
        menuCode: "main",
        path: "/main/all",
        allShow: true,
        children: [
            {
                icon: "all",
                name: "全部",
                category: "all",
                path: "/main/all",
            },
            {
                icon: "video",
                name: "视频",
                category: "video",
                path: "/main/video",
            },
            {
                icon: "music",
                name: "音频",
                category: "music",
                path: "/main/music",
            },
            {
                icon: "image",
                name: "图片",
                category: "image",
                path: "/main/image",
            },
            {
                icon: "doc",
                name: "文档",
                category: "doc",
                path: "/main/doc",
            },
            {
                icon: "more",
                name: "其他",
                category: "others",
                path: "/main/others",
            },
        ],
    },
    {
        path: "/myshare",
        icon: "share",
        name: "分享",
        menuCode: "share",
        allShow: true,
        children: [
            {
                name: "分享记录",
                path: "/myshare",
            },
        ],
    },
    {
        path: "/recycle",
        icon: "del",
        name: "回收站",
        menuCode: "recycle",
        tips: "回收站为你保存10天内删除的文件",
        allShow: true,
        children: [
            {
                name: "删除的文件",
                path: "/recycle",
            },
        ],
    },
    {
        path: "/settings/fileList",
        icon: "settings",
        name: "设置",
        menuCode: "settings",
        allShow: false,
        children: [
            {
                name: "用户文件",
                path: "/settings/fileList",
            },
            {
                name: "用户管理",
                path: "/settings/userList",
            },
            {
                name: "系统设置",
                path: "/settings/sysSetting",
            },
        ],
    },
];

const currentMenu = ref({});
const currentPath = ref();
const jump = (data) => {
    if (!data.path || data.menuCode == currentMenu.value.menuCode) {
        return;
    }
    router.push(data.path);
};

const setMenu = (menuCode, path) => {
    const menu = menus.find((item) => {
        return item.menuCode === menuCode;
    });
    currentMenu.value = menu;
    currentPath.value = path;
};
watch (
    () => route,
    (newVal, oldVal) => {
        if (newVal.meta.menuCode) {
            setMenu(newVal.meta.menuCode, newVal.path);
        }
    },
    { immediate: true, deep: true }
);

// 修改头像
const updateAvatarRef = ref();
const updateAvatar = () => {
    updateAvatarRef.value.show(userInfo.value);
};

const reloadAvatar = () => {
    userInfo.value = proxy.VueCookies.get("userInfo");
    timestamp.value = new Date().getTime();
};

// 修改密码
const updatePasswordRef = ref();
const updatePassword = () => {
    updatePasswordRef.value.show();
};

// 退出
const logout = async () => {
    proxy.Confirm(`你确定要退出吗`, async () => {
        let result = await proxy.Request({
        url: api.logout,
        });
        if (!result){
            return;
        }
        proxy.VueCookies.remove("userInfo");
        router.push("/login");
    });
};

// 使用空间
const useSpaceInfo = ref({ useSpace: 0, totalSpace: 1 });
const getUseSpace = async () => {
    let result = await proxy.Request({
        url: api.getUseSpace,
        showLoading: false,
    });
    if (!result) {
        return;
    }
    useSpaceInfo.value = result.data;
};

getUseSpace();
</script>

<style lang="scss" scoped>
.framework {
    height: 100vh;
    display: flex;
    flex-direction: column;

    .header {
        height: 60px;
        background: #ffffff;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0 20px;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
        position: relative;
        z-index: 1;

        .logo {
            display: flex;
            align-items: center;
            cursor: pointer;
            padding: 8px 16px;
            border-radius: 12px;
            background: linear-gradient(135deg, rgba(64, 158, 255, 0.1) 0%, rgba(64, 158, 255, 0.05) 100%);
            backdrop-filter: blur(8px);
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            position: relative;
            overflow: hidden;

            &::before {
                content: '';
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                background: linear-gradient(45deg, 
                    rgba(64, 158, 255, 0) 0%,
                    rgba(64, 158, 255, 0.1) 50%,
                    rgba(64, 158, 255, 0) 100%);
                transform: translateX(-100%);
                transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
            }

            &:hover {
                transform: translateY(-2px);
                box-shadow: 0 4px 12px rgba(64, 158, 255, 0.15);

                &::before {
                    transform: translateX(100%);
                }

                .icon-pan {
                    transform: rotate(15deg) scale(1.1);
                    color: #409EFF;
                }

                .name {
                    background: linear-gradient(to right, #409EFF, #40E0D0);
                    -webkit-background-clip: text;
                    color: transparent;
                    letter-spacing: 1px;
                }
            }

            .icon-pan {
                font-size: 28px;
                color: #2c3e50;
                margin-right: 12px;
                transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
                position: relative;

                &::after {
                    content: '';
                    position: absolute;
                    width: 8px;
                    height: 8px;
                    background: #409EFF;
                    border-radius: 50%;
                    top: 0;
                    right: -2px;
                    transform: scale(0);
                    transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
                }
            }

            .name {
                font-size: 20px;
                font-weight: 600;
                color: #2c3e50;
                transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
                position: relative;
                
                &::after {
                    content: '';
                    position: absolute;
                    bottom: -2px;
                    left: 0;
                    width: 100%;
                    height: 2px;
                    background: linear-gradient(to right, #409EFF, transparent);
                    transform: scaleX(0);
                    transform-origin: left;
                    transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
                }
            }

            &:hover {
                .icon-pan::after {
                    transform: scale(1);
                }

                .name::after {
                    transform: scaleX(1);
                }
            }
        }

        .right-panel {
            display: flex;
            align-items: center;
            gap: 20px;
            .icon-transfer {
                cursor: pointer;
            }
            .user-info {
                margin-left: 10px;
                display: flex;
                align-items: center;
                cursor: pointer;
                .avatar {
                    margin: 0px 5px 0px 15px;
                }
                .nick-name {
                    color: #4F4F4F;
                    animation-name: glitched;
                    animation-duration: calc(.9s * 3.4);
                    animation-iteration-count: infinite;
                    animation-timing-function: linear;
                    @keyframes glitched {
                    0% { left: -4px; transform: skew(-20deg); }
                    11% { left: 2px; transform: skew(0deg); }
                    50% { transform: skew(0deg); }
                    51% { transform: skew(10deg); }
                    60% { transform: skew(0deg); }
                    100% { transform: skew(0deg); }
                    }
                    }
            }
        }
    }
    .body {
        display: flex;
        .left-sider {
            border-right: 1px solid #f1f2f4;
            display: flex;
            .menu-list {
                height: calc(100vh - 56px);
                width: 80px;
                border-right: 1px solid #f1f2f4;
                .menu-item {
                    text-align: center;
                    font-size: 14px;
                    font-weight: bold;
                    padding: 20px 0px;
                    cursor: pointer;
                    &:hover {
                        background: #f3f3f3;
                    }
                    .iconfont {
                        font-weight: normal;
                        font-size: 28px;
                    }
                }
                .active {
                    .iconfont {
                        color: #4F4F4F;
                    }
                    .text {
                        color: #4F4F4F;
                    }
                }
            }
            .menu-sub-list {
                width: 200px;
                padding: 20px 10px 0px;
                position: relative;
                .menu-item-sub {
                    text-align: center;
                    line-height: 40px;
                    border-radius: 5px;
                    cursor: pointer;
                    &:hover {
                        background: #f3f3f3;
                    }
                    .iconfont {
                        font-size: 14px;
                        margin-right: 20px;
                    }
                    .text {
                        font-size: 13px;
                    }
                }
                .active {
                    background: #eef9fe;
                    .iconfont {
                        color: #4F4F4F;
                    }
                    .text {
                        color: #4F4F4F;
                    }
                }

                .tips {
                    margin-top: 10px;
                    color: #888888;
                    font-size: 13px;
                }

                .space-info {
                    position: absolute;
                    bottom: 10px;
                    width: 100%;
                    padding: 0px 5px;
                    .percent {
                        padding-right: 10px;
                    }
                    .space-use {
                        margin-top: 5px;
                        color: #7e7e7e;
                        display: flex;
                        justify-content: space-around;
                        .use {
                            flex: 1;
                        }
                        .iconfont {
                            cursor: pointer;
                            margin-right: 20px;
                            color: #4F4F4F;
                        }
                    }
                }
            }
        }
        .body-content {
            flex: 1;
            width: 0;
            padding-left: 20px;
        }
    }
}
</style>
