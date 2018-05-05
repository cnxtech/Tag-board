<template>
  <div id="app">
    <div class="img-box">
      <div class="img-body-wrapper">
        <span class="img-body" ref="imgBody">
          <img :src="imageUrl" ref="img" class="img" width="100%" height="100%">
          <div class="tag-content" ref="tagContent" @mouseup="finishTag($event)" @mousemove="moveTag($event)"
               @mousedown="clickContent">
            <div class="tag-wrapper">
                <div class="tag" v-for="(item, index) in tags" :key="index"
                     :style="{
                 left: (item.ex-item.x > 0 ? item.x : item.ex)*100 + '%',
                 top: (item.ey-item.y > 0 ? item.y : item.ey)*100 + '%',
                 width: Math.abs(item.ex-item.x) * 100 + '%',
                 height: Math.abs(item.ey-item.y) * 100 + '%',
                 background: showShade ? 'rgba(0, 0, 0, 0)' : 'rgba(255, 0, 0, 0.3)'}"
                     :class="{'active': index === activeIndex}"
                     @mousedown.stop="clickTag($event, index, item)">
                <span class="tag-text" v-show="!(showShade && activeIndex === index)">{{ item.text }}</span>
                <div class="spot-wrapper">
                  <i class="spot" id="spot" v-for="i in 8" :key="i" @mousedown.stop="clickSpot($event, i, item)"></i>
                </div>
                <div class="em-wrapper" v-show="showShade && activeIndex === index">
                  <div class="em" v-for="i in 4" :key="i"></div>
                </div>
              </div>
            </div>
          </div>
        </span>
      </div>
      <ul class="drop-menu" ref="dropMenu" v-show="showMenu">
        <div>
          <div class="btn delete-btn" @click="deleteTag">删除</div>
          <div class="btn edit-btn" @click="editTag">编辑</div>
        </div>
      </ul>
    </div>
    <transition enter-active-class="bounceIn" leave-active-class="bounceOut" class="animated">
      <div class="input-box-wrapper" v-show="showEdit">
        <div class="input-box">
          <div class="header">
            <span class="title">编辑内容</span>
            <button type="button" class="close close-btn" aria-label="Close" @click="showEdit = false">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="content">
            <input type="text" class="input" v-model="text">
            <div class="my-btn-group">
              <button type="button" class="btn confirm-btn btn-primary btn-sm" @click="confirm">确定</button>
              <button type="button" class="btn cancel-btn btn-outline-dark btn-sm" @click="showEdit = false">取消</button>
            </div>
          </div>
        </div>
      </div>
    </transition>

  </div>
</template>

