
<template>
<div id="screenshot" class="container" @mousemove="move" @mousedown="mouseDown" @mouseup="mouseUp">
  
  <transition name="screenshot">
    <div class="Flash" v-if="tookScreenShot"></div>
  </transition>
  
  
  <div class="overlay" :class="{ 'highlighting' : mouseIsDown }" :style="{ borderWidth: borderWidth }" ref="overlay"></div>
  
  <div class="crosshairs" :class="{ 'hidden' : isDragging }" :style="{ left: crossHairsLeft + 'px', top: crossHairsTop + 'px' }" ref="crosshairs"></div>
  
  <div class="borderedBox" :class="{ 'hidden': !isDragging }" :style="{ left: boxLeft + 'px', top: boxTop + 'px', width: boxEndWidth + 'px', height: boxEndHeight + 'px' }"></div>
  
  <span class="tooltip" :class="{ 'hidden': !isDragging }" :style="{ left: toolTipLeft + 'px', top: toolTipTop + 'px'}" ref="tooltip">{{boxEndWidth}} x {{boxEndHeight}}px</span>
</div>
</template>

<script>
import html2canvas from 'html2canvas'
/* eslint-disable no-unused-vars */
/* eslint-disable no-debugger */

var test
export default {
emits: ['setDefaultScreenShotStatus'],
  data() { return {
mouseIsDown: false,
    isDragging: false, 
    tookScreenShot: false, // After the mouse is released
    
    // Used to calculate where to start showing the dragging area
    startX: 0,
    startY: 0,
    endX: 0,
    endY: 0,
    
    borderWidth: "",
    
    // Handling the positioning of the crosshairs
    crossHairsLeft: 0,
    crossHairsTop: 0,
    
    // The box that contains the border and all required numbers.
    boxTop: 0,
    boxLeft: 0,
    boxEndWidth: 0,
    boxEndHeight: 0,
    
    // The tooltip's required positioning numbers.
    toolTipLeft: 0,
    toolTipTop: 0,
    toolTipWidth: 0,
    toolTipHeight: 0,
    
    windowHeight: 0,
    windowWidth: 0,
    tooltipMargin: 0,
    crosshairs: '',
    overlay: '',
    tooltip: '',
    imageUrl: ''
  }
  },

  methods: {

    move: function (e) {
      
      this.crossHairsTop = e.clientY;
      this.crossHairsLeft = e.clientX;
      
      var tooltipBoundingRect = this.tooltip.getBoundingClientRect();
      
      this.toolTipWidth = tooltipBoundingRect.width;
      this.toolTipHeight = tooltipBoundingRect.height;
      
      /* 
       * Change how the borderWidth is being calculated based on the x and y values.
       * Calculate the box with the 1px border's positioning and width 
       * Calculate the positioning of the tooltip */
      if (this.mouseIsDown) {
        
        var endY = this.endY = e.clientY,
            endX = this.endX = e.clientX,
            startX = this.startX,
            startY = this.startY,
            windowWidth = this.windowWidth,
            windowHeight = this.windowHeight;
        
         // Calculating the values differently depending on how the user start's dragging.
         if (endX >= startX && endY >= startY) {
           
           this.isDragging = true;
           
           this.borderWidth = startY + "px " + (windowWidth - endX) + "px " + (windowHeight - endY) + "px " + startX + "px";
          
           this.boxTop = startY;
           this.boxLeft = startX;
           this.boxEndWidth = endX - startX;
           this.boxEndHeight = endY - startY;
           
           this.toolTipLeft = endX;
           this.toolTipTop = endY;
           
           if (endX + this.toolTipWidth >= windowWidth) {
             this.toolTipLeft = windowWidth - this.toolTipWidth - (this.tooltipMargin * 2);
           }
           
           if (endY + this.toolTipHeight + (this.tooltipMargin * 2) >= windowHeight) {
             this.toolTipTop = windowHeight - this.toolTipHeight - (this.tooltipMargin * 2);
           }
           
         } else if (endX <= startX && endY >= startY) {
           
           this.isDragging = true;
           
           this.borderWidth = startY + "px " + (windowWidth - startX) + "px " + (windowHeight - endY) + "px " + endX + "px";
           
           this.boxLeft = endX;
           this.boxTop = startY;
           this.boxEndWidth = startX - endX;
           this.boxEndHeight = endY - startY;
           
           this.toolTipLeft = endX - this.toolTipWidth;
           this.toolTipTop = endY;
           
           if (endX - this.toolTipWidth <= 0) {
             this.toolTipLeft = this.tooltipMargin;
           }
           
           if (endY + this.toolTipHeight + (this.tooltipMargin * 2) >= windowHeight) {
             this.toolTipTop = windowHeight - this.toolTipHeight - (this.tooltipMargin * 2);
           }
           
         } else if (endX >= startX && endY <= startY) {
           
           this.isDragging = true;
           
           this.boxLeft = startX;
           this.boxTop = endY;
           this.boxEndWidth = endX - startX;
           this.boxEndHeight = startY - endY;
           
           this.toolTipLeft = endX;
           this.toolTipTop = endY - this.toolTipHeight;
           
           this.borderWidth = endY + "px " + (windowWidth - endX) + "px " + (windowHeight - startY) + "px " + startX + "px";
           
           if (endX + this.toolTipWidth >= windowWidth) {
             this.toolTipLeft = windowWidth - this.toolTipWidth - (this.tooltipMargin * 2);
           }
           
           if (endY - this.toolTipHeight <= 0) {
             this.toolTipTop = this.tooltipMargin;
           }
           
         } else if (endX <= startX && endY <= startY) {
           
           this.isDragging = true;
           
           this.boxLeft = endX;
           this.boxTop = endY;
           this.boxEndWidth = startX - endX;
           this.boxEndHeight = startY - endY;
           
           this.borderWidth = endY + "px " + (windowWidth - startX) + "px " + (windowHeight - startY) + "px " + endX + "px";
           
           this.toolTipLeft = endX - this.toolTipWidth;
           this.toolTipTop = endY - this.toolTipHeight;
           
           if (endX - this.toolTipWidth <= 0) {
             this.toolTipLeft = this.tooltipMargin;
           }
           
           if (endY - this.toolTipHeight <= 0) {
             this.toolTipTop = this.tooltipMargin;
           }
           
         } else {
           this.isDragging = false;
         }
        
      }
      
    },

    mouseDown: function (e) {
      this.borderWidth = this.windowWidth + "px " + this.windowHeight + "px"; 
      
      this.startX = e.clientX;
      this.startY = e.clientY;
      
      this.toolTipWidth = this.tooltip.getBoundingClientRect().width;
      
      this.mouseIsDown = true;
      this.tookScreenShot = false;
    },

    mouseUp: function () {
      this.borderWidth = 0;
      
      if (this.isDragging) {
        // Don't take the screenshot unless the mouse moved somehow.
        this.tookScreenShot = true;
      }
      
      this.isDragging = false;
      this.mouseIsDown = false;
      this.takeScreenshot()
    },

    takeScreenshot: function () {
    // eslint-disable-next-line no-debugger
    
    html2canvas(document.querySelector('body')).then(canvas => {
      let croppedCanvas = document.createElement('canvas'),
          croppedCanvasContext = croppedCanvas.getContext('2d');

      croppedCanvas.width  = this.endX - this.startX;
      croppedCanvas.height = this.endY - this.startY;
      croppedCanvasContext.drawImage(canvas, this.startX, this.startY, croppedCanvas.width , croppedCanvas.height, 0, 0, croppedCanvas.width, croppedCanvas.height);

      this.imageUrl = croppedCanvas.toDataURL();
      console.log(this.imageUrl)
      debugger
      this.$emit('setDefaultScreenShotStatus')
    });
          

  }
},
  mounted() {
    
    this.crosshairs = document.querySelector(".crosshairs");
    this.overlay = document.querySelector(".overlay");
    this.tooltip = document.querySelector(".tooltip");
    
    var self = this;
    
    this.windowWidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
    this.windowHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;
    
    this.toolTipWidth = self.tooltip.getBoundingClientRect().width;
    
    // To recalculate the width and height if the screen size changes.
    window.onresize = function () {
      self.windowWidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
      self.windowHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;
    };
    
    this.tooltipMargin = window.getComputedStyle(this.$refs['tooltip']).margin.split("px")[0]
    
  },
}

