<template lang="pug">
.step2
  image-clipper(v-ref:clipper, :visible.sync='clipper', :src='avatarData', @clip='onClip')
  school-picker(:visible.sync='school.show', :academy='true', @school='school.school=$arguments[0]', @academy='school.academy=$arguments[0]', @year='school.enterYear=$arguments[0]')
  //- div(v-transfer-dom)
  //-   mt-popup(style='width: 100%;', :visible.sync='school.yearShow', position='bottom')
  //-     mt-picker(:slots='school.yearSlot', @change='onYearChange', :visible-item-count='5', v-ref:picker)
  group.first-group
    .upload
      .preview
        img(v-show='previewData.length > 0', :src='previewData')
      input-file.float(:style='inputFileStyle', :src='uploadIcon', @data='clipAvatar')
    x-input(:value.sync='nickname', placeholder='填写昵称', v-ref:phone)
    cell(:title='schoolInfo', is-link, @click='school.show=true')
    // cell(:title='enterYearTitle', is-link, @click='openYear')
  group
    radio(:options="['男', '女']", :value.sync='gender')
  group
    x-button.blue(@click='step2') 下一步
</template>
<style lang="less">
.first-group {
  .vux-no-group-title::before {
    display: none;
  }
}
</style>
<style lang="less" scoped>
.step2 {
  position: relative;
  .blue {
    background-color: #508cee;
    color: #fff;
  }
  .blue:active {
    background-color: #407cde;
  }
  .upload {
    display: block;
    margin: 0 auto;
    width: 100px;
    height: 100px;
    margin-top: 10px;
    margin-bottom: 50px;
    position: relative;
    .preview {
      height: 100%;
      width: 100%;
      position: absolute;
      left: 0;
      top: 0;
      background-color: #fff;
      > img {
        width: 100%;
        height: 100%;
        border-radius: 100%;
      }
    }
    .float {
      width: 100%;
      height: 100%;
    }
  }
}
</style>

<script>
import Group from 'vux-components/group'
import XInput from 'vux-components/x-input'
import XButton from 'vux-components/x-button'
import Cell from 'vux-components/cell'
import Radio from 'vux-components/radio'
import { resizeToScreenMax } from 'utils/canvas'
import services from 'utils/services'

export default {
  components: {
    Group,
    XInput,
    XButton,
    Cell,
    Radio
  },
  data () {
    return {
      uploadIcon: require('assets/icon/register/upload.png'),
      clipper: false,
      avatarData: '',
      previewData: '',
      nickname: '',
      gender: '',
      school: {
        show: false,
        // yearShow: false,
        // yearSlot: [{
        //   values: years
        // }],
        school: {},
        academy: {},
        enterYear: 0
      }
    }
  },
  methods: {
    openYear () {
      let enterYear = this.school.enterYear
      if (enterYear !== 0) {
        this.$refs.picker.setValues([enterYear])
      }

      let query = this.$route.query
      query.pickyear = 1
      this.$router.go({
        query: query
      })
      this.school.yearShow = true
    },
    onYearChange (picker, values) {
      this.school.enterYear = values[0]
    },
    clipAvatar (src) {
      console.log('clipBegin')
      resizeToScreenMax(src, (r) => {
        console.log('clipEnd')
        this.avatarData = r
        this.clipper = true
      })
    },
    onClip (src) {
      this.previewData = src
    },
    step2 () {
      let go = () => {
        this.$emit('info', {
          gender: this.gender,
          nick: this.nickname,
          schoolId: this.school.school.id,
          academyId: this.school.academy.id,
          enterYear: this.school.enterYear
        })
        const t76 = true // #76 直接注册
        if (t76) {
          this.$emit('register')
        } else {
          // 跳过这步
          this.$router.go({
            query: {
              step: 3,
              action: 'register'
            }
          })
        }
      }
      if (this.gender === '') {
        services.utils.toast('请选择性别')
        return
      }
      if (this.nickname.length < 2) {
        services.utils.toast('昵称过短, 请再长一点哦')
        return
      }
      if (this.schoolInfo === '选择学校') {
        services.utils.toast('请选择学校和专业')
        return
      }
      if (this.previewData === '') {
        services.utils.toast('请添加头像')
        return
      }
      if (this.school.enterYear === 0) {
        services.utils.toast('请选择入学年份')
        return
      }
      // 确定昵称没被占用
      services.user.infoByName(this.nickname)
        .then(r => {
          if (r.id) {
            services.utils.toast('昵称已被占用')
          } else {
            go()
          }
        })
    }
  },
  computed: {
    inputFileStyle () {
      return {
        opacity: this.previewData.length > 0 ? '0' : '1'
      }
    },
    schoolInfo () {
      let info = this.school
      let text = ''
      if (info.school.name && info.academy.name) {
        text = `${info.school.name}-${info.academy.name}`
      } else {
        text = '选择学校'
      }
      return text
    },
    enterYearTitle () {
      if (this.school.enterYear === 0) {
        return '选择入学年份'
      } else {
        return this.school.enterYear
      }
    }
  },
  watch: {
    'school.yearShow': function (val) {
      if (!val) {
        if (this.$route.query.pickyear) {
          window.history.back()
        }
      }
    },
    '$route.query.pickyear': function (val) {
      if (!val) {
        this.school.yearShow = false
      }
    }
  }
}
</script>