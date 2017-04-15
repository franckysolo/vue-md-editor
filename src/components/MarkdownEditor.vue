<template>
  <div class="md-editor">
    <uploader v-if="showUploader" @close="showUploader = false"
    csrf="code-secret">
    </uploader>
    <ul class="nav nav-tabs" role="tablist">
      <li class="active">
        <a data-toggle="tab" href="#md-content" aria-controls="md-content">
          Editeur
        </a>
      </li>
      <li>
        <a data-toggle="tab" href="#md-preview" aria-controls="md-preview">
          Rendu
        </a>
      </li>
    </ul>
    <div class="tab-content">
      <div role="tabpanel" id="md-content" class="tab-pane fade in active">
          <textarea class="form-control"
            :id="name"
            :name="name"
            v-model="input"
            @input="update">
          </textarea>
      </div>
      <div role="tabpanel" id="md-preview" class="tab-pane fade">
        <div v-html="compiledMarkdown" class="md-preview"></div>
      </div>
    </div>
    <div class="btn-toolbar" role="group">
      <div class="btn-group dropdown" role="group">
        <button title="Titres" type="button" class="btn btn-default dropdown-toggle"
          data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
          <span class="fa fa-header"></span>
          <span class="caret"></span>
        </button>
        <ul class="dropdown-menu">
          <li v-for="(title, index) in titles">
            <a class="pt-0 pb-0" href="#" :class="title" @click.prevent="snipet(title)">
              Titre {{ (index + 1) }}
            </a>
          </li>
        </ul>
      </div>
      <div class="btn-group" role="group" aria-label="toolbar">
        <button :key="index" v-for="(button, index) in buttons"
          class="btn btn-default" :title="button.label" @click.prevent="snipet(button.click)">
          <span class="fa" :class="'fa-' + button.icon"></span>
        </button>
        <button title="Images" type="button" class="btn btn-default" @click="showImages()">
          <span class="fa fa-picture-o"></span>
        </button>
      </div>
      <div class="btn-group pull-right" role="group">
        <button type="reset" class="btn btn-default" @click="resetInput()">
          <span class="fa fa-eraser"></span>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash'
