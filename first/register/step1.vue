<template lang="pug">
.step1
  group
    x-input(:value.sync='phone', placeholder='输入手机号码', type='number', is-type='china-mobile', v-ref:phone)
    .inline
      .div-line
      .inline-input
        x-input(:value.sync='code', placeholder='输入短信验证码')
      .vline
      button.inline-btn(:style='codeStyle', @click='sendCode') {{ codeHint }}
      .clear
    x-input(:value.sync='password', type='password', placeholder='输入密码, 长度不小于6位')
  group
    x-button.blue(@click='step1') 下一步
</template>

<script>
import Group from 'vux-components/group'
import XInput from 'vux-components/x-input'
import XButton from 'vux-components/x-button'
import services from 'utils/services'

export default {
  components: {
    Group,
    XInput,
    XButton
  },
  computed: {
    codeHint () {
      if (this.codeCooldown <= 0) {
        return '发送验证码'
      } else {
        return `重新发送 (${this.codeCooldown})`
      }
    },
    codeStyle () {
      return {
        color: this.codeCooldown <= 0 ? '#000' : '#D9D9D9'
      }
    }
  },
  data () {
    return {
      codeCooldown: 0,
      phone: '',
      password: '',
      code: '',
      sentPhone: '?',
      codeOk: false
    }
  },
  methods: {
    sendCode () {
      if (this.codeCooldown <= 0) {
        this.codeCooldown = 60
        let id = setInterval(() => {
          this.codeCooldown--
          if (this.codeCooldown <= 0) {
            clearInterval(id)
          }
        }, 1000)
        this.sentPhone = this.phone
        // send
        return services.user.sendCode(this.phone)
          .catch(() => {
            this.codeCooldown = 0
          })
      }
    },
    step1 () {
      let go = () => {
        this.$emit('info', {
          phone: this.phone,
          password: this.password
        })
        this.$router.go({
          query: {
            step: 2,
            action: 'register'
          }
        })
      }
      if (this.sentPhone !== this.phone) {
        services.utils.toast('请发送验证码')
        return
      }
      if (this.password.length < 6) {
        services.utils.toast('密码长度不足6位')
        return
      }
      if (this.codeOk) {
        go()
        return
      }
      return services.user.checkCode(this.code)
        .then((r) => {
          this.codeOk = r.ok
          if (r.ok) {
            go()
          } else {
            services.utils.toast('手机验证码错误')
          }
        })
    }
  },
  watch: {
    phone () {
      this.codeOk = false
    }
  }
}
</script>

<style lang="less" scoped>
.step1 {
  .blue {
    background-color: #508cee;
    color: #fff;
  }
  .blue:active {
    background-color: #407cde;
  }
  .inline {
    position: relative;
    display: flex;
    flex-wrap: nowrap;
    align-items: stretch;
    .div-line {
      position: absolute;
      left: 0;
      top: 0;
      width: 100%;
      height: 1px;
      border-top: 1px solid #D9D9D9;
      color: #D9D9D9;
      transform-origin: 0 0;
      transform: scaleY(0.5);
      left: 15px;
    }
    .inline-input {
      display: block;
      flex-grow: 1;
    }
    .vline {
      flex: 0 0 1px;
      display: flex;
      align-items: center;
    }
    .vline:before {
      content: " ";
      width: 1px;
      height: 30px;
      background-color: #D9D9D9;
    }
    .inline-btn {
      display: block;
      padding: 0 5px;
      border-width: 0;
      background-color: rgba(0,0,0,0);
      white-space: nowrap;
      font-size: inherit;
    }
    .clear {
      clear: both;
    }
  }
}
</style>