</script>

<style>
/* Foundation */
*,
*:before,
*:after {
    box-sizing: border-box;
}

html, body {
  padding: 0;
  margin: 0;
  height: 100%;
}

.overlay,
.crosshairs,
.tooltip,
.borderedBox {
  user-select: none;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.overlay.highlighting {
  background: none;
  border-color: rgba(0, 0, 0, 0.5);
  border-style: solid;
}

.crosshairs {
  height: 100%;
  position: absolute;
  width: 100%;
  z-index: 2147483645;
}

.crosshairs.hidden {
  display: none;
}


.crosshairs::before,
.crosshairs::after {
  content: "";
  height: 100%;
  width: 100%;
  position: absolute;
  border: none !important;
  border-image:  !important;
}

.crosshairs::before {
  left: -100%;
  top: -100%;
  border-right: 1px solid rgba(255, 255, 255, 0.3) !important;
  border-bottom: 1px solid rgba(255, 255, 255, 0.3) !important;
}

.crosshairs::after {
  left: 0px;
  top: 0px;
  border-top: 1px solid rgba(255, 255, 255, 0.3) !important;
  border-left: 1px solid rgba(255, 255, 255, 0.3) !important;
}

.container {
  clear: both;
  overflow: hidden;
  width: 100%;
  height: 100%;
  background-image: url("https://s3-us-west-2.amazonaws.com/s.cdpn.io/260969/kendrick-lamar.png");
  background-repeat: no-repeat;
  background-size: cover;
}

.borderedBox {
  border: 1px solid #fff;
  position: absolute;
}

.borderedBox.hidden {
  display: none;
}

.tooltip {
  
  display: inline-block;
  position: absolute;
  
  background-color: grey;
  color: #fff;
  
  border-radius: 4px;
  
  font-size: 12px;
  font-family: monospace;
  
  padding: 6px;
  margin: 6px;
  white-space: nowrap;
}

.tooltip.hidden {
  display: none;
}

.Flash {
  position: absolute;
  width: 100%;
  height: 100%;
  
  top: 0;
  left: 0;
  
  background-color: #fff;
  z-index: 2147483646;
  
  opacity: 1;
  
  animation-delay: 0.2s;
  animation-name: fade-out;
  animation-duration: 1s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
}

.screenshot-enter-active, .screenshot-leave-active {
  transition: opacity .2s;
}

.screenshot-enter, .screenshot-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

@keyframes fade-out {
  from {
    opacity: 1;
  }

  to {
    opacity: 0;
  }
}
</style>