<script>
  let isMouseDown = false
  let moveTag = false
  let moveSpot = false
  let newTag = false
  let obj = {} // 鼠标click的位置
  let oldTag = {}
  let tagItem = {}
  let spotIndex = 0
  export default {
    name: "App",
    data () {
      return {
        text: '',
        activeIndex: -1,
        showMenu: false,
        showEdit: false,
        showShade: false,
        imageUrl: "https://i1.mifile.cn/f/i/18/mitv4A/40/build.jpg",
        tags: [{
          x: 0.3,
          y: 0.3,
          ex: 0.5,
          ey: 0.5,
          text: 'opo'
        }],
        shade: false
      }
    },
    computed: {},
    mounted () {
      let t = this.$refs.tagContent
      t.oncontextmenu = function () {
        return false
      }
      this.resizeImg()
      // 当窗口大小调整时，画板大小也同步调整
      window.addEventListener('resize', this.resizeImg)
      // 离开页面前二次确认
      window.onbeforeunload = function (e) {
        e = e || window.event
        if (e) {
          e.returnValue = 'Sure?'
        }
        return 'Sure?'
      }
    },
    methods: {
      resizeImg: function () {
        let naturalWidth = this.$refs.img.naturalWidth
        let naturalHeight = this.$refs.img.naturalHeight
        let t = this.$refs.tagContent
        this.$refs.imgBody.style.height = naturalHeight * t.offsetWidth / naturalWidth + 'px'
      },
      clickTag: function (e, i, item) {
        // 右键点击，显示菜单
        if (e.which === 3) {
          this.$refs.dropMenu.style.left = e.clientX + 'px'
          this.$refs.dropMenu.style.top = e.clientY + 'px'
          this.showMenu = true
          this.activeIndex = i
          return
        }
        isMouseDown = true
        this.showMenu = false
        this.showShade = true
        moveTag = true
        this.activeIndex = i
        let t = this.$refs.tagContent
        obj = {
          x: this._offset(t).left,
          y: this._offset(t).top,
          cx: e.clientX,
          cy: e.clientY,
          w: t.offsetWidth,
          h: t.offsetHeight
        }
        Object.assign(oldTag, item)
        tagItem = item
      },
      moveTag: function (e) {
        if (isMouseDown && moveTag) {
          tagItem.x = oldTag.x + (e.clientX - obj.cx) / obj.w
          tagItem.y = oldTag.y + (e.clientY - obj.cy) / obj.h
          tagItem.ex = oldTag.ex + (e.clientX - obj.cx) / obj.w
          tagItem.ey = oldTag.ey + (e.clientY - obj.cy) / obj.h
          if (tagItem.y < 0) {
            tagItem.y = 0
          }
          if (tagItem.x < 0) {
            tagItem.x = 0
          }
          if (tagItem.ey < 0) {
            tagItem.ey = 0
          }
          if (tagItem.ex < 0) {
            tagItem.ex = 0
          }
          if (tagItem.ey > 1) {
            tagItem.ey = 1
          }
          if (tagItem.ex > 1) {
            tagItem.ex = 1
          }
          if (tagItem.y > 1) {
            tagItem.y = 1
          }
          if (tagItem.x > 1) {
            tagItem.x = 1
          }
        } else if (moveSpot && isMouseDown) {
          if (spotIndex === 1) {
            if (oldTag.x > oldTag.ex) {
              tagItem.ex = oldTag.ex + (e.clientX - obj.cx) / obj.w
            } else {
              tagItem.x = oldTag.x + (e.clientX - obj.cx) / obj.w
            }
            if (oldTag.y > oldTag.ey) {
              tagItem.ey = oldTag.ey + (e.clientX - obj.cx) / obj.w
            } else {
              tagItem.y = oldTag.y + (e.clientX - obj.cx) / obj.w
            }
          } else if (spotIndex === 2) {
            if (oldTag.y > oldTag.ey) {
              tagItem.ey = oldTag.ey + (e.clientY - obj.cy) / obj.h
            } else {
              tagItem.y = oldTag.y + (e.clientY - obj.cy) / obj.h
            }
          } else if (spotIndex === 3) {
            if (oldTag.x > oldTag.ex) {
              tagItem.x = oldTag.x + (e.clientX - obj.cx) / obj.w
            } else {
              tagItem.ex = oldTag.ex + (e.clientX - obj.cx) / obj.w
            }
            if (oldTag.y > oldTag.ey) {
              tagItem.ey = oldTag.ey + (e.clientY - obj.cy) / obj.h
            } else {
              tagItem.y = oldTag.y + (e.clientY - obj.cy) / obj.h
            }
          } else if (spotIndex === 4) {
            if (oldTag.x > oldTag.ex) {
              tagItem.ex = oldTag.ex + (e.clientX - obj.cx) / obj.w
            } else {
              tagItem.x = oldTag.x + (e.clientX - obj.cx) / obj.w
            }
          } else if (spotIndex === 5) {
            if (oldTag.x > oldTag.ex) {
              tagItem.x = oldTag.x + (e.clientX - obj.cx) / obj.w
            } else {
              tagItem.ex = oldTag.ex + (e.clientX - obj.cx) / obj.w
            }
          } else if (spotIndex === 6) {
            if (oldTag.x > oldTag.ex) {
              tagItem.ex = oldTag.ex + (e.clientX - obj.cx) / obj.w
            } else {
              tagItem.x = oldTag.x + (e.clientX - obj.cx) / obj.w
            }
            if (oldTag.y > oldTag.ey) {
              tagItem.y = oldTag.y + (e.clientY - obj.cy) / obj.h
            } else {
              tagItem.ey = oldTag.ey + (e.clientY - obj.cy) / obj.h
            }
          } else if (spotIndex === 7) {
            if (oldTag.y > oldTag.ey) {
              tagItem.y = oldTag.y + (e.clientY - obj.cy) / obj.h
            } else {
              tagItem.ey = oldTag.ey + (e.clientY - obj.cy) / obj.h
            }
          } else if (spotIndex === 8) {
            if (oldTag.x > oldTag.ex) {
              tagItem.x = oldTag.x + (e.clientX - obj.cx) / obj.w
            } else {
              tagItem.ex = oldTag.ex + (e.clientX - obj.cx) / obj.w
            }
            if (oldTag.y > oldTag.ey) {
              tagItem.y = oldTag.y + (e.clientY - obj.cy) / obj.h
            } else {
              tagItem.ey = oldTag.ey + (e.clientY - obj.cy) / obj.h
            }
          }
        } else if (isMouseDown) {
          if (!newTag) {
            let t = this.$refs.tagContent
            obj = {
              x: this._offset(t).left,
              y: this._offset(t).top,
              cx: e.clientX,
              cy: e.clientY,
              w: t.offsetWidth,
              h: t.offsetHeight
            }
            let item = {
              x: (e.clientX - obj.x) / obj.w,
              y: (e.clientY - obj.y) / obj.h,
              ex: (e.clientX - obj.x) / obj.w,
              ey: (e.clientY - obj.y) / obj.h
            }
            this.tags.push(item)
            tagItem = item
            newTag = true
          } else {
            this.showShade = true
            this.activeIndex = this.tags.length - 1
            tagItem.ex = tagItem.x + (e.clientX - obj.cx) / obj.w
            tagItem.ey = tagItem.y + (e.clientY - obj.cy) / obj.h
          }
        }
      },
      clickSpot: function (e, i, item) {
        isMouseDown = true
        moveSpot = true
        this.showShade = true
        let t = this.$refs.tagContent
        obj = {
          x: this._offset(t).left,
          y: this._offset(t).top,
          cx: e.clientX,
          cy: e.clientY,
          w: t.offsetWidth,
          h: t.offsetHeight
        }
        Object.assign(oldTag, item)
        spotIndex = i
        tagItem = item
      },
      finishTag: function (e) {
        isMouseDown = false
        moveTag = false
        moveSpot = false
        if (newTag) {
          this.showEdit = true
          newTag = false
        }
        this.showShade = false
      },
      clickContent: function () {
        isMouseDown = true
        this.activeIndex = -1
        this.showMenu = false
      },
      confirm: function () {
        tagItem.text = this.text
        this.text = ''
        this.showEdit = false
      },
      deleteTag: function () {
        this.tags.splice(this.activeIndex, 1)
        this.showMenu = false
      },
      editTag: function () {
        this.showEdit = true
        this.text = this.tags[this.activeIndex].text
        this.showMenu = false
      },
      _offset: function (target) {
        let top, left
        top = 0
        left = 0

        while (target.offsetParent) {
          top += target.offsetTop
          left += target.offsetLeft
          target = target.offsetParent
        }

        return {
          top: top,
          left: left
        }
      }
    }
  }
