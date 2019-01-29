<template>
    <div class="ebook">
      <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
      <div class="read-wrapper">
        <div id="read"></div>
        <div class="mask">
          <div class="left" @click="prevPage"></div>
          <div class="center" @click="toggleTitleAndMenu"></div>
          <div class="right" @click="nextPage"></div>
        </div>
      </div>
      <menu-bar ref="menuBar"
                :fontSizeList="fontSizeList"
                :defaultFontSize="defaultFontSize"
                :ifTitleAndMenuShow="ifTitleAndMenuShow"
                @setFontSize="setFontSize"
                :themeList="themeList"
                :defaultTheme="defaultTheme"
                @setTheme="setTheme"
                :bookAvailable="bookAvailable"
                @onProgressChange="onProgressChange"
                :navigation="navigation"
                @jumpTo="jumpTo"
      ></menu-bar>
    </div>
</template>

<script>
    import Epub from 'epubjs'
    import TitleBar from '@/components/TitleBar'
    import MenuBar from '@/components/MenuBar'

    const DOWNLOAD_URL = '/static/2018_Book_AgileProcessesInSoftwareEngine.epub'
    global.epub = Epub

    export default {
        name: "Ebook",
        components: {
          TitleBar,
          MenuBar
        },
        data(){
            return {
              ifTitleAndMenuShow: false,
              fontSizeList: [{
                fontSize: 12
              },{
                fontSize: 14
              },{
                fontSize: 16
              },{
                fontSize: 18
              },{
                fontSize: 20
              },{
                fontSize: 22
              },{
                fontSize: 24
              }],
              defaultFontSize: 16,
              themeList: [{
                name: 'default',
                style: {
                  body: {
                    'color': '#000',
                    'background': '#fff'
                  }
                }
              },{
                name: 'eye',
                style: {
                  body: {
                    'color': '#000',
                    'background': '#ceeaba'
                  }
                }
              },{
                name: 'night',
                style: {
                  body: {
                    'color': '#fff',
                    'background': '#000'
                  }
                }
              },{
                name: 'gold',
                style: {
                  body: {
                    'color': '#000',
                    'background': 'rgb(241,236,226)'
                  }
                }
              }],
              defaultTheme: 0,
              // 图书是否处于可用状态
              bookAvailable: false,
              navigation: {}
            }
        },
        methods: {
           showEpub(){
             // 生成book
             this.book = new Epub(DOWNLOAD_URL)
             // 生成Rendition,通过Book.renderTo
             this.Rendition = this.book.renderTo('read', {
               width: window.innerWidth,
               height: window.innerHeight
             })
             // 通过Rendition.display渲染电子书
             this.Rendition.display()
             // 获取到themes对象
             this.themes = this.Rendition.themes
             // 设置默认字体
             this.setFontSize(this.defaultFontSize)
             // this.themes.register(name,styles)
             // this.themes.select(name)
             this.registerTheme()
             this.setTheme(this.defaultTheme)
             // 获取locations对象
             // 通过epubjs的钩子函数来实现
             this.book.ready.then(() => {
               this.navigation = this.book.navigation
               return this.book.locations.generate()
             }).then(result => {
               this.locations = this.book.locations
               this.bookAvailable = true
             })
           },
          prevPage(){
             // 翻页Rendition.prev
            if(this.Rendition) {
              this.Rendition.prev()
            }
          },
          nextPage(){
            // 翻页Rendition.next
            if(this.Rendition) {
              this.Rendition.next()
            }
          },
          toggleTitleAndMenu () {
             this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow
             if(!this.ifTitleAndMenuShow){
               this.$refs.menuBar.hideSetting()
             }
          },
          setFontSize(fontSize){
            this.defaultFontSize = fontSize
            if(this.themes){
              this.themes.fontSize(fontSize + 'px')
            }
          },
          registerTheme(){
             this.themeList.forEach(theme => {
               this.themes.register(theme.name, theme.style)
             })
          },
          setTheme(index){
             this.themes.select(this.themeList[index].name)
             this.defaultTheme = index
          },
          onProgressChange(progress){
             // progress 进度条的数值0-100
            const percentage = progress/100
            const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0
            this.Rendition.display(location)
          },
          // 根据链接跳转到指定位置
          jumpTo(href){
            this.Rendition.display(href)
            this.hideTitleAndMenu()
          },
          hideTitleAndMenu(){
             // 隐藏标题栏和菜单栏
            this.ifTitleAndMenuShow = false
            // 隐藏菜单栏弹出的设置栏
            this.$refs.menuBar.hideSetting()
            // 隐藏目录
            this.$refs.menuBar.hideContent()
          }
        },
        mounted(){
            this.showEpub()
        }
    }
</script>

<style lang="scss" scoped>
@import 'assets/styles/global';
.ebook{
  position: relative;
  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 100;
      width: 100%;
      height: 100%;
      display: flex;
      .left {
        flex: 0 0 px2rem(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
      }
    }
  }
}
</style>
