<template>
    <div class="login-body">
        <div class="bg"></div>
        <div class="login-panel">
            <el-form
            class="login-register"
                :model="formData"
                :rules="rules"
                ref="formDataRef"
                @submit.prevent
            >
            <div class="login-title">个人云盘</div>
            <!-- input输入 -->
            <el-form-item prop="email" >
                <el-input
                size="large"
                clearable
                placeholder="请输入邮箱"
                v-model.trim="formData.email"
                maxLength="150"
                >
                <template #prefix>
                    <span class="iconfont icon-account"></span>
                </template>
            </el-input>
            </el-form-item>
            <!-- 登录密码 -->
            <el-form-item prop="password" v-if="opType == 1">
                <el-input
                    type="password"
                    size="large"
                    placeholder="请输入密码"
                    v-model.trim="formData.password"
                    show-password
                    >
                    <template #prefix>
                        <span class="iconfont icon-password"></span>
                    </template>
                </el-input>
            </el-form-item>
            <!-- 注册 -->
            <div v-if="opType == 0 || opType == 2">
                <el-form-item prop="emailCode">
                    <div class="send-email-panel">
                        <el-input
                        size="large"
                        placeholder="请输入邮箱验证码"
                        v-model.trim="formData.emailCode"
                    >
                        <template #prefix>
                            <span class="iconfont icon-checkcode"></span>
                        </template>
                    </el-input>
                    <el-button class="send-mail-btn" type="primary" size="large" @click="getEmailCode">
                        获取验证码
                    </el-button>
                    </div>
                    <el-popover placement="left" :width="500" trigger="click">
                        <div>
                            <p>1.在垃圾箱中查找邮箱验证码</p>
                            <p>2.在邮箱中头像->设置->反垃圾->白名单->设置邮件地址白名单</p>
                            <p>3.将邮箱【3503201604@qq.com】添加到白名单不知道怎么设置?</p>
                        </div>
                        <template #reference>
                            <span class="a-link" :style="{ 'font-size': '14px' }"
                            >未收到邮箱验证码?</span>
                        </template>
                    </el-popover>
                </el-form-item>
                <!-- 昵称 -->
                <el-form-item prop="nickName" v-if="opType == 0">
                    <el-input
                        size="large"
                        placeholder="请输入昵称"
                        v-model.trim="formData.nickName"
                        maxLength="20"
                    >
                        <template #prefix>
                            <span class="iconfont icon-account"></span>
                        </template>
                    </el-input>
                </el-form-item>
                <!-- 注册密码, 找回密码 -->
            <el-form-item prop="registerPassword">
                <el-input
                    type="password"
                    size="large"
                    placeholder="请输入密码"
                    v-model.trim="formData.registerPassword"
                    show-password
                >
                    <template #prefix>
                        <span class="iconfont icon-password"></span>
                    </template>
                </el-input>
            </el-form-item>
            <!-- 重复密码 -->
            <el-form-item prop="reRegisterPassword">
                <el-input
                    type="password"
                    size="large"
                    placeholder="请再次输入密码"
                    v-model.trim="formData.reRegisterPassword"
                    show-password
                >
                    <template #prefix>
                        <span class="iconfont icon-password"></span>
                    </template>
                </el-input>
            </el-form-item>
            </div>
            <!-- 验证码 -->
            <el-form-item prop="checkCode">
                <div class="check-code-panel">
                <el-input
                size="large"
                placeholder="请输入验证码"
                v-model.trim="formData.checkCode"
                @keyup.enter="doSubmit"
                >
                <template #prefix>
                    <span class="iconfont icon-checkcode"></span>
                </template>
              </el-input>
              <img
                :src="checkCodeUrl"
                class="check-code"
                @click="changeCheckCode(0)"
               />
             </div>
            </el-form-item>
            <!-- 登录 -->
            <el-form-item v-if="opType == 1" class="login-options">
                <div class="left-options">
                    <el-checkbox v-model="formData.rememberMe" class="remember-checkbox">记住我</el-checkbox>
                </div>
                <div class="right-options">
                    <a href="javascript:void(0)" class="text-btn" @click="showPanel(2)">忘记密码?</a>
                    <a href="javascript:void(0)" class="text-btn" @click="showPanel(0)">没有账号?</a>
                </div>
            </el-form-item>
            <!-- 找回密码 -->
            <el-form-item v-if="opType == 2">
                <div class="single-link">
                    <a href="javascript:void(0)" class="text-btn" @click="showPanel(1)">去登录?</a>
                </div>
            </el-form-item>
            <el-form-item v-if="opType == 0">
                <div class="single-link">
                    <a href="javascript:void(0)" class="text-btn" @click="showPanel(1)">已有账号?</a>
                </div>
            </el-form-item>
            <!-- 登录按钮 -->
            <el-form-item>
                <el-button
                  type="primary"
                  class="op-btn"
                  size="large"
                  @click="doSubmit"
                >
                    <span v-if="opType == 0">注册</span>
                    <span v-if="opType == 1">登录</span>
                    <span v-if="opType == 2">重置密码</span>
                </el-button>
            </el-form-item>
            <div class="login-btn-qq" v-if="opType == 1">
                快捷登录<img src="@/assets/qq.png" @click="qqLogin" />
            </div>
          </el-form>
        </div>
        <Dialog
            :show="dialogConfig4SendMailCode.show"
            :title="dialogConfig4SendMailCode.title"
            :buttons="dialogConfig4SendMailCode.buttons"
            width="500px"
            :showCancel="false"
            @close="dialogConfig4SendMailCode.show = false"
        >
        <el-form
            :model="formData4SendMailCode"
            :rules="rules"
            ref="formData4SendMailCodeRef"
            label-width="80px"
            @submit.prevent
        >
            <!-- 邮箱 -->
            <el-form-item label="邮箱">
                {{ formData.email }}
            </el-form-item>
            <!-- 邮箱验证码 -->
            <el-form-item label="验证码" prop="checkCode">
                <div class="check-code-panel">
                <el-input
                size="large"
                placeholder="请输入验证码"
                v-model.trim="formData4SendMailCode.checkCode"
                >
                <template #prefix>
                    <span class="iconfont icon-checkcode"></span>
                </template>
              </el-input>
              <img
                :src="checkCodeUrl4SendMailCode"
                class="check-code"
                @click="changeCheckCode(1)"
               />
             </div>
            </el-form-item>
          </el-form>
        </Dialog>
    </div>