import $ from 'jquery'
import marked from 'marked'
import hg from 'highlight.js'
import Uploader from './ImageManager'
export default {
  name: 'MarkdownEditor',
  components: {
    Uploader
  },
  props: {
    name: {
      type: String,
      default: 'content'
    }
  },
  created () {
    for (var i = 1; i <= 6; i++) {
      this.titles.push('h' + i)
    }
    this.$on('show-images', () => { this.showUploader = true })
    this.$on('insert-image', image => this.parseImage(image))
    this.$on('hide-images', () => { this.showUploader = false })
    marked.setOptions({
      highlight: function (code) {
        return require('highlight.js').highlightAuto(code).value
      }
    })
  },
  computed: {
    compiledMarkdown () {
      return marked(this.input, {
        sanitize: true,
        highlight: (code) => {
          return hg.highlightAuto(code).value
        }
      })
    }
  },
  data () {
    return {
      input: '',
      output: '',
      showUploader: false,
      crlf: '\r\n',
      titles: [],
      buttons: [
        { icon: 'bold', label: 'Gras', click: 'bold' },
        { icon: 'italic', label: 'Italique', click: 'italic' },
        { icon: 'minus-square-o', label: 'Barre d\'espace', click: 'hr' },
        { icon: 'list', label: 'Liste à puces', click: 'list' },
        { icon: 'list-ol', label: 'Liste à puces', click: 'numlist' },
        { icon: 'link', label: 'Lien', click: 'link' },
        { icon: 'list-alt', label: 'Citation', click: 'cite' },
        { icon: 'code', label: 'Code', click: 'code' },
        { icon: 'table', label: 'Tableau', click: 'table' }
      ]
    }
  },
  methods: {
    // @TODO translation i18n
    update: _.debounce(function (e) {
      this.input = e.target.value
      this.$emit('md-preview', this.compiledMarkdown)
      this.optimizeStyle()
    }, 300),
    resetInput () {
      this.input = ''
    },
    showImages () {
      this.$emit('show-images')
    },
    parseImage (image) {
      let selected = this.getSelectedText()
      this.insertAtCursor(selected, this.parse('img', image, selected.text), 'img')
    },
    snipet (type) {
      let selected = this.getSelectedText()
      this.insertAtCursor(selected, this.parse(type, selected.text), type)
    },
    optimizeStyle () {
      let tables = $('#md-preview').find('table')
      $.each(tables, (i, t) => {
        if (!$(t).hasClass('table')) {
          $(t).addClass('table table-striped table-bordered')
        }
      })
    },
    getSelectedText () {
      let selectedText = ''
      let textarea = $('#' + this.name)
      let start = textarea.prop('selectionStart')
      let end = textarea.prop('selectionEnd')
      selectedText = textarea.val().substring(start, end)
      return {
        input: textarea,
        text: selectedText,
        start: start,
        end: end
      }
    },
    insertAtCursor (el, value, type) {
      let start = el.start
      let end = el.end
      let text = el.input.val()
      let before = text.substring(0, start)
      let after = text.substring(end, text.length)
      console.log(before)
      console.log(after)
      console.log(before + value + after)
      el.input.val(before + value + after)
      if (el.text === '') {
        let pattern = /(hr|list|listnum|table|cite|img)/
        if (pattern.test(type)) {
          el.input.prop('selectionEnd', end + value.length)
        } else if (type === 'link') {
          el.input.prop('selectionEnd', end + value.length - 1)
        } else if (type === 'code') {
          el.input.prop('selectionEnd', end + 3)
        } else if (type === 'bold' || type === 'italic') {
          el.input.prop('selectionEnd', start + (el.text.length / 2) + (value.length / 2))
        }
      } else {
        el.input.prop('selectionStart', start + value.length)
      }
      this.input = el.input.val()
      el.input.focus()
      return false
    },
    parse (type, value, label) {
      switch (type) {
        case 'bold': return '**' + value + '**'
        case 'italic': return '*' + value + '*'
        case 'hr': return this.crlf + '___' + this.crlf
        case 'list': return this.mdList(value)
        case 'numlist': return this.mdNumList(value)
        case 'table': return this.mdTable(value)
        case 'link': return this.mdLink(value)
        case 'img': return this.mdImage(value, label)
        case 'cite': return '> Citation : ' + this.crlf + value
        case 'code': return this.mdCode(value)
        case 'h1': return '# ' + value + this.crlf
        case 'h2': return '## ' + value + this.crlf
        case 'h3': return '### ' + value + this.crlf
        case 'h4': return '#### ' + value + this.crlf
        case 'h5': return '##### ' + value + this.crlf
        case 'h6': return '###### ' + value + this.crlf
        default: return ''
      }
    },
    createList (items, numeric = false) {
      let string = ''
      let isDefault = Number.isInteger(items)
      let max = isDefault ? items : items.length
      for (var i = 0; i < max; i++) {
        let index = i + 1
        let label = isDefault ? 'Item ' + index : items[i]
        string += numeric ? `${index}. ${label}` : `* ${label}`
        string += this.crlf
      }
      return string
    },
    mdList (value) {
      if (value === '') {
        return this.createList(3)
      }
      let array = value.split(/\r?\n/)
      if (array.length > 1) {
        return this.createList(array)
      }
    },
    mdNumList (value) {
      if (value === '') {
        return this.createList(3, true)
      }
      let array = value.split(/\r?\n/)
      if (array.length > 1) {
        return this.createList(array, true)
      }
    },
    mdTable (value) {
      let header = '| header 1 | header 2 | header 3 |'
      let tr = '|:---:|:---:|:---:|'
      let row = '| texte | texte | texte |'
      if (value === '') {
        return [header, tr, row, row].join(this.crlf)
      }
      let array = value.split(/\r?\n?\s/)
      header = ''
      for (var i = 1; i <= array.length; i++) {
        let label = array[i - 1]
        header += `| ${label} ${i} `
      }
      header += '|'
      return [header, tr, row, row].join(this.crlf)
    },
    mdLink (value) {
      if (value === '') {
        return '[Texte du lien]()'
      }
      let pattern = /http|www/
      if (pattern.test(value)) {
        return `[Texte du lien](${value})`
      }
      return `[${value}](http://www.domain.com)`
    },
    mdImage (src, alt) {
      if (alt === '') {
        return `![Texte alternatif de l'image](${src} "Mon Image")`
      }

      return `![${alt}](${src} "${alt}")`
    },
    mdCode (value) {
      let blocCode = '```'
      if (value === '') {
        return blocCode + this.crlf + blocCode
      }
      return blocCode + this.crlf + value + this.crlf + blocCode
    }
  }
}
</script>

<style lang="less">
.md {
  &-editor {
    margin-top: 2%;
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 5px;
    .tab {
      &-pane {
        margin: 5px 0;
        textarea {
          padding: 2px;
          width: 100%;
          max-width: 100%;
          min-width: 100%;
          min-height: 120px;
          border-color: #ddd;
          &:focus {
            box-shadow: none;
          }
        }
      }
    }
    .dropdown-menu > li > a {
      padding: 5px;
      margin: 0 5px;
    }
    .dropdown-menu {
      > li {
        > a {
          &.h(1-6) {
            padding: 0;
          }
        }
      }
    }
  }

  &-preview {
    min-height: 120px;
    border: 2px dashed #ddd;
    padding: 20px;
  }
}
</style>
