<template>
  <div class="dropdown-wrapper"
    ref="wrapper"
    v-bind:class="{
      active: show, 
      fluid: fluidClass, 
      disabled: dead, 
      loading: requesting, 
      error: errorGenerator, 
      up: topPosition
    }">
    <input
      v-on:blur="blurDropdown"
      v-bind:placeholder="placeholder"
      v-model="searchValue"
      v-on:focus="renders" 
      v-on:keyup.enter="closeDropdown"
      v-on:keyup.up="arrowSelection('up')"
      v-on:keyup.down="arrowSelection('down')"
      v-bind:class="{hidden: !hiddenClass, search: searchMode}"
      type="text">
    <div 
      v-on:click.stop="clickDropdown"
      v-if="!searchMode"
      class="dropdown-default">
        <span 
          v-if="requesting && !searchMode">Tunggu..</span>
        <span 
          v-if="!requesting && !searchMode">{{selected}}</span>
      </div>
    <ul ref="dropdown-list" class="dropdown-list">
      <li
        v-bind:key="{index}"
        v-for="(item, index) in filteredList"
        v-on:click="selectDropdown(index, 'close')" 
        v-bind:class="{selected: parseInt(idSelected) === parseInt(item[fieldId])}">
        {{item[fieldName]}}
      </li>
      <li class="isnotlist" v-if="list.length < 1 || filteredList.length < 1">Tidak ada data</li>
    </ul>
  </div>
</template>

