<template>
  <div v-if="toggle_view" class="croppie-page">
    <div class="file-uploader" v-if="!tmp_file" @click="reset_img">
      <img class="icon-url" :src="icon_url" v-if="icon_url" />
      <template v-else>
        <img src="@/assets/img/svg/img.svg" />
        <p>{{$t('information.icon_desc')}}</p>
      </template>
    </div>
    <div v-else class="croppie" @click="reset_img">
      <vue-croppie
        ref="croppieRef"
        :enableZoom="true"
        :mouseWheelZoom="false"
        :showZoomer="false"
        :enableResize="false"
        :enableOrientation="true"
        :boundary="{ width: size, height: size}"
        :viewport="{ width: size, height: size, 'type':'circle' }"
      ></vue-croppie>
    </div>
    <input type="file" accept="image/*" @change="croppie" ref="upload_dom" @click="clickInput" />
  </div>
</template>

<script>
  export default {
    props: {
      icon_url: {
        type: String,
        default: ""
      },
      toggle_app: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        size: null,
        tmp_file: null,
        toggle_view: true
      }
    },
    watch: {
      toggle_app() {
        this.toggle_view = false
        this.$nextTick(() => (this.toggle_view = true))
        this.tmp_file = null
      }
    },
    methods: {
      clickInput(e) {
        console.log(e)
      },
      croppie(e) {
        const files = e.target.files || e.dataTransfer.files
        if (!files.length) return
        this.tmp_file = e
        const reader = new FileReader()
        reader.onload = e => {
          this.$refs.croppieRef &&
          this.$refs.croppieRef.bind({
            url: e.target.result
          })
        }
        reader.readAsDataURL(files[0])
      },
      crop() {
        return new Promise(resolve => {
          if (!this.tmp_file) return resolve(false)
          let options = {
            type: "base64",
            size: { width: 512, height: 512 },
            format: "png"
          }
          this.$refs.croppieRef.result(options, output => resolve(output))
        })
      },
      reset_img() {
        this.$refs.upload_dom.click()
      }
    }
  }
</script>

<style lang="scss" scoped>
  /deep/ .cr-viewport {
    border: 0 !important;
    box-shadow: none !important;
  }

  .croppie-page {
    width: 15rem;
    height: 15rem;
    position: relative;

    &>input {
      width: 0;
      height: 0;
    }
  }

  .croppie {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    overflow: hidden;
  }

  .file-uploader {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    background-color: #fff;
    border-radius: 50%;

    img {
      border-radius: 0;
    }

    p {
      text-align: center;
      width: 8.75rem;
      font-size: 0.88rem;
      color: #a9b0bf;
      margin-top: 1rem;
    }

    &:active {
      background: rgba(250, 250, 250, 1);
    }

    .icon-url {
      width: 100%;
      height: 100%;
      border-radius: 50%;
    }
  }

  @media screen and (max-width: 70rem) {
    .file-uploader p {
      margin-top: 0;
      position: absolute;
      left: calc(100% + 1rem);
    }
  }
</style>
