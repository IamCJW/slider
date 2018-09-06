<template>
  <div id="slider" class="slider-container">
    <div class="slider-group" id="slider-group" :style="{transform: 'translateX('+translateX+'px)'}"
         v-vueswipe="sliderDo">
      <div class="slider-item" :style="{width:itemWidth+'px',margin:'0 '+itemMargin+'px'+' 0 '+itemBorderMargin+'px',}" :class="{'active':activeIndex === sliderList.length - 2}">
        <img :src="sliderList[sliderList.length-2]" alt="">
      </div><div class="slider-item" :style="{width:itemWidth+'px','margin-right':itemMargin+'px',}" :class="{'active':activeIndex === sliderList.length - 1}">
        <img :src="sliderList[sliderList.length-1]" alt="">
      </div><div class="slider-item" :class="{'active':activeIndex === index}" v-for="(item,index) in sliderList"
           :style="{width:itemWidth+'px','margin-right':itemMargin+'px',}">
        <img :src="item" alt="">
      </div><div class="slider-item" :class="{'active':activeIndex === 0}" :style="{width:itemWidth+'px','margin-right':itemMargin+'px',}">
        <img :src="sliderList[0]" alt="">
      </div><div class="slider-item" :class="{'active':activeIndex === 1}" :style="{width:itemWidth+'px','margin-right':itemBorderMargin+'px',}">
        <img :src="sliderList[1]" alt="">
      </div>
    </div>
    <div class="dot-group">
      <div class="dot-item" v-for="(item,index) in sliderList" :class="{'active': index === activeIndex}"></div>
    </div>
  </div>
</template>