<script>
  import $ from 'webpack-zepto'

  export default {
    name: 'Dropdown-vue-semantic',
    props: [
      'name', // field name
      'fluid',
      'searchAble', // search able optional
      'toLowerCase', // to lowercase optional
      'placeholder', // placeholder optional
      'fieldId', // field name like {id: 2}

      'fieldName', // field name like {id: 2}
      'defaultSelected',

      'touch',
      'error',

      'dependOn',
      'dependOnName',

      'endPoint', // string
      'endPointChild', // string

      'options', // ex: [{name: 'Options1', id: 1}, {name: 'Options2', id: 2}]
      'disabled'
    ],
    data () {
      return {
        show: false, // flag show
        dead: false, // flag disabling

        selected: '', // selected name field
        idSelected: 0, // id selected

        topPosition: false, // positioning
        requesting: false, // flag requesting

        searchMode: false, // flag search mode
        searchValue: '', // string search by name

        list: [], // list showing ready
        childList: [] // list have child
      }
    },
    mounted () {
      // if endpoint set, do requset
      if (typeof (this.endPoint) !== 'undefined') this.fetchData()
      // set list if options defined
      this.list = (typeof (this.options) !== 'undefined') ? this.options : []

      this.getSelected() // find selected
      this.disablingClass() // disabling if needed

      // create click, keyup(keycode:27) escape on document close dropdown
      document.addEventListener('click', () => { this.closeDropdown() })
      document.addEventListener('keyup', (e) => { if (e.keyCode === 27) this.closeDropdown() })
    },
    computed: {
      /**
       * filtering, if searchAble defined
       * set to lowercase for makesure is the same
       * @return {array} list data
       */
      filteredList () {
        let list = this.prettyWordList
        let input = new RegExp(this.searchValue.toLowerCase())
        let data = list.filter(e => { return input.test(e[this.fieldName].toLowerCase()) })
        if (typeof (this.searchAble) === 'undefined') data = list
        return data
      },
      /**
       * pretty words, set to lowercase first then capitalize
       * @return {array}
       */
      prettyWordList () {
        let data = this.list
        if (typeof (this.toLowerCase) !== 'undefined') {
          data.forEach(e => {
            e[this.fieldName] = e[this.fieldName].toLowerCase()
            e[this.fieldName] = this.capsWord(e[this.fieldName])
          })
        }
        return data
      },
      /**
       * hidden class if searchable defined
       * @return {boolean}
       */
      hiddenClass () {
        return (typeof (this.searchAble) !== 'undefined')
      },
      /**
       * fluid class if fluid defined
       * @return {[type]} [description]
       */
      fluidClass () {
        return (typeof (this.fluid) !== 'undefined')
      },
      /**
       * set flag error class if dependOn is defined but 0 and error
       * @return {boolean}
       */
      errorGenerator () {
        if (typeof (this.dependOn) !== 'undefined') return this.dependOn > 0 && this.error
        return this.error
      }
    },
    watch: {
      /**
       * depend on field of parent id
       */
      dependOn () {
        this.selected = this.placeholder
        this.list = []
        this.show = false
        if (this.disabled !== 'disabled') this.dead = this.dependOn < 1
        if (typeof (this.endPoint) !== 'undefined') this.fetchData()
      },
      defaultSelected () {
        this.getSelected()
      },
      disabled () {
        this.disablingClass()
      },
      show () {
        if (!this.show) this.$emit('touch', this.name)
        else {
          let index = this.filteredList.findIndex(e => { return parseInt(e[this.fieldId]) === this.idSelected })
          let listWrapper = $(this.$refs['dropdown-list'])
          if (!this.dead) listWrapper.scrollTop(parseInt(index) * 38)
        }
      },
      options () {
        this.selected = this.placeholder
        this.dead = false
        this.requesting = false
        this.list = this.options
        this.getSelected()
      },
      idSelected () {
        let pub = {
          value: parseInt(this.idSelected),
          name: this.name
        }
        this.$emit('selected', pub)
      }
    },
    methods: {
      capsWord (string) {
        let newWord = string.replace(/\w\S*/g, str => {
          return str.charAt(0).toUpperCase() + str.substr(1).toLowerCase()
        })
        // make it elegant
        let dk = /Dki/.test(newWord)
        let diy = /Di Yog/.test(newWord)
        if (dk) return 'DKI Jakarta'
        if (diy) return 'DI Yogyakarta'
        return newWord
      },
      arrowSelection (type) {
        let index = this.filteredList.findIndex(e => { return parseInt(e[this.fieldId]) === this.idSelected })
        let listWrapper = $(this.$refs['dropdown-list'])

        if (type === 'down' && this.show && index < this.filteredList.length - 1) {
          this.selectDropdown(index + 1, 'still')
          listWrapper.scrollTop(parseInt(index + 1) * 46)
        }
        if (type === 'up' && this.show && index > 0) {
          this.selectDropdown(index - 1, 'still')
          listWrapper.scrollTop(parseInt(index - 1) * 46)
        }
        if (index < 0) {
          this.selectDropdown(0, 'still')
          listWrapper.scrollTop(1 * 46)
        }
      },
      getSelected () {
        const select = this.filteredList.find(this.findSelected)
        this.selected = (typeof (this.placeholder) !== 'undefined') ? this.placeholder : 'Pilih'
        if (this.list.length > 0 && typeof (this.defaultSelected !== 'undefined')) {
          if (this.defaultSelected === 0) this.selected = this.placeholder
          this.idSelected = (this.defaultSelected > 0) ? this.defaultSelected : 0
          if (select) this.selected = select[this.fieldName]
        }
      },
      disablingClass () {
        let disab = true
        if (
          (typeof (this.dependOn === 'undefined') && this.disabled !== 'disabled') ||
          (typeof (this.disabled) === 'undefined' && this.disabled !== 'disabled') ||
          (typeof (this.dependOn !== 'undefined') && this.dependOn > 0 && this.disabled !== 'disabled')) {
          disab = false
          if (this.dependOn < 1) disab = true
        }
        this.dead = disab
      },
      findSelected (e) {
        let index = parseInt(this.defaultSelected)
        return parseInt(e[this.fieldId]) === index
      },
      fetchDataChild (id) {
        if (typeof (this.endPointChild) !== 'undefined') {
          this.$http.get(this.endPointChild + '/' + id)
            .then((response) => {
              this.$emit('child', response.data.items)
            })
            .catch(err => {
              if (err) console.log(err)
            })
        }
      },
      fetchData () {
        this.idSelected = 0

        var self = this
        let endPoint = this.endPoint
        let dependOn = parseInt(this.dependOn)
        if (dependOn > 0) this.requesting = true
        if (dependOn > 0) endPoint = endPoint + '/' + dependOn
        if (dependOn > 0 || typeof (this.dependOn) === 'undefined') {
          this.$http.get(endPoint)
            .then((response) => {
              self.list = response.data.items
              self.getSelected()
            })
            .then(() => {
              if (dependOn > 0 && this.disabled !== 'disabled') this.dead = false
              this.requesting = false
            })
            .catch(err => {
              if (err) console.log(err)
            })
        }
      },
      selectDropdown (index, type) {
        this.selected = this.filteredList[index][this.fieldName]
        this.idSelected = this.filteredList[index][this.fieldId]
        if (typeof (endPointChild) !== 'undefined') this.fetchDataChild(this.idSelected)
        if (type === 'close') this.closeDropdown()
      },
      getPosition () {
        var refs = $(this.$refs['dropdown-list'])
        if (refs.offset().top + 200 > $(window).height() + $(window).scrollTop()) return true
        return false
      },
      clickDropdown () {
        this.topPosition = this.getPosition()
        if (typeof (this.searchAble) !== 'undefined' && !this.dead) this.searchMode = true
        if (this.show) $(this.$el).find('input').blur()
        if (!this.show) $(this.$el).find('input').focus()
      },
      renders () {
        if (!this.dead) this.show = !this.show
      },
      blurDropdown () {
        setTimeout(() => { this.closeDropdown() }, 200)
      },
      showDropdown () {
        let index = this.filteredList.findIndex(e => { return parseInt(e[this.fieldId]) === this.idSelected })
        let listWrapper = $(this.$refs['dropdown-list'])
        if (!this.dead) {
          this.show = true
          listWrapper.scrollTop(parseInt(index) * 42)
        }
      },
      closeDropdown () {
        this.show = false
        this.searchMode = false
        this.searchValue = ''

        $(this.$el).find('input').blur()
      }
    }
  }
