<template>
    <div class="share">
        <div class="bg"></div>
        <div class="body-content">
            <div class="share-panel">
                <div class="share-title">分享文件</div>
                <div class="file-info">
                    <div class="avatar">
                        <Avatar
                          :userId="shareInfo.userId"
                          :avatar="shareInfo.avatar"
                          :width="50"
                        ></Avatar>
                    </div>
                    <div class="share-info">
                        <div class="user-info">
                            <span class="nick-name">{{ shareInfo.nickName }}</span>
                            <span class="share-time">分享于 {{ shareInfo.shareTime }}</span>
                        </div>
                        <div class="file-name">分享文件：{{ shareInfo.fileName }}</div>
                    </div>
                </div>
                <div class="code-body">
                    <div class="tips">请输入提取码</div>
                    <div class="input-area">
                        <el-form
                          :model="formData"
                          :rules="rules"
                          ref="formDataRef"
                          @submit.prevent
                        >
                          <el-form-item prop="code">
                            <el-input
                              class="input"
                              clearable
                              v-model.trim="formData.code"
                              @keyup.enter="checkShare"
                              placeholder="请输入5位提取码"
                            >
                                <template #prefix>
                                    <span class="iconfont icon-checkcode"></span>
                                </template>
                            </el-input>
                            <el-button class="get-btn" type="primary" @click="checkShare">提取文件</el-button>
                          </el-form-item>
                        </el-form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, reactive, getCurrentInstance, nextTick } from "vue";
const { proxy } = getCurrentInstance();
import { useRouter, useRoute } from "vue-router";
const router = useRouter();
const route = useRoute();

const api = {
    getShareInfo: "/showShare/getShareInfo",
    checkShareCode: "/showShare/checkShareCode",
};

const shareId = route.params.shareId;
const shareInfo = ref({});
const getShareInfo = async () => {
    let result = await proxy.Request({
        url: api.getShareInfo,
        params: {
            shareId,
        },
    });
    if (!result) {
        return;
    }
    shareInfo.value = result.data;
};
getShareInfo();

const formData = ref({});
const formDataRef = ref();
const rules = {
    code: [
        { required: true, message: "请输入提取码" },
        { min: 5, message: "提取码为5位" },
        { max: 5, message: "提取码为5位" },
    ],
};

const checkShare = async () => {
    formDataRef.value.validate(async (valid) => {
        if (!valid) {
            return;
        }
        let params = {};
        Object.assign(params, formData.value);
        let result = await proxy.Request({
            url: api.checkShareCode,
            params: {
                shareId: shareId,
                code: formData.value.code,
            },
        });
        if (!result) {
            return;
        }
        router.push(`/share/${shareId}`);
    });
};
</script>

<style lang="scss" scoped>
.share {
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    position: relative;
    overflow: hidden;

    .bg {
        position: absolute;
        width: 100%;
        height: 100%;
        filter: blur(10px);
        transform: scale(1.1);
        z-index: 0;
        background: inherit;
        
        &::after {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.3);
        }
    }

    .body-content {
        position: relative;
        z-index: 1;
        width: 500px;
        padding: 30px;
        background: rgba(255, 255, 255, 0.95);
        border-radius: 15px;
        box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
        backdrop-filter: blur(10px);
        transition: all 0.3s ease;

        &:hover {
            transform: translateY(-5px);
            box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
        }

        .share-panel {
            .share-title {
                font-size: 24px;
                font-weight: 600;
                color: #333;
                text-align: center;
                margin-bottom: 25px;
            }

            .file-info {
                padding: 20px;
                background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                border-radius: 10px;
                color: #fff;
                display: flex;
                align-items: center;
                margin-bottom: 25px;
                
                .avatar {
                    margin-right: 15px;
                    border-radius: 50%;
                    overflow: hidden;
                    border: 2px solid rgba(255, 255, 255, 0.3);
                    transition: all 0.3s ease;

                    &:hover {
                        transform: scale(1.05);
                        border-color: rgba(255, 255, 255, 0.8);
                    }
                }
                
                .share-info {
                    flex: 1;
                    
                    .user-info {
                        display: flex;
                        align-items: center;
                        margin-bottom: 8px;
                        
                        .nick-name {
                            font-size: 16px;
                            font-weight: 500;
                            color: #fff;
                        }
                        
                        .share-time {
                            margin-left: 15px;
                            font-size: 13px;
                            color: rgba(255, 255, 255, 0.9);
                        }
                    }
                    
                    .file-name {
                        font-size: 14px;
                        color: rgba(255, 255, 255, 0.9);
                    }
                }
            }

            .code-body {
                .tips {
                    font-size: 16px;
                    color: #666;
                    margin-bottom: 15px;
                    text-align: center;
                }

                .input-area {
                    :deep(.el-form-item) {
                        margin-bottom: 0;
                    }

                    :deep(.el-input) {
                        .el-input__wrapper {
                            border-radius: 8px;
                            padding: 8px 15px;
                            box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
                            transition: all 0.3s ease;

                            &:hover, &.is-focus {
                                box-shadow: 0 4px 16px rgba(102, 126, 234, 0.15);
                            }
                        }

                        .el-input__inner {
                            font-size: 15px;
                            letter-spacing: 2px;
                        }

                        .iconfont {
                            color: #667eea;
                            font-size: 18px;
                        }
                    }

                    .get-btn {
                        margin-top: 20px;
                        width: 100%;
                        height: 44px;
                        font-size: 16px;
                        border-radius: 8px;
                        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                        border: none;
                        transition: all 0.3s ease;

                        &:hover {
                            transform: translateY(-2px);
                            box-shadow: 0 4px 16px rgba(102, 126, 234, 0.3);
                        }

                        &:active {
                            transform: translateY(0);
                        }
                    }
                }
            }
        }
    }
}

// 动画效果
@keyframes fadeInUp {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.share-panel {
    animation: fadeInUp 0.6s ease-out;
}
</style>
