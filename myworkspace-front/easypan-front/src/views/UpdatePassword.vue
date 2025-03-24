<template>
    <div>
        <Dialog
          :show="dialogConfig.show"
          :title="dialogConfig.title"
          :buttons="dialogConfig.buttons"
          width="500px"
          :showCancel="false"
          @close="dialogConfig.show = false"
          class="password-dialog"
        >
          <el-form
            :model="formData"
            :rules="rules"
            ref="formDataRef"
            label-width="80px"
            @submit.prevent
            class="password-form"
          >
            <div class="form-title">修改密码</div>
            <div class="form-tips">密码要求：8-18位，包含数字和字母</div>
            
            <!-- 修改密码 -->
            <el-form-item label="新密码" prop="password" class="password-item">
                <el-input
                  type="password"
                  size="large"
                  placeholder="请输入新密码"
                  v-model.trim="formData.password"
                  show-password
                >
                <template #prefix>
                    <span class="iconfont icon-password"></span>
                </template>
              </el-input>
            </el-form-item>
            <el-form-item label="确认密码" prop="rePassword" class="password-item">
                <el-input
                  type="password"
                  size="large"
                  placeholder="请再次输入新密码"
                  v-model.trim="formData.rePassword"
                  show-password
                >
                <template #prefix>
                    <span class="iconfont icon-password"></span>
                </template>
              </el-input>
            </el-form-item>
          </el-form>
        </Dialog>
    </div>
</template>

<script setup>
import { ref, reactive, getCurrentInstance, nextTick } from "vue";
const { proxy } = getCurrentInstance();

const api = {
    updatePassword: "updatePassword"
}

const checkRePassword = (rule, value, callback) => {
    if (value !== formData.value.password) {
        callback(new Error(rule.message));
    } else {
        callback();
    }
};

const formData = ref({});
const formDataRef = ref();
const rules = {
    password: [
        { required: true, message: "请输入新密码" },
        {
            validator: proxy.Verify.password,
            message: "密码只能是数字,字母,特殊字符8-18位",
        },
    ],
    rePassword: [
        { required: true, message: "请再次输入新密码" },
        {
            validator: checkRePassword,
            message: "两次输入的密码不一致",
        },
    ],
};

const show = () => {
    dialogConfig.value.show = true;
    nextTick(() => {
        formDataRef.value.resetFields();
        formData.value = {};
    });
};

defineExpose({ show });

const dialogConfig = ref({
    show: false,
    title: "修改密码",
    buttons: [
        {
            type: "primary",
            text: "确定",
            click: (e) => {
                submitForm();
            },
            class: "confirm-button",
        },
    ],
});

const submitForm = async () => {
    formDataRef.value.validate(async (valid) => {
        if (!valid) {
            return;
        }
        let result = await proxy.Request({
            url: api.updatePassword,
            params: {
                password: formData.value.password,
            },
        });

        if (!result) {
            return;
        }
        dialogConfig.value.show = false;
        proxy.Message.success("密码修改成功");
    });
};
</script>

<style lang="scss" scoped>
.password-dialog {
    :deep(.el-dialog__body) {
        padding: 30px 20px;
    }
}

.password-form {
    .form-title {
        font-size: 1.5rem;
        font-weight: 600;
        color: #333;
        margin-bottom: 0.5rem;
        text-align: center;
    }
    
    .form-tips {
        font-size: 0.875rem;
        color: #666;
        text-align: center;
        margin-bottom: 2rem;
    }
    
    .password-item {
        margin-bottom: 1.5rem;
        
        :deep(.el-input) {
            --el-input-height: 50px;
            
            .el-input__wrapper {
                background: #f5f7fa;
                border-radius: 8px;
                border: 1px solid #e4e7ed;
                transition: all 0.3s ease;
                
                &:hover, &.is-focus {
                    border-color: #409eff;
                    box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.1);
                }
            }
            
            .iconfont {
                font-size: 1.25rem;
                color: #909399;
            }
        }
    }
}

.confirm-button {
    background-color: #409eff;
    border-color: #409eff;
    color: #fff;
    padding: 8px 24px;
    border-radius: 4px;
    transition: all 0.3s ease;
    
    &:hover {
        background-color: #66b1ff;
        border-color: #66b1ff;
        transform: translateY(-1px);
        box-shadow: 0 2px 8px rgba(64, 158, 255, 0.2);
    }
}
</style>