</script>

<style lang="scss" scoped>
  $gray: lighten(#fdfdfd, 15);
  $default-height: 32px;
  $border-radius: 0.15em;

  .dropdown-wrapper {
    text-align: left;
    height: $default-height;
    line-height: $default-height;
    position: relative;
    padding: 0;

    &.fluid {
      width: 100%;
    }
    &.disabled {
      .dropdown-default {
        background: #f4f4f4;
        border-color: #efefef;
        cursor: not-allowed;
      }
    }
    &.loading {
      &.disabled {
        .dropdown-default, .default.-list {
          background-image : linear-gradient(
            to right,
            darken(#eee,20) 8%,
            darken(#e5e5e5,20) 18%,
            darken(#eee,20) 33%);
        }
      }
      .dropdown-default, .dropdown-list {
        background-image : linear-gradient(
          to right,
          #eee 8%,
          #e5e5e5 18%,
          #eee 33%);
        animation-name: spark;
        animation-fill-mode: forwards;
        animation-duration: 800ms;
        animation-iteration-count: infinite;
        animation-timing-function: linear;
        background-size: 800px 104px;
      }
    }
    &.active {
      &.error {
        &.up .dropdown-default  {
          border-bottom: 1px solid lighten(red, 20);
          border-radius: 0 0 $border-radius $border-radius;
        }
        .dropdown-default {
          background-color: #FFFFFF;
          border-radius: $border-radius $border-radius 0 0;
          border-bottom: 0;
        }
      }
      border-radius: $border-radius $border-radius 0 0;
      .dropdown-list {
        max-height: 50vh;
        transition: max-height 0.2s ease-in;
        height: auto;
        overflow-y: auto;
        box-shadow: 2px 3px 2px rgba(0,0,0,0.1);
        visibility: visible;
        z-index: 9999;
      }
      .search {
        border-bottom: 0;
        border-radius: $border-radius $border-radius 0 0;
      }
      &.up {
        .dropdown-default {
          border-bottom: 1px solid darken($gray, 10);
        }
      }
      .dropdown-default {
        border-bottom: 0;
        z-index: 99999;
      }
      .dropdown-default:after {
        transform: rotate(45deg);
        top: 16px;
      }
    }
    &.up.active {
      .dropdown-list {
        top: inherit;
        bottom: 0;
        padding: 0 0 $default-height 0;
        border-bottom: 0;
        border-radius: $border-radius $border-radius 0 0;
      }
      .dropdown-default, .search {
        border-top: 0;
        border-radius: 0 0 $border-radius $border-radius;
      }
    }
    input {
      position: absolute;
      top: 0;
      visibility: none;
      opacity: 0;
      left: 0;
      width: inherit;
    }
    .search {
      position: absolute;
      top: 0;
      opacity: 1;
      visibility: visible;
      background-color: #FFFFFF;
      border-radius: $border-radius;
      border: 1px solid darken($gray, 10);
      border-right: 0;
      padding: 0.15em 1em;
      height: 38px;
      outline: 0;
      width: inherit;
      z-index: 10000;
    }
    &.error {
      .dropdown-default {
        color: #9F3A38;
        border-color: #E0B4B4;
        background-color: #FFFFFF;
      } 
      .dropdown-list {
        border-color: #E0B4B4;
      } 
    }
    .dropdown-default {
      padding: 0.15em 1em;
      // width: 100%;
      width: auto;
      cursor: pointer;
      position: relative;
      white-space: nowrap;
      background-color: #FFFFFF;
      border-radius: $border-radius;
      border: 1px solid darken($gray, 10);
      z-index: 3;
      overflow: hidden;
      &:after {
        content: '';
        position: absolute;
        top: 13px;
        right: 10px;
        width: 8px;
        height: 8px;
        border-right: 1px solid lighten(black, 20);
        border-top: 1px solid lighten(black, 20);
        transition: 0.1s;
        transform: rotate(135deg);
      }
    }
    .dropdown-list {
      border: 1px solid darken($gray, 10);
      top: 0;
      left: 0;
      position: absolute;
      margin: 0;
      overflow: hidden;
      visibility: hidden;
      z-index: 1;
      max-height: $default-height;
      list-style: none;
      width: inherit;
      padding: $default-height 0 0 0;
      background: #FFFFFF;
      border-radius: $border-radius;
      transition: max-height 0s ease-out;
      li.isnotlist {
        cursor: normal;
        &:hover {
          background: $gray;
        }
      }
      li.selected, .item.selected {
        background: darken($gray, 10)
      }
      li,.item {
        padding: 0.5em 1em;
        white-space: nowrap;
        width: auto;
        cursor: pointer;
        &:hover {
          background: darken($gray, 10)
        }
      }
    }
  }
  @keyframes spark{
    0% {
      background-position: -468px 0;
    }
    100% {
      background-position: 468px 0;
    }
  }
</style>

