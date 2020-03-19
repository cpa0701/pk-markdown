# pk-markdown

![https://img.shields.io/badge/tui.editor-1.4.10-green](https://img.shields.io/badge/tui.editor-1.4.10-green)
![https://img.shields.io/badge/vue-2.6.10-brightgreen](https://img.shields.io/badge/vue-2.6.10-brightgreen)
![https://img.shields.io/badge/vue--cli-3.9.2-orange](https://img.shields.io/badge/vue--cli-3.9.2-orange)
![https://img.shields.io/badge/jquery-3.4.1-yellowgreen](https://img.shields.io/badge/jquery-3.4.1-yellowgreen)
![https://img.shields.io/badge/v--viewer-1.4.2-yellowgreen](https://img.shields.io/badge/v--viewer-1.4.2-red)
![https://img.shields.io/badge/axios-0.19.2-yellowgreen](https://img.shields.io/badge/axios-0.19.2-blueviolet)

[pk生态服务平台](https://www.ccyunchina.com/#/) markdown组件，基于tui.editor，所见即所得（wysiwyg)

项目github地址[https://github.com/cpa0701/pk-markdown.git](https://github.com/cpa0701/pk-markdown.git)

### 1.文档地址与demo
[tui.editor](https://nhn.github.io/tui.editor/latest/)

[demo](http://rc001.chenpengan.top/pk-markdown/)

### 2.使用
`npm i pk-markdown`
```vue
<template>
  <div id="app">
    <pk-markdown :upload-url="'/user-api/uploadFile/image'" ref="pkMarkdown"/>
  </div>
</template>

<script>
  import PkMarkdown from "../package/pk-markdown"

  export default {
    name: 'App',
    components: {
      PkMarkdown
    }
  }
</script>
```
通过$refs可以获得内部的editor对象，从而进一步封装大家自己的功能
### 3.自定义功能
1. 图片添加预览功能
在上传图片（指定了上传地址后）和viewer模式下调用parseImg可以对图片添加预览功能

```javascript
parseImg (){
    setTimeout(() => {
      $('.tui-editor-contents').find('img:not(.viewer-image)').each((i, v) => {
        const markedVue = new Vue({
          components: {
            Viewer
          },
          data () {
            return {
              image: v.src
            }
          },
          template: `
            <viewer :options="{toolbar: false, title: false, navbar: false}" :images="[image]"><img :src="image"
                                                                                                    class="viewer-image">
            </viewer>`
        }).$mount()
        $(v).replaceWith(markedVue.$el)
      })
    }, 500)
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200317150601444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NwYTA3MDE=,size_16,color_FFFFFF,t_70)
	2. 文字与图片分离展示功能
根据需求传入参数divideImg可以将此图片抽出加入至临近的.img-list元素中进行展示

```javascript
divider () {
        setTimeout(() => {
          $(`#${this.id}`).find('img:not(.viewer-image)').each((i, v) => {
            const markedVue = new Vue({
              components: {
                Viewer
              },
              data () {
                return {
                  image: v.src
                }
              },
              template: `
                <viewer :options="{toolbar: false, title: false, navbar: false}" :images="[image]"><img :src="image"
                                                                                                        class="viewer-image">
                </viewer>`
            }).$mount()
            $(v).remove()
            const $targetDom = $(`#${this.id}`).next('.img-list')
            $targetDom.children().length < 9 ? $targetDom.append(markedVue.$el) : ''
          })
        }, 500)
      }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200317150622672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NwYTA3MDE=,size_16,color_FFFFFF,t_70)
3.  添加emoji表情

```javascript
/**
       * 生成emoji按钮
       */
      initEmojiItem () {
        const emoji = `<button class="emoji"></button>`
        // 添加emoji
        this.toolbar.addItem({
          type: 'button',
          options: {
            name: 'emoji',
            $el: $(emoji),
            event: 'emojiButtonClicked',
            tooltip: 'emoji表情'
          }
        })
        const $emojiRoot = $('<ul></ul>')
        Object.values(emojiJson).map(v => {
          const emojiText = `&#x${v[0].substring(2)};`
          const $emoji = $(`<li class="emoji-icon">${emojiText}</li>`)
          $emoji.on('click', (e) => {
            this.editor.insertText(e.target.innerHTML)
          })
          $emojiRoot.append($emoji)
        })
        // 绑定点击emoji按钮事件
        const emojiButtonIndex = this.toolbar.indexOfItem('emoji')
        const $button = this.toolbar.getItem(emojiButtonIndex).$el
        this.editor.eventManager.addEventType('emojiButtonClicked')
        this.editor.eventManager.listen('emojiButtonClicked', () => {
          if (popup.isShow()) {
            popup.hide()
            return
          }

          const _$button$get = $button.get(0)
          const offsetTop = _$button$get.offsetTop
          const offsetLeft = _$button$get.offsetLeft

          popup.$el.css({
            top: offsetTop + $button.outerHeight(),
            right: _$button$get.parentElement.offsetWidth - offsetLeft - _$button$get.offsetWidth
          })

          popup.show()
        })
        // 生成emoji弹框
        const popup = this.editor.getUI().createPopup({
          header: false,
          title: false,
          content: $emojiRoot,
          className: 'emoji-list',
          $target: this.editor.getUI().getToolbar().$el,
          css: {
            'width': '300px',
            'height': '260px',
            'position': 'absolute'
          }
        })
        // 聚焦时弹框消失
        this.editor.eventManager.listen('focus', function () {
          popup.hide()
        })
      },
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200317150751780.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NwYTA3MDE=,size_16,color_FFFFFF,t_70)
4.  添加全屏功能

```javascript
/**
       * 生成全屏非全屏按钮
       */
      initFullScreenItem () {
        const $root = this.editor.getUI().$el
        this.editor.eventManager.addEventType('toggleFullScreen')
        this.editor.eventManager.listen('toggleFullScreen', function () {
          const $fullscreen = $($root).find('.fullscreen')
          if ($fullscreen.hasClass('exit-fullscreen')) {
            $fullscreen.removeClass('exit-fullscreen')
          } else {
            $fullscreen.addClass('exit-fullscreen')
          }
          toggle.toggleFullScreen($root[0])
        })
        this.toolbar.addItem({
          type: 'button',
          options: {
            name: 'fullScreen',
            tooltip: '全屏/非全屏',
            event: 'toggleFullScreen',
            $el: $('<button class="fullscreen"></button>')
          }
        })
      },
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200317150822329.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NwYTA3MDE=,size_16,color_FFFFFF,t_70)
