<template>
  <div class="ta-upload">
    <div
      ref="upload"
      class="ta-upload__container"
      :class="{ 'is-active': draging, 'is-preview': isSuccess }">
      <template v-if="!isSuccess">
        <div>
          <ta-icon name="android-upload"></ta-icon>
          <p>将文件拖拽到此处或<span>点击上传</span></p>
        </div>
        <input type="file" multiple>
      </template>
      <img v-else :src="img">
    </div>
    <transition name="slide-down">
      <div class="ta-upload__tip" v-if="fileName">
        <p>{{ fileName }}</p>
        <p>
          <span :class="{ 'is-failed': !isSuccess }">
            {{ isUploading 
                ? '正在上传中...' 
                : (isSuccess ? '上传成功' : '上传失败') 
            }}
          </span>
          <ta-icon 
            v-show="!isUploading" 
            :name="tipIcon" 
            :class="{ 'is-failed': !isSuccess }">
          </ta-icon>
        </p>
      </div>
    </transition>
  </div>
</template>

<script>
import { on } from '@/util/event'
import { QiniuProvider } from '@/provider/qiniu-provider'
import TaIcon from '../../common/icon'
import message from '../../common/message'

export default {
  name: 'TaUpload',

  components: {
    TaIcon
  },

  data() {
    return {
      draging: false,
      fileName: '',
      tipIcon: 'checkmark-circled', // 失败的话就是close-circled
      img: null,                    // 预览图
      isUploading: true,            // 正在上传
      isSuccess: false,              // 上传是否成功
      uploadToken: null,            // 七牛上传token
    }
  },

  mounted() {
    const upload = this.$refs['upload']
    const input = upload.querySelector('input')
    this.getQiniuToken()

    on(upload, 'dragover', e => {
      e.stopPropagation()
      e.preventDefault()
      this.draging = true
    })
    on(upload, 'dragleave', e => {
      e.stopPropagation()
      e.preventDefault()
      this.draging = false
    })
    // drop
    on(upload, 'drop', e => {
      e.stopPropagation()
      e.preventDefault()
      const file = e.dataTransfer.files[0]
      this.handleFile(file)
    })
    // 模拟点击了input
    on(upload, 'click', e => {
      input.click()
    })

    on(input, 'change', e => {
      this.handleFile(input.files[0])
    })
  },


  methods: {
    handleFile(file) {
      this.fileName = file.name
      this.draging = false

      this.preview(file)
      this.uploadFile(file)
    },

    preview(file) {
      const reader = new FileReader()
      reader.onload = () => {
        const url = reader.result
        this.img = url
      }
      reader.readAsDataURL(file)
    },

    async getQiniuToken() {
      try {
        const uploadToken = await QiniuProvider.getQiniuToken()
        this.uploadToken = uploadToken
      } catch (err) {
        message(err.message, 'error')
      }
    },

    async uploadFile(file) {
      this.isUploading = true
      try {
        const res = await QiniuProvider.uploadImage(file, this.uploadToken)
        message('图片上传成功', 'success')
        this.isSuccess = true
        this.$emit('success', res)
      } catch (err) {
        message(err.message, 'error')
        this.isSuccess = false
      } finally {
        this.isUploading = false
        this.tipIcon = 'close-circled'
      }
    },
  }
}
</script>