</template>

<script setup>
import { ref, reactive, getCurrentInstance, nextTick, onMounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import md5 from "js-md5";
const { proxy } = getCurrentInstance();
const router = useRouter();
const route = useRoute();
const api = {
    checkCode: "/api/checkCode",
    sendEmailCode: "/sendEmailCode",
    register: "/register",
    login: "/login",
    resetPwd: "/resetPwd",
    qqlogin: "/qqlogin",
};

// 操作类型 0:注册 1:登录 2:重置密码
const opType = ref(1);
const showPanel = (type) => {
    opType.value = type;
    restForm();
};

onMounted(() => {
    showPanel(1);
});

const checkRePassword = (rule, value, callback) => {
    if ( value !== formData.value.registerPassword ) {
        callback(new Error(rule.message));
    } else {
        callback();
    }
};
const formData = ref({});
const formDataRef = ref();
const rules = {
    email: [
        { required: true, message: "请输入正确的邮箱" },
        { validator: proxy.Verify.email, message: "请输入正确的邮箱" },
        ],
        password: [{ required: true, message: "请输入密码" }],
        emailCode: [{ required: true, message: "请输入邮箱验证码" }],
        nickName: [{ required: true, message: "请输入昵称" }],
        registerPassword: [
            { required: true, message: "请输入密码" },
            { 
                validator: proxy.Verify.password,
                message: "密码只能是数字, 字母， 特殊字符8-18位",
            },
        ],
        reRegisterPassword: [
            { required: true, message: "请再次输入密码" },
            {
                validator: checkRePassword,
                message: "两次输入的密码不一致",
            },
        ],
        checkCode: [{ required: true, message: "请输入图片验证码" }],
};

const checkCodeUrl = ref(api.checkCode);
const checkCodeUrl4SendMailCode = ref(api.checkCode)
const changeCheckCode = (type) => {
    if (type == 0) {
      checkCodeUrl.value = 
        api.checkCode + "?type=" + type + "&time=" + new Date().getTime();
    } else {
        checkCodeUrl4SendMailCode.value = 
        api.checkCode + "?type=" + type + "&time=" + new Date().getTime();
    }
};

// 发送邮箱验证码
const formData4SendMailCode = ref({});
const formData4SendMailCodeRef = ref();

const dialogConfig4SendMailCode = reactive({
    show: false,
    title: "发送邮箱验证码",
    buttons: [
        {
            type: "primary",
            text: "发送验证码",
            click: (e) => {
                sendEmailCode();
            },
        },
    ],
});
const getEmailCode = () => {
    formDataRef.value.validateField("email", (valid) => {
        if (!valid) {
            return;
        }
        dialogConfig4SendMailCode.show = true;
        nextTick( () => {
            changeCheckCode(1);
            formData4SendMailCodeRef.value.resetFields();
            formData4SendMailCode.value = {
                email: formData.value.email,
            };
        });
    });
};
// 发送邮箱验证码
const sendEmailCode = ()=> {
    formData4SendMailCodeRef.value.validate(async (valid) => {
        if (!valid) {
            return;
        }
        const params = Object.assign({}, formData4SendMailCode.value);
        params.type = opType.value == 0 ? 0 : 1;
        let result = await proxy.Request ({
            url: api.sendEmailCode,
            params: params,
            errorCallback: () => {
                changeCheckCode(1);
            },
        });
        if (!result) {
            return;
        }
        proxy.Message.success("验证码发送成功,请登录邮箱查看");
        dialogConfig4SendMailCode.show = false;
    });
};

// 重置表单
const restForm = () => {
    changeCheckCode(0);
    formDataRef.value.resetFields();
    formData.value = {};
    // 登录
    if (opType.value == 1) {
        const cookieLoginInfo = proxy.VueCookies.get("loginInfo");
        if (cookieLoginInfo) {
            formData.value = cookieLoginInfo;
        }
    }
};
// 登录、注册、重置密码、提交表单
const doSubmit = ()=> {
    formDataRef.value.validate(async (valid) => {
        if (!valid) {
            return;
        }
        let params = {};
        Object.assign(params, formData.value);
        // 注册
        if (opType.value == 0 || opType.value == 2) {
            params.password = params.registerPassword;
            delete params.registerPassword;
            delete params.reRegisterPassword;
        }
        // 登录
        if (opType.value == 1) {
            let cookieLoginInfo = proxy.VueCookies.get("loginInfo");
            let cookiePassword = cookieLoginInfo == null ? null : cookieLoginInfo.password;
            if (params.password !== cookiePassword) {
                params.password = md5(params.password);
            }
        }
        let url = null;
        if (opType.value == 0) {
            url = api.register;
        } else if (opType.value == 1) {
            url = api.login;
        } else if (opType.value == 2) {
            URL = api.resetPwd;
        }
        let result = await proxy.Request({
            url: url,
            params: params,
            errorCallback: () => {
                changeCheckCode(0);
            }
        })
        if (!result) {
            return;
        }
        // 注册返回
        if (opType.value == 0) {
            proxy.Message.success("注册成功,请登录");
            showPanel(1);
        } else if (opType.value == 1) {
            if (params.rememberMe) {
                const loginInfo = {
                    email: params.email,
                    password: params.password,
                    rememberMe: params.rememberMe,
                };
                proxy.VueCookies.set("loginInfo", loginInfo, "7d");
            } else {
                proxy.VueCookies.remove("loginInfo");
            }
            proxy.Message.success("登录成功");
            // 存储cookie
            proxy.VueCookies.set("userInfo", result.data, 0);
            // 重定向到原始页面
            const redirectUrl = route.query.redirectUrl || "/";
            router.push(redirectUrl);
        } else if (opType.value == 2) {
            // 重置密码
            proxy.Message.success("重置密码成功,请登录");
            showPanel(1);
        }
    });
};

// qq登录
const qqLogin = async () => {
    let result = await proxy.Request ({
        url: api.qqlogin,
        params: {
            callbackUrl: route.query.redirectUrl || "",
        },
    });
    if (!result) {
        return;
    }
    proxy.VueCookies.remove("userInfo");
    document.location.href = result.data;
};
</script>

<style lang="scss" scoped>
.login-body {
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
        //background: url('@/assets/login_bg.png') center/cover;
        filter: blur(10px);
        transform: scale(1.1);
        z-index: 0;
        
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

    .login-panel {
        position: relative;
        z-index: 1;
        width: 420px;
        padding: 40px;
        background: rgba(255, 255, 255, 0.9);
        backdrop-filter: blur(10px);
        border-radius: 20px;
        box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
        transform: translateY(0);
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

        &:hover {
            transform: translateY(-5px);
            box-shadow: 0 30px 60px -12px rgba(0, 0, 0, 0.35);
        }

        .login-title {
            font-size: 28px;
            font-weight: 600;
            color: #2c3e50;
            text-align: center;
            margin-bottom: 30px;
            background: linear-gradient(to right, #667eea, #764ba2);
            -webkit-background-clip: text;
            color: transparent;
            position: relative;
            
            &::after {
                content: '';
                position: absolute;
                bottom: -10px;
                left: 50%;
                transform: translateX(-50%);
                width: 50px;
                height: 3px;
                background: linear-gradient(to right, #667eea, #764ba2);
                border-radius: 3px;
            }
        }

        .el-input {
            --el-input-height: 50px;
            margin-bottom: 20px;

            .el-input__wrapper {
                background: rgba(255, 255, 255, 0.8);
                border-radius: 12px;
                border: 1px solid rgba(0, 0, 0, 0.1);
                transition: all 0.3s ease;

                &:hover, &.is-focus {
                    border-color: #667eea;
                    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
                }
            }

            .iconfont {
                font-size: 20px;
                color: #667eea;
            }
        }

        .send-email-panel {
            display: flex;
            gap: 10px;

            .send-mail-btn {
                flex-shrink: 0;
                background: linear-gradient(to right, #667eea, #764ba2);
                border: none;
                border-radius: 12px;
                transition: all 0.3s ease;

                &:hover {
                    transform: translateY(-2px);
                    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
                }
            }
        }

        .check-code-panel {
            display: flex;
            gap: 10px;
            align-items: center;

            .check-code {
                height: 50px;
                border-radius: 12px;
                cursor: pointer;
                transition: all 0.3s ease;

                &:hover {
                    transform: scale(1.05);
                }
            }
        }

        .login-options {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin: 0 0 20px;
            padding: 0 2px;
            position: relative;

            .left-options {
                .remember-checkbox {
                    :deep(.el-checkbox__input) {
                        .el-checkbox__inner {
                            width: 16px;
                            height: 16px;
                            border-color: #667eea;
                            background-color: transparent;
                            
                            &::after {
                                height: 8px;
                                left: 5px;
                                top: 2px;
                            }
                        }
                        
                        &.is-checked .el-checkbox__inner {
                            background-color: #667eea;
                            border-color: #667eea;
                        }
                    }
                    
                    :deep(.el-checkbox__label) {
                        color: #666;
                        font-size: 14px;
                    }
                }
            }

            .right-options {
                display: flex;
                gap: 24px;
                margin-left: auto;
                padding-left: 40px;

                .text-btn {
                    color: #667eea;
                    font-size: 14px;
                    text-decoration: none;
                    position: relative;
                    
                    &:first-child::after {
                        content: '';
                        position: absolute;
                        right: -12px;
                        top: 50%;
                        transform: translateY(-50%);
                        width: 1px;
                        height: 12px;
                        background-color: #ddd;
                    }

                    &:hover {
                        color: #764ba2;
                    }
                }
            }
        }

        .single-link {
            text-align: right;
            margin: -10px 0 15px;

            .text-btn {
                color: #667eea;
                font-size: 14px;
                text-decoration: none;

                &:hover {
                    color: #764ba2;
                }
            }
        }

        .op-btn {
            width: 100%;
            height: 50px;
            background: linear-gradient(to right, #667eea, #764ba2);
            border: none;
            border-radius: 12px;
            font-size: 16px;
            font-weight: 600;
            letter-spacing: 1px;
            transition: all 0.3s ease;

            &:hover {
                transform: translateY(-2px);
                box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
            }
        }

        .login-btn-qq {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
            margin-top: 20px;
            color: #666;
            font-size: 14px;

            img {
                width: 24px;
                height: 24px;
                cursor: pointer;
                transition: all 0.3s ease;

                &:hover {
                    transform: scale(1.1) rotate(360deg);
                }
            }
        }
    }
}

// 弹窗样式优化
:deep(.el-dialog) {
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);

    .el-dialog__header {
        margin: 0;
        padding: 20px;
        background: linear-gradient(to right, #667eea, #764ba2);
        
        .el-dialog__title {
            color: white;
            font-weight: 600;
        }
    }

    .el-dialog__body {
        padding: 30px;
    }

    .el-form-item__label {
        font-weight: 500;
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

.login-panel {
    animation: fadeInUp 0.6s ease-out;
}
</style>