<script>
  let vueTouch = function (el, binding, type) {//触屏函数
    let _this = this;
    this.obj = el;
    this.binding = binding;
    this.touchType = type;
    this.vueTouches = {x: 0, y: 0};//触屏坐标
    this.vueMoves = true;
    this.vueLeave = true;
    this.vueCallBack = function (e, disXY) {
      binding.value(el, e, disXY);
    };
    this.start = function (e) {//监听touchstart事件
      this.vueMoves = true;
      this.vueLeave = true;
      this.longTouch = true;
      this.vueTouches = {x: e.changedTouches[0].pageX, y: e.changedTouches[0].pageY};
      this.time = setTimeout(function () {
        if (this.vueLeave && this.vueMoves) {
          this.touchType === "vuelongtap" && this.vueCallBack(e);
          this.longTouch = false;
        }
      }.bind(this), 1000);
    };
    this.end = function (e) {//监听touchend事件
      let disX = e.changedTouches[0].pageX - this.vueTouches.x;//计算移动的位移差
      let disY = e.changedTouches[0].pageY - this.vueTouches.y;
      let disXY = {
        disX: disX,
        disY: disY
      };
      clearTimeout(this.time);
      if (Math.abs(disX) > 10 || Math.abs(disY) > 100) {//当横向位移大于10，纵向位移大于100，则判定为滑动事件
        this.touchType === "vueswipe" ? this.vueCallBack(e, disXY) : '';//若为滑动事件则返回
        if (Math.abs(disX) > Math.abs(disY)) {//判断是横向滑动还是纵向滑动
          if (disX > 10) {
            this.touchType === "vueswiperight" ? this.vueCallBack(e, disXY) : '';//右滑
          }
          if (disX < -10) {
            this.touchType === "vueswipeleft" ? this.vueCallBack(e, disXY) : '';//左滑
          }
        } else {
          if (disY > 10) {
            this.touchType === "vueswipedown" ? this.vueCallBack(e, disXY) : '';//下滑
          }
          if (disY < -10) {
            this.touchType === "vueswipeup" ? this.vueCallBack(e, disXY) : '';//上滑
          }
        }

      } else {
        if (this.longTouch && this.vueMoves) {
          this.touchType === "vuetap" ? this.vueCallBack(e) : '';
          this.vueLeave = false
        }
      }
    };
    this.move = function (e) {//监听touchmove事件
      this.vueMoves = false;
    };
    let EVENT_START, EVENT_MOVE, EVENT_END;
    if ('ontouchstart' in window) {
      EVENT_START = 'touchstart';
      EVENT_MOVE = 'touchmove';
      EVENT_END = 'touchend';
    } else {
      EVENT_START = 'mousedown';
      EVENT_MOVE = 'mousemove';
      EVENT_END = 'mouseup';
    }
    this.obj.addEventListener(EVENT_START, function (e) {
      _this.start(e);
    }, false);
    this.obj.addEventListener(EVENT_MOVE, function (e) {
      _this.end(e);
    }, false);
    this.obj.addEventListener(EVENT_END, function (e) {
      _this.move(e);
    }, false);
  };
  export default {
    name: 'HelloWorld',
    data() {
      return {
        activeIndex: 0,//激活下标
        sliderList: ['../static/1.png', '../static/2.png', '../static/3.png'],
        translateXOld: '',//历史位移
        translateX: 0,//实际位移
        windowWidth: '',//屏幕宽度
        itemWidth: '',//元素宽度
        itemMargin: '',//元素间距
        itemBorderMargin: '',//元素边界
        translateCell: '',//每次位移的偏移
        touchX: '',//触摸位移
      }
    },
    mounted() {
      this.sliderDone();
      this.init(0.6, 0.05);
    },
    methods: {
      /*
      * @width:元素宽度 {Number} 百分比
      * @margin:元素间距 {Number} 百分比
      * */
      init(width, margin) {
        this.windowWidth = document.body.clientWidth;
        this.itemWidth = width * this.windowWidth;
        this.itemMargin = margin * this.windowWidth;
        this.itemBorderMargin = (1 - width) * this.windowWidth / 2;
        this.translateCell = this.itemWidth + this.itemMargin;
        this.translateXOld = this.translateX = this.translateCell * -2;
      },
      sliderDo(a, b, c) {
        this.translateX = this.translateXOld + c.disX;
        this.touchX = c.disX;
      },
      sliderDone() {
        document.getElementById('slider-group').addEventListener('touchend', () => {
          if (Math.abs(this.touchX) > this.translateCell / 4) {
            if(this.touchX > 0) {
              this.translateXOld += this.translateCell;
              this.activeIndex -= 1;
              this.activeIndex < 0 ? this.activeIndex = this.sliderList.length - 1 : false;
            }else if (this.touchX < 0) {
              this.translateXOld -= this.translateCell;
              this.activeIndex += 1;
              this.activeIndex >= this.sliderList.length ? this.activeIndex = 0 : false;
            }
          }
          let _this = this;
          let distance = _this.translateXOld - _this.translateX;
          let distanceSlice = (_this.translateXOld - _this.translateX) / 100;
          let time = function () {
            setTimeout(() => {
              _this.translateX += distanceSlice;
              if (distance > 0 && _this.translateX > _this.translateXOld) {
                _this.translateX = _this.translateXOld;
                if(_this.translateXOld > -2*_this.translateCell){
                  _this.translateXOld = _this.translateX = -(_this.sliderList.length+1)*_this.translateCell;
                }
              } else if (distance < 0 && _this.translateX < _this.translateXOld) {
                _this.translateX = _this.translateXOld;
                if(_this.translateXOld < -(_this.sliderList.length+1)*_this.translateCell){
                  _this.translateXOld = _this.translateX = _this.translateCell * -2;
                }
              } else {
                time()
              }
            }, 4)
          };
          time();
        })
      }
    },
    directives: {
      vueswipe: {//滑动事件
        inserted: function (el, binding) {
          new vueTouch(el, binding, "vueswipe");
        }
      }
    },
  }
</script>
<style scoped lang="stylus">
  .slider-container
    position relative
    width 100%
    overflow hidden
    box-sizing border-box
    .slider-group
      white-space nowrap
      position relative
      padding 10px 0
      .slider-item
        display inline-block
        border-radius 5%
        overflow hidden
        transition transform .3s
        transform scale(1)
        &.active
          transform scale(1.08)
        &:first-child
          margin-left 15%
        &:last-child
          margin-right 15%
        img
          display block
          width 100%
    .dot-group
      position absolute
      width 100%
      bottom 18px
      text-align center
      .dot-item
        display inline-block
        width 6px
        height 6px
        border 1px solid #ddd
        border-radius 100%
        margin 0 3px
        &.active
          background #04a3ee

</style>
