<template>
<div style="height: 100vw; width: 100vw; position: absolute;" v-bind:class="{windowOpen :appData.applications.solitare.openApp}">
<transition-group name="fade"  tag="div" class="windows">
<div v-if="appData.applications.solitare.openApp" v-bind:name="appData.applications.solitare.text" v-bind:open="appData.applications.solitare.openApp" v-bind:key="3" id="solitare" class="box-md app" @mousedown="high" v-bind:style="{ left: x + 'px', top: y + 'px', height: height + 'px', width: width + 'px', 'z-index': z }">
<div class="big-rap">
<div class="box-header" @mousedown.prevent="startMove" @touchstart.prevent="startMove">
    <div class="title-box">
      <div class="subtitle fancy">
      <span><h2>{{appData.applications.solitare.text}}</h2></span>
      </div>
      <div class="button-section">
        <button v-touch:tap="explode" v-on:click.prevent="explode" class="opt red" type="button" name="expand"><svgicon name="close" height="6" width="6" :original="true"></svgicon></button>
        <button class="opt green" type="button" name="fullSize" v-touch:tap="fullSize" v-on:click="fullSize" @mouseover="parentOn" @mouseleave="parentOff"><svgicon name="open" height="6" width="6" :original="true" v-on:click="fullSize"></svgicon></button>
      </div>
    </div>
  </div>
    <div class="wrappa-da-rappa">
        <div class="app-content viewport" name="solitare">
          <p id="solitare-message">Screw it, you already won!<br> Just toss the cards!</p>
          <div class="button-wrapper">
            <div class="prebee">
              <button id="throwsum" v-on:click.prevent="throwsum"  @mousedown="buttonpress" @mouseup="buttonup" v-bind:class="{ active: isActive }">Throw Some</button>
            </div>
          </div>
        </div>
      </div>
  </div>
    <div class="resize bottom-right" @touchstart.prevent='resizebottomright' @mousedown.prevent='resizebottomright'></div>
    <div class="resize top-right" @touchstart='resizetopright' @mousedown='resizetopright'></div>
    <div class="resize bottom-left" @touchstart.prevent='resizebottomleft' @mousedown.prevent='resizebottomleft'></div>
    <div class="resize top-left" @touchstart.prevent='resizetopleft' @mousedown.prevent='resizetopleft'></div>
</div>
</transition-group>
</div>
</template>

