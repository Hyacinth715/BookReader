<template>
    <div class="ebook-bookmark" ref="bookmark">
        <div class="ebook-bookmark-text-wrapper">
            <div class="ebook-bookmark-down-wrapper" ref="iconDown">
                <span class="icon-down"></span>
            </div>
            <div class="ebook-bookmark-text">{{text}}</div>
        </div>
        <div class="ebook-bookmark-icon-wrapper" :style="isFixed ? fixedStyle : {}">
            <bookmark :color="color" :width="15" :height="35"></bookmark>
        </div>
    </div>
</template>

<script>
import { ebookMixin } from '../../utils/mixin'
import { realPx } from '../../utils/utils'
import Bookmark from '../common/Bookmark.vue'
import {getBookmark, saveBookmark} from '../../utils/localStorage'
const BLUE = '#346cbc'
const WHITE = '#fff'
export default {
    mixins:[ebookMixin],
  components: { Bookmark },
  computed:{
      height(){
          return realPx(35)
      },
      threshold(){
          return realPx(55)
      },
      fixedStyle(){
          return{
              position: 'fixed',
              top:0,
              right:`${(window.innerWidth - this.$refs.bookmark.clientWidth) / 2}px `
          }
      }
  },
  watch:{
      offsetY(v){
          if(!this.bookAvailable || this.menuVisible || this.settingVisible >= 0){
              return
          }
          if( v >= this.height && v <= this.threshold){
              //2 未到达临界状态
              this.beforeThreshold(v)
          }else if(v >= this.threshold){
              //3 超越临界状态
              this.afterThreshold(v)
          }else if (v > 0 && v < this.height){
              this.beforeHeight()
              
          }else if(v === 0){
              this.restore()
          }
      },
      //判断标签 使得不会翻页后一直存在
      isBookmark(isBookmark){
          this.isFixed = isBookmark
          if(isBookmark){
              this.color = BLUE
              
          }else{
              this.color = WHITE
             
          }
      }
  },
    data(){
        return{
            text:'',
            color:WHITE,
            isFixed:false
        }
    },
    methods:{
        addBookmark(){
            this.bookmark = getBookmark(this.fileName)
                if (!this.bookmark) {
                this.bookmark = []
            }
            const currentLocation = this.currentBook.rendition.currentLocation()
            const cfibase = currentLocation.start.cfi.replace(/!.*/, '').replace('epubcfi(', '')
            const cfistart = currentLocation.start.cfi.replace(/.*!/, '').replace(/\)/, '')
            const cfiend = currentLocation.end.cfi.replace(/.*!/, '').replace(/\)/, '')
            const cfiRange = `epubcfi(${cfibase}!,${cfistart},${cfiend})`
            const cfi = currentLocation.start.cfi
             this.currentBook.getRange(cfiRange).then(range => {
                    let text = range.toString()
            
                    text = text.replace(/\s\s/g, '')
                    text = text.replace(/\r/g, '')
                    text = text.replace(/\n/g, '')
                    text = text.replace(/\t/g, '')
                    text = text.replace(/\f/g, '')
                    this.bookmark.push({
                        cfi: cfi,
                        text: text
                    })
                this.setIsBookmark(true)
                saveBookmark(this.fileName, this.bookmark)
        })

        },
        removeBookmark(){
            const currentLocation = this.currentBook.rendition.currentLocation()
            const cfi = currentLocation.start.cfi
            if (this.bookmark) {
                this.bookmark = this.bookmark.filter(item => item.cfi !== cfi)
                saveBookmark(this.fileName, this.bookmark)
            }
            this.setIsBookmark(false)
        },
        restore(){
            //4归位
            setTimeout(() => {
                this.$refs.bookmark.style.top = `${-this.height}px`
                this.$refs.iconDown.style.transform = 'rotate(0deg)'
            },200)
            if(this.isFixed){
               this.setIsBookmark(true) 
               this.addBookmark()
            }else{
                this.setIsBookmark(false)
                this.removeBookmark()
            }
            
        },
        beforeHeight(){
            //1.未超过书签的高度
            if(this.isBookmark){
                  this.text = this.$t('book.pulldownDeleteMark')
                   this.color = BLUE
                   this.isFixed = true
              }else{
                  this.text = this.$t('book.pulldownAddMark')
                   this.color = WHITE
                   this.isFixed = false
              }
            
        },
        beforeThreshold(v){
            this.$refs.bookmark.style.top = `${-v}px`
              
              this.beforeHeight()
              
              const iconDown = this.$refs.iconDown
              if(iconDown.style.transform === 'rotate(180deg)'){
                  iconDown.style.transform = 'rotate(0deg)'
              }
              
        },
        afterThreshold(v){
            //3
            this.$refs.bookmark.style.top = `${-v}px`
              if(this.isBookmark){
                  this.text = this.$t('book.releaseDeleteMark')
                    this.color = WHITE
                    this.isFixed = false
              }else{
                  this.text = this.$t('book.releaseAddMark')
                    this.color = BLUE
                    this.isFixed = true
              }
              
              const iconDown = this.$refs.iconDown
              if(iconDown.style.transform === '' || iconDown.style.transform === 'rotate(0deg)'){
                  iconDown.style.transform = 'rotate(180deg)'
              }
              
        }
    }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
    @import "../../assets/styles/global";
    .ebook-bookmark{
        position: absolute;
        top: px2rem(-35);
        left: 0;
        z-index: 200;
        width: 100%;
        height: px2rem(35);
        .ebook-bookmark-text-wrapper {
            position: absolute;
            right: px2rem(45);
            bottom: 0;
            display: flex;
            .ebook-bookmark-down-wrapper {
                font-size: px2rem(14);
                color: white;
                transition: all .2s linear;
                @include center;
            }
            .ebook-bookmark-text {
                font-size: px2rem(14);
                color: white;
            }
         }   
        .ebook-bookmark-icon-wrapper {
            position: absolute;
            right: 0;
            bottom: 0;
            margin-right: px2rem(15);
        }
  
    }
</style>
