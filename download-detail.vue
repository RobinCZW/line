<template lang="pug">
.download-detail
  file-detail(
    v-if='curFile',
    :file='curFile',
    :back='true',
    @open='onOpen'
  )
    .delete(slot='right', @click='onDelete') 删除
</template>
<style lang="less" scoped>
.download-detail {
  .delete {
    color: #fff;
  }
}
</style>
<script>
import services from 'utils/services'
export default {
  computed: {
    curFile () {
      let byMd5 = services.download.byMd5
      if (this.md5 && byMd5[this.md5]) {
        let item = byMd5[this.md5]
        item = {
          name: item.filename,
          size: item.size,
          path: item.path,
          md5: item.md5
        }
        this.cachedFile = item
        return item
      } else {
        return this.cachedFile
      }
    },
    md5 () {
      return this.$route.params.md5
    }
  },
  methods: {
    onOpen () {
      console.log('onOpen', this.md5)
      let byMd5 = services.download.byMd5
      byMd5[this.md5].recordOpTime()
    },
    onDelete () {
      console.log('ondelete', this.md5)
      return services.download.deleteByMd5(this.md5)
        .then(() => {
          window.history.back()
        })
    }
  },
  data () {
    return {
      cachedFile: null
    }
  }
}
</script>