</script>

<style lang="stylus">
  #app
    font-family "Avenir", Helvetica, Arial, sans-serif
    -webkit-font-smoothing antialiased
    -moz-osx-font-smoothing grayscale
    color #2c3e50
    .btn
      cursor pointer
    .img-box
      position fixed
      left: 0
      top 0
      width 100%
      height: 100%
      background-color rgb(155, 155, 155)
      .img-body-wrapper
        display flex
        justify-content center
        align-items center
        height: 100%
        .img-body
          position relative
          overflow hidden
          display inline-block
          width: 100%
          height 300px
          .img
            position absolute
            width 100%
            height 100%
          .tag-content
            position: absolute
            width: 100%
            height: 100%
            z-index: 100
            cursor crosshair
            .tag-wrapper
              position relative
              width 100%
              height 100%
              .tag
                position absolute
                cursor move
                padding 1px
                border 1px dashed #959595
                .tag-text
                  position absolute
                  left: 0
                  top: 0
                  width: 100%
                  height: 100%
                  display flex
                  justify-content center
                  align-items center
                  color #fff
                  font-size 16px
                .spot-wrapper
                  .spot
                    visibility hidden
                    display inline-block
                    position absolute
                    width: 9px
                    height 9px
                    cursor default !important
                    border 1px solid #fff
                    border-radius 50%
                    background-color #959595
                    &:nth-child(1)
                      top: -6px
                      left -6px
                    &:nth-child(2)
                      top: -6px
                      left: 50%
                      transform translateX(-5px)
                    &:nth-child(3)
                      top: -6px
                      right: -6px
                    &:nth-child(4)
                      top: 50%
                      left -6px
                      transform translateY(-5px)
                    &:nth-child(5)
                      top: 50%
                      right: -6px
                      transform translateY(-5px)
                    &:nth-child(6)
                      bottom: -6px
                      left -6px
                    &:nth-child(7)
                      bottom: -6px
                      left: 50%
                      transform translateX(-5px)
                    &:nth-child(8)
                      bottom: -6px
                      right: -6px
                &.active
                  .spot
                    visibility visible
                .em-wrapper
                  .em
                    position absolute
                    background-color rgba(0, 0, 0, 0.4)
                    z-index: -1
                    width: 50000px
                    height 50000px
                    &:nth-child(1)
                      bottom 100%
                      transform translateX(-10000px)
                    &:nth-child(2)
                      height: 100%
                      left 100%
                      transform translateY(-1px)
                    &:nth-child(3)
                      top: 100%
                      transform translateX(-1px)
                    &:nth-child(4)
                      right: 100%
                      transform translateY(-1px)
      .drop-menu
        position fixed
        width: 100px
        z-index: 100000
        text-align center
        background-color #fff
        box-shadow 0 16px 24px 2px rgba(0, 0, 0, .06), 0 6px 30px 5px rgba(0, 0, 0, .12), 0 8px 10px -7px rgba(0, 0, 0, .2)
        .btn
          width: 100%
          border-radius 0
        .btn:hover
          background-color #aaa
    .input-box-wrapper
      position fixed
      left 0
      top 0
      width 100%
      height 100%
      display flex
      justify-content center
      z-index: 10000
      animation-duration: 0.5s;
      .input-box
        width 300px
        margin: auto
        .header
          width 100%
          height 40px
          padding 0 15px
          line-height 40px
          background-color #333
          color: #fff
          .title
            float left
          .close-btn
            margin-top 7px
            float right
            color #fff
        .content
          width: 100%
          padding 10px 20px 25px 20px
          background-color #fff
          .input
            margin: 20px 0
            width: 100%
            padding 2px 8px
            border 1px solid #888
            border-radius 5px
          .my-btn-group
            display flex
            justify-content space-between
            .btn
              width: 80px
</style>