<script>
  import SolitaireWin from 'solitairewin'
  import appData from '../../appData.json'

  export default {
    components: { SolitaireWin },
    name: 'Solitare',
    data () {
      return {
        appData,
        dragState: false,
        parent: false,
        isActive: false,
        x: 0,
        y: 0,
        height: 400,
        width: 400,
        z: 200,
        count: 0
      }
    },
    updated () {
      if (this.count === 0) {
        this.startup()
        this.count = 1
      }
    },
    methods: {
      throwsum: function (event) {
        document.getElementById('solitare-message').remove()
        var win = new SolitaireWin({
          viewport: document.querySelector('.viewport'),
          path: 'static/cards/',
          images: [
            'AC.svg',
            'AD.svg',
            'AH.svg',
            'AS.svg',
            'JC.svg',
            'JD.svg',
            'JH.svg',
            'JS.svg',
            'KC.svg',
            'KD.svg',
            'KH.svg',
            'KS.svg',
            'QC.svg',
            'QD.svg',
            'QH.svg',
            'QS.svg'
          ],
          resize: true
        })
        win.load(function () {
          win.start()
        })
      },
      onResize: function (x, y, width, height) {
        var hasCanvas = document.getElementsByClassName('sw-canvas')
        if (typeof (hasCanvas) !== 'undefined' && hasCanvas !== null) {
          if (this.$el.getElementsByClassName('sw-canvas').length > 0) {
            var resizeEvent = window.document.createEvent('UIEvents')
            resizeEvent.initUIEvent('resize', true, false, window, 0)
            window.dispatchEvent(resizeEvent)
          }
        }
      },
      startup: function () {
        var initalWidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth
        var initalX = (initalWidth / 2) - 252
        var initalY = 25
        var activeApps = document.getElementsByClassName('app')
        var appArray = []
        for (var i = 0; i < activeApps.length; i++) {
          var eachActive = activeApps[i]
          appArray.push(eachActive)
        }
        initalY = 50 + (50 * appArray.length)
        initalX = initalX + (50 * appArray.length)
        this.x = initalX
        this.y = initalY
      },
      fullSize: function () {
        this.height = window.innerHeight - 55
        this.width = window.innerWidth
        this.x = 0
        this.y = 0
      },
      startMove: function (event) {
        var $this = this
        const touch = event.type === 'touchstart'
        if (!touch && event.button !== 0) return
        const events = touch ? {move: 'touchmove', stop: 'touchend'} : {move: 'mousemove', stop: 'mouseup'}
        const point = {
          x: event.clientX || event.touches[0].clientX,
          y: event.clientY || event.touches[0].clientY
        }
        const getPos = touch ? getTouchPos : getMousePos
        var moving = true
        var differenceX = $this.x - point.x
        var differenceY = $this.y - point.y
        const updateFn = () => {
          if (moving) {
            requestAnimationFrame(updateFn)
            $this.x = point.x - Math.abs(differenceX)
            $this.y = point.y - Math.abs(differenceY)
          }
        }
        const moveFn = event => getPos(event, point)
        const stopFn = event => {
          moving = false
          var initalWidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth
          var initalHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight
          if ($this.x < -$this.width / 2) {
            $this.x = -$this.width / 2
          }
          if ($this.x > (initalWidth - ($this.width / 2))) {
            $this.x = initalWidth - ($this.width / 2)
          }
          if ($this.y < -50) {
            $this.y = 0
          }
          if ($this.y > (initalHeight - 75)) {
            $this.y = initalHeight - 100
          }
          window.removeEventListener(events.move, moveFn)
          window.removeEventListener(events.stop, stopFn)
        }
        requestAnimationFrame(updateFn)
        moveFn(events)
        window.addEventListener(events.move, moveFn)
        window.addEventListener(events.stop, stopFn)
      },
      resizebottomright: function (event) {
        var $this = this
        const touch = event.type === 'touchstart'
        if (!touch && event.button !== 0) return
        const events = touch ? {move: 'touchmove', stop: 'touchend'} : {move: 'mousemove', stop: 'mouseup'}
        const point = {
          x: event.clientX || event.touches[0].clientX,
          y: event.clientY || event.touches[0].clientY
        }
        const getPos = touch ? getTouchPos : getMousePos
        var moving = true
        const updateFn = () => {
          if (moving) {
            requestAnimationFrame(updateFn)
            $this.width = Math.abs($this.x - point.x)
            var height = Math.abs($this.y - point.y) - 55
            $this.height = height
            // $this.y = point.y - Math.abs(differenceY)
          }
        }
        const moveFn = event => getPos(event, point)
        const stopFn = event => {
          moving = false
          window.removeEventListener(events.move, moveFn)
          window.removeEventListener(events.stop, stopFn)
        }
        requestAnimationFrame(updateFn)
        moveFn(events)
        window.addEventListener(events.move, moveFn)
        window.addEventListener(events.stop, stopFn)
      },
      resizebottomleft: function (event) {
        var $this = this
        const touch = event.type === 'touchstart'
        if (!touch && event.button !== 0) return
        const events = touch ? {move: 'touchmove', stop: 'touchend'} : {move: 'mousemove', stop: 'mouseup'}
        const point = {
          x: event.clientX || event.touches[0].clientX,
          y: event.clientY || event.touches[0].clientY
        }
        const getPos = touch ? getTouchPos : getMousePos
        var moving = true
        const updateFn = () => {
          if (moving) {
            requestAnimationFrame(updateFn)
            $this.width = ($this.x - point.x) + parseInt($this.$el.querySelector('#solitare').style.width, 10)
            $this.x = point.x
            var height = Math.abs($this.y - point.y) - 55
            $this.height = height
          }
        }
        const moveFn = event => getPos(event, point)
        const stopFn = event => {
          moving = false
          window.removeEventListener(events.move, moveFn)
          window.removeEventListener(events.stop, stopFn)
        }
        requestAnimationFrame(updateFn)
        moveFn(events)
        window.addEventListener(events.move, moveFn)
        window.addEventListener(events.stop, stopFn)
      },
      resizetopright: function (event) {
        var $this = this
        const touch = event.type === 'touchstart'
        if (!touch && event.button !== 0) return
        const events = touch ? {move: 'touchmove', stop: 'touchend'} : {move: 'mousemove', stop: 'mouseup'}
        const point = {
          x: event.clientX || event.touches[0].clientX,
          y: event.clientY || event.touches[0].clientY
        }
        const getPos = touch ? getTouchPos : getMousePos
        var moving = true
        const updateFn = () => {
          if (moving) {
            requestAnimationFrame(updateFn)
            $this.width = Math.abs($this.x - point.x)
            var heightchange = ($this.y - point.y) + 55
            if (isNaN(heightchange)) {
              heightchange = 0
            }
            $this.height = $this.height + heightchange
            $this.y = point.y - 55
          }
        }
        const moveFn = event => getPos(event, point)
        const stopFn = event => {
          moving = false
          window.removeEventListener(events.move, moveFn)
          window.removeEventListener(events.stop, stopFn)
        }
        requestAnimationFrame(updateFn)
        moveFn(events)
        window.addEventListener(events.move, moveFn)
        window.addEventListener(events.stop, stopFn)
      },
      resizetopleft: function (event) {
        var $this = this
        const touch = event.type === 'touchstart'
        if (!touch && event.button !== 0) return
        const events = touch ? {move: 'touchmove', stop: 'touchend'} : {move: 'mousemove', stop: 'mouseup'}
        const point = {
          x: event.clientX || event.touches[0].clientX,
          y: event.clientY || event.touches[0].clientY
        }
        const getPos = touch ? getTouchPos : getMousePos
        var moving = true
        const updateFn = () => {
          if (moving) {
            requestAnimationFrame(updateFn)
            $this.width = ($this.x - point.x) + parseInt($this.$el.querySelector('#solitare').style.width, 10)
            $this.x = point.x
            var heightchange = ($this.y - point.y) + 55
            if (isNaN(heightchange)) {
              heightchange = 0
            }
            $this.height = $this.height + heightchange
            $this.y = point.y - 55
          }
        }
        const moveFn = event => getPos(event, point)
        const stopFn = event => {
          moving = false
          window.removeEventListener(events.move, moveFn)
          window.removeEventListener(events.stop, stopFn)
        }
        requestAnimationFrame(updateFn)
        moveFn(events)
        window.addEventListener(events.move, moveFn)
        window.addEventListener(events.stop, stopFn)
      },
      buttonpress: function () {
        this.isActive = true
      },
      buttonup: function () {
        this.isActive = false
      },
      dragOn: function () {
        this.dragState = true
      },
      dragOff: function () {
        this.dragState = false
      },
      parentOn: function () {
        this.parent = true
      },
      parentOff: function () {
        this.parent = false
      },
      high: function () {
        var activeApps = document.getElementsByClassName('app')
        var initalHi = 200
        var zIndexs = []
        for (var i = 0; i < activeApps.length; i++) {
          var zindex = parseInt(activeApps[i].style.zIndex)
          zIndexs.push(zindex)
        }
        var largest = Math.max.apply(Math, zIndexs)
        if (initalHi <= largest) {
          this.z = largest + 1
        }
      },
      explode: function () {
        document.getElementsByClassName('panel')['0'].style.WebkitAnimation = 'inherit'
        document.getElementsByClassName('panel')['0'].style.backgroundImage = `url(${require('../../assets/gifs/explode.gif')})`
        document.getElementsByClassName('panel')['0'].style.backgroundSize = '100% 100%'
        this.appData.applications.solitare.openApp = false
        this.count = 0
        this.height = 400
        this.width = 400
        setTimeout(function () {
          document.getElementsByClassName('panel')['0'].style.WebkitAnimation = 'inherit'
          document.getElementsByClassName('panel')['0'].style.backgroundImage = ''
          document.getElementsByClassName('panel')['0'].style.backgroundSize = ''
        }, 1000)
      }
    }
  }
  function getMousePos (mouseEvent, point) {
    point.x = mouseEvent.clientX
    point.y = mouseEvent.clientY
  }
  function getTouchPos (touchEvent, point) {
    point.x = event.touches[0].clientX
    point.y = event.touches[0].clientY
  }
</script>
