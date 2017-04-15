<template>
  <transition name="modal">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-dialog modal-md">
          <div class="modal-content" @click.stop>
            <div class="modal-header">
              <h3 class="modal-title h4">
                Vos images
                <a title="Envoyer une image" @click="openFileManager" class="btn btn-default pull-right">
                  <span class="fa fa-upload"></span>
                </a>
              </h3>
            </div>
            <div class="modal-body mb-10">
              <wait-loading :loading="loading" v-if="loading">
                Chargement en cours, veuillez patientez...
              </wait-loading>
              <div id="images-container" class="row" v-if="images.length > 0 && !loading">
                <div class="col-xs-6 col-sm-2" v-for="image in images">
                  <div class="thumbnail">
                    <a @click="insertImage(image)" class="btn btn-xs btn-default">
                      <img :src="image" :alt="image" class="img-responsive center-block">
                    </a>
                    <div class="caption text-right">
                      <a @click="removeImage(image)" class="btn btn-xs btn-default">
                        <span class="fa fa-remove"></span>
                      </a>
                    </div>
                  </div>
                </div>
              </div>
              <div class="alert alert-info" v-if="images.length === 0 && !loading">
                <span class="fa fa-warning"></span>
                Pas d'images dans la bibliothèque
              </div>
            </div>
            <div class="modal-footer">
            <form action="images" enctype="multipart/form-data">
              <input type="hidden" name="_token" :value="csrf" >
              <input type="hidden" name="_method" value="POST" >
              <span v-show="error" class="pull-left text-danger">{{ error }}</span>
              <input ref="image" id="file-upload" type="file" name="image" class="hide" @change="uploadFile">
              <button type="button" class="btn btn-danger" @click="$emit('close')">
               Fermer
              </button>
            </form>
          </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import $ from 'jquery'
import toastr from 'toastr'
import WaitLoading from './Loading'
export default {
  name: 'ImageManager',
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    url: {
      type: String,
      default: '/manager/image/list'
    },
    uploadUrl: {
      type: String,
      default: '/manager/image'
    },
    removeUrl: {
      type: String,
      default: '/manager/image/remove'
    },
    csrf: {
      type: String,
      required: true
    }
  },
  components: {
    WaitLoading
  },
  data () {
    return {
      images: [],
      image: '',
      loading: false,
      show: this.visible,
      error: ''
    }
  },
  mounted () {
    this.loading = true
    if (this.$http) {
      this.$http.get(this.url)
      .then(response => {
        this.images = response.data
      }).then(() => {
        this.loading = false
      }).catch(response => console.error(response))
    } else {
      this.images.push('static/logo.png')
      this.loading = false
    }
  },
  methods: {
    open () {
      this.visible = true
    },
    uploadFile (e) {
      e.preventDefault()
      if (this.$http) {
        var files = this.$refs.image.files
        var data = new FormData()
        // for single file
        data.append('image', files[0])
        this.loading = true
        this.$http.post(this.uploadUrl, data)
        .then(response => {
          this.$set(this.$data, 'images', response.data)
          this.loading = false
        }).catch((error) => {
          this.loading = false
          this.error = error.response.data.image[0]
        })
      } else {
        toastr.warning('Uploading is disabled on demo')
      }
    },
    openFileManager () {
      $('#file-upload').trigger('click')
    },
    insertImage (image) {
      this.$parent.$emit('insert-image', image)
      this.$emit('close')
    },
    removeImage (image) {
      if (this.$http) {
        this.$http.post(this.removeUrl, {data: image})
        .then(response => {
          let vm = this
          this.$nextTick(() => {
            vm.$set(vm.$data, 'images', response.data)
          })
          this.loading = false
        }).catch((error) => {
          this.loading = false
          this.error = error.response.data.image[0]
        })
      } else {
        toastr.warning('Image removing is disabled on demo')
      }
    },
    close () {
      this.visible = false
    }
  }
}
</script>

<style lang="less">
.modal-mask {
 position: fixed;
 z-index: 9998;
 top: 0;
 left: 0;
 width: 100%;
 height: 100%;
 background-color: rgba(0, 0, 0, .5);
 display: table;
 transition: opacity .3s ease;
}

.modal-wrapper {
 display: table-cell;
 vertical-align: middle;
}

.modal-container {
 max-width: 300px;
 margin: 0px auto;
 padding: 20px 30px;
 background-color: #fff;
 border-radius: 2px;
 box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
 transition: all .3s ease;
}

.modal-enter, .modal-leave {
 opacity: 0;
}

.modal-enter .modal-container,
.modal-leave .modal-container {
 -webkit-transform: scale(1.1);
 transform: scale(1.1);
}
</style>
