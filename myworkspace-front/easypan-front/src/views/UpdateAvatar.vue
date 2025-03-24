<template>
    <div>
        <Dialog
          :show="dialogConfig.show"
          :title="dialogConfig.title"
          :buttons="dialogConfig.buttons"
          width="500px"
          :showCancel="false"
          @close="dialogConfig.show = false"
          class="avatar-dialog"
        >
        <el-form
          :model="formData"
          ref="formDataRef"
          label-width="80px"
          @submit.prevent
          class="avatar-form"
        >
        <el-form-item label="昵称" class="nickname-item">
            <span class="nickname-text">{{ formData.nickName }}</span>
        </el-form-item>

        <el-form-item label="头像" class="avatar-item">
          <AvatarUpload v-model="formData.avatar"></AvatarUpload>
        </el-form-item>
       </el-form>
      </Dialog>
    </div>
</template>

<script setup>
import AvatarUpload from "@/components/AvatarUpload.vue";
import { ref, reactive, getCurrentInstance, nextTick } from "vue";
const { proxy } = getCurrentInstance();

const api = {
    updateUserAvatar: "updateUserAvatar",
};

const formData = ref({});
const formDataRef = ref();

const show = (data) => {
    formData.value = Object.assign({}, data);
    formData.value.avatar = { userId: data.userId, qqAvatar: data.avatar };
    dialogConfig.value.show = true;
};

defineExpose({ show });

const dialogConfig = ref({
    show: false,
    title: "修改头像",
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

const emit = defineEmits();
const submitForm = async () => {
    if (!(formData.value.avatar instanceof File)) {
        dialogConfig.value.show = false;
        return;
    }

    let result = await proxy.Request ({
        url: api.updateUserAvatar,
        params: {
            avatar: formData.value.avatar,
        },
    });
    if (!result) {
        return;
    }
    dialogConfig.value.show = false;
    const cookieUserInfo = proxy.VueCookies.get("userInfo");
    delete cookieUserInfo.avatar;
    proxy.VueCookies.set("userInfo", cookieUserInfo, 0);
    emit("updateAvatar");
};
</script>

<style lang="scss" scoped>
.avatar-dialog {
    :deep(.el-dialog__body) {
        padding: 30px 20px;
    }
}

.avatar-form {
    .nickname-item {
        margin-bottom: 30px;
        
        .nickname-text {
            font-size: 1.1rem;
            color: #333;
            font-weight: 500;
        }
    }
    
    .avatar-item {
        margin-bottom: 0;
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
