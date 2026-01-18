<template>
  <v-card>
    <v-card-title class="text-center"> 消息 </v-card-title>

    <!-- 用户信息部分 -->
    <div class="px-4 py-2">
      <!-- 昵称 -->
      <v-card class="mb-3 elevation-4 rounded-lg" subtitle="用户信息">
        <v-list>
          <v-list-item class="text-right" @click="alert_avatar">
            <template #prepend>
              <span>头像</span>
            </template>
            <template #append>
              <v-avatar :image="user.avatar"></v-avatar>
            </template>
          </v-list-item>

          <v-list-item class="text-right" :title="user.email">
            <template #prepend>
              <span>邮箱</span>
            </template>
          </v-list-item>

          <v-list-item class="text-right" @click="editNickname = true" :title="user.nickname">
            <template #prepend>
              <span>昵称</span>
            </template>
            <template #append>
              <v-icon>mdi-chevron-right</v-icon>
            </template>
          </v-list-item>

          <v-list-item class="text-right" @click="editPassword = true" title="(点击更改)" append-icon="mdi-chevron-right">
            <template #prepend>
              <span>密码</span>
            </template>
          </v-list-item>

          <v-list-item class="text-right" @click="checkLogout = true" append-icon="mdi-chevron-right">
            <template #prepend>
              <span>退出登录</span>
            </template>
          </v-list-item>

        </v-list>

      </v-card>
    </div>

    <!-- 用户信息部分 -->
    <div class="px-4 py-2">
      <!-- 昵称 -->
      <v-card class="mb-3 elevation-4 rounded-lg" subtitle="章评互动信息">

    <!-- 互动消息部分 -->
    <v-list v-if="messages.length === 0" density="compact" class="mr-4">
      <v-list-item class="my-4">
        <v-list-item-title class="text-center">无新的互动消息</v-list-item-title>
      </v-list-item>
    </v-list>
    <v-list id="book-comments" density="compact" class="mr-4">
      <template v-for="c in messages" :key="c.id">
        <v-list-item class="pr-0 align-self-start mb-4" :prepend-avatar="c.avatar" :subtitle="c.nickName + ' @《宿命之环》'">
          <div class="my-2">{{ thumb_or_content(c) }}</div>
          <v-card variant="tonal" color="surface-variant" :subtitle="'这一段写得真厉害哦'"></v-card>
        </v-list-item>
      </template>
    </v-list>

    </v-card>
    </div>
    <!-- 修改昵称对话框 -->
    <v-dialog v-model="editAvatar" persistent>
      <template #default>
        <v-alert></v-alert>
      </template>
    </v-dialog>


    <!-- 修改昵称对话框 -->
    <v-dialog v-model="editNickname" persistent>
      <template #default>
        <v-card>
          <v-card-title class="text-center">修改昵称</v-card-title>
          <v-card-text>
            <v-text-field v-model="newNickname" label="新昵称"></v-text-field>
            <v-alert v-if="alert.msg" :type="alert.type" dismissible>{{ alert.msg }}</v-alert>
          </v-card-text>
          <v-card-actions>
            <v-btn text @click="editNickname = false">取消</v-btn>
            <v-btn text @click="saveNickname">保存</v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>

    <!-- 修改密码对话框 -->
    <v-dialog v-model="editPassword" persistent z-index="2999">
      <template #default>
        <v-card>
          <v-card-title class="text-center">修改密码</v-card-title>
          <v-card-text>
            <v-text-field v-model="oldPassword" label="当前密码"></v-text-field>
            <v-text-field v-model="newPassword" label="新密码" :rules="[rules.pass]"></v-text-field>
            <v-text-field v-model="examPassword" label="确认密码" :rules="[double_check_password]"></v-text-field>
            <v-alert v-if="alert.msg" :type="alert.type" dismissible>{{ alert.msg }}</v-alert>
          </v-card-text>
          <v-card-actions>
            <v-btn text @click="editPassword = false">取消</v-btn>
            <v-btn text @click="savePassword">保存</v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>


    <!-- 修改昵称对话框 -->
    <v-dialog v-model="checkLogout" persistent>
      <template #default>
        <v-card>
          <v-card-title class="text-center">请确认</v-card-title>
          <v-card-text>
            是否要退出登录？
            <v-alert v-if="alert.msg" :type="alert.type" dismissible>{{ alert.msg }}</v-alert>
          </v-card-text>
          <v-card-actions>
            <v-btn text @click="checkLogout = false">取消</v-btn>
            <v-btn text @click="do_logout">确认</v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>


