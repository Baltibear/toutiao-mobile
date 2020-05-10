<template>
  <div class="login-container">
    <van-nav-bar
  class="app-nav-bar"
  title="登录"
  left-arrow
  @click-left="$router.back()"
/>

<van-form @submit="onLogin" :show-error="false" :show-error-message="false" validate-first @failed="onFailed" ref="login-form">
  <van-field
    v-model="user.mobile"
    icon-prefix="icon"
    left-icon="shouji"
    center
    placeholder="请输入手机号"
    name="mobile"
    :rules="formRules.mobile"
  />
  <van-field
    v-model="user.code"
    clearable
    icon-prefix="icon"
    left-icon="yanzhengma"
    center
    placeholder="请输入验证码"
    name="code"
    :rules="formRules.code"
>
 <template #button>
    <van-count-down v-if="isCountDownShow" :time="1000 * 60" format="ss s" @finish="isCountDownShow = false" />
    <van-button v-else class="send-btn" size="small" round @click.prevent="onSendSms" :loading="isSendSmsLoading" >发送验证码</van-button>
  </template>
</van-field>
<div class="login-btn-wrap">
  <van-button class="login-btn" type="info" block >登录</van-button>
</div>
</van-form>
  </div>
</template>

<script>
import { login, sendSms } from '@/api/user'
import { Toast } from 'vant'

export default {
  name: 'LoginIndex',
  components: {},
  props: {},
  data () {
    return {
      user: {
        mobile: '',
        code: ''
      },
      formRules: {
        mobile: [
          { required: true, message: '请填写手机号' },
          { pattern: /^1[3|5|7|8|9]\d{9}$/, message: '手机号格式错误' }
        ],
        code: [
          { required: true, message: '请填写验证码' },
          { pattern: /^\d{6}$/, message: '验证码错误' }
        ]
      },
      isCountDownShow: false,
      isSendSmsLoading: false
    }
  },
  computed: {},
  watch: {},
  created () {},
  mounted () {},
  methods: {
    async onLogin () {
      Toast.loading({
        message: '登录中...',
        forbidClick: true,
        duration: 0
      })
      try {
        const res = await login(this.user)
        console.log(res)
        Toast.success('登录成功')
      } catch (err) {
        console.log(err)
        Toast.fail('登陆失败，手机号或验证码错误')
      }
    },

    onFailed (error) {
      if (error.errors[0]) {
        this.$toast({
          message: error.errors[0].message,
          position: 'top'
        })
      }
    },

    async onSendSms () {
      try {
        await this.$refs['login-form'].validate('mobile')
        this.isSendSmsLoading = true
        await sendSms(this.user.mobile)
        this.isCountDownShow = true
      } catch (err) {
        let message = ''
        if (err && err.response && err.response.status === 429) {
          message = '发送太频繁，请稍后重试'
        } else if (err.name === 'mobile') {
          message = err.message
        } else {
          message = '发送失败'
        }
        this.$toast({
          message,
          position: 'top'
        })
      }
      this.isSendSmsLoading = false
    }
  }
}
</script>
<style scoped lang="less">
  .login-container{
    .send-btn{
      width: 82px;
      height: 23px;
      background-color: #ededed;
      .van-button__text {
        font-size: 11px;
        color: #666666;
      }
    }
    .login-btn-wrap{
      padding: 26px 16px;
      .login-btn {
        background-color: #6db4fb;
        border: none;
        .van-button__text {
          font-size: 15px;
        }
      }
    }
  }
</style>