</v-card>
</template>

<script>
export default {
  name: 'UserCenter',
  props: ['messages', 'user'],
  data: () => ({
    editAvatar: false,
    editNickname: false,
    editPassword: false,
    checkLogout: false,
    alert: {
      msg: '',
      type: ''
    },

    newNickname: '',
    oldPassword: '',
    newPassword: '',
    examPassword: '',
    rules: {
      pass: v => (20 >= v.length && v.length >= 8) || '8 ~ 20 characters',
      nick: v => v.length >= 2 || 'Min 2 characters',
      email: function (email) {
        var re = /^(([^<>()[\.,;:@"]+([^<>()[\.,;:@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
        return re.test(email) || "Invalid email format";
      },
    },
  }),
  watch: {
    // 监听修改昵称弹窗关闭事件，清空输入框内容
    editNickname(newVal) {
      if (!newVal) {
        this.newNickname = '';
        this.alert.msg = '';
      }
    },
    // 监听修改密码弹窗关闭事件，清空输入框内容
    editPassword(newVal) {
      if (!newVal) {
        this.oldPassword = '';
        this.newPassword = '';
        this.examPassword = '';
        this.alert.msg = '';
      }
    },
    // 监听退出登录弹窗关闭事件，清空错误信息
    checkLogout(newVal) {
      if (!newVal) {
        this.alert.msg = '';
      }
    },
  },
  methods: {
    thumb_or_content: function (c) {
      if (Math.random() > 0.5) {
        return c.content;
      } else {
        return "赞了你的评论";
      }
    },
    double_check_password: function (v) {
      if (v.length < 8) {
        return 'Min 8 characters';
      }
      return v == this.newPassword || "Password are not same."
    },
    alert_avatar: function () {
      alert('请前往 https://cavatar.cn 更改')
    },

    saveNickname: function () {
      // 清除之前的错误信息
      this.alert.msg = "";
      this.update_user({
        nickname: this.newNickname,
      }).then(() => {
        this.editNickname = false;
      }).catch(() => {
        // 捕获错误，不关闭弹窗
        console.log("修改昵称失败");
      })
    },
    savePassword() {
      if (this.examPassword != this.newPassword) {
        this.alert.msg = "两次输入的密码不一致";
        this.alert.type = "error";
        return;
      }
      // 清除之前的错误信息
      this.alert.msg = "";
      this.update_user({
        password0: this.oldPassword,
        password1: this.newPassword,
      }).then(() => {
        // 只有当update_user成功时才会执行这里
        this.editPassword = false;
      }).catch(() => {
        // 捕获错误，不关闭弹窗
        console.log("修改密码失败");
      })
    },
    update_user: function (new_info) {
      this.user.nickName = this.newNickname;
      return this.$backend(`/api/user/update`, {
        method: 'POST',
        body: JSON.stringify(new_info),
      }).then((rsp) => {
        if (rsp.err != 'ok') {
          this.alert.msg = rsp.msg;
          this.alert.type = "error";
          throw new Error(rsp.msg);
        }
        this.$emit('update', rsp.data);
      });
    },
    do_logout: function () {
      // 清除之前的错误信息
      this.alert.msg = "";
      return this.$backend(`/api/user/sign_out`).then((rsp) => {
        if (rsp.err != 'ok') {
          this.alert.msg = rsp.msg;
          this.alert.type = "error";
          throw new Error(rsp.msg);
        }
        this.$emit('logout');
        this.checkLogout = false;
      }).catch(() => {
        // 捕获错误，不关闭弹窗
        console.log("退出登录失败");
      });
    },
  },
};
</script>

<style scoped>
.v-card {
  transition: box-shadow 0.3s ease;
}

.v-card:hover {
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.v-btn {
  text-transform: none;
}
</style>
