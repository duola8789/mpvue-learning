<template>
  <div class="container">
    <div class="wrapper">
       <canvas canvas-id="myCanvas" style="width: 100%; height: 500px;"></canvas>
    </div>
    <button @click="share" class="button">share</button>
  </div>
</template>

<script>
import card from '@/components/card'

export default {
  data() {
    return {
      isShow: false,
      filePath: '',
    }
  },
  computed: {
    test() {
      return this.number + 100
    }
  },

  components: {
    card
  },

  mounted() {
    const self = this;
    self.isShow = true;

  },
  methods: {
    share() {
      const self = this;
      wx.showLoading({
        title: '图片生成中',
        mask: true
      });
      self.getCanvas();
    },

    // 生成Canvas
    getCanvas() {
      const self = this;

      // 获取设备宽度等信息
      const systemInfo = wx.getSystemInfoSync();
      const deciveWidth = systemInfo.screenWidth;

      // 设定相对宽度等信息
      const left = (deciveWidth - 200) / 2;

      // 创建画布
      const ctx = wx.createCanvasContext('myCanvas');

      // 绘制头像
      // 保存画布
      ctx.save();
      ctx.beginPath();
      // 圆心中点
      const circleX = deciveWidth/2;
      const circleY = 80;
      // 圆半径
      const circleRadius = 50;
      ctx.arc(circleX, circleY, circleRadius, 0, 2 * Math.PI);
      // 剪切
      ctx.clip();
      // 绘制图片
      const imgPath = '/static/1.jpg';
      ctx.drawImage(imgPath, circleX - circleRadius, circleY - circleRadius, circleRadius * 2, circleRadius * 2);
      // 恢复画布
      ctx.restore();


      // 绘制空心矩形
      ctx.beginPath();
      ctx.setLineWidth(1);
      ctx.setStrokeStyle("red");
      ctx.rect(left, 180, 200, 30);
      ctx.stroke();

      // 绘制文字
      ctx.setFontSize(18);
      ctx.setFillStyle('#6F6F6F');
      ctx.setTextAlign('left');
      const text = '义可就不一样了，表示最大需要换行的宽度，此参数可缺省，默认会使用canvas画布';
      this.fillTextWrap(ctx, text, left, 200, 190, 30);

      // 绘制第二张图片
      const imgPath2 = 'https://mmbiz.qpic.cn/mmbiz_jpg/DCI1x4FsN5q1Fxmq7L67l3jYQABcCAp7rPOXsebdcDhjruX83OvmC0tAWqJanUG3n0eJibOtNAq1gJJbBFkYfXw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1';
      ctx.drawImage(imgPath2, left, 320, 200, 150);

      ctx.draw(false, function () {
        wx.canvasToTempFilePath({
          canvasId: 'myCanvas',
          success: function(res) {
            // 获得图片临时路径
            self.filePath = res.tempFilePath;
            wx.saveImageToPhotosAlbum({
              filePath: self.filePath,
              success(res) {
                console.log(self.filePath);
                wx.hideLoading();
                wx.showToast({
                  title: '图片保存成功',
                  icon: 'success'
                })
              },
              fail(res) {
                console.log(res.errMsg);
                wx.hideLoading();
              },
            })
          }
        })
      });
    },

    // 文字换行
    fillTextWrap(ctx, text, x, y, maxWidth, lineHeight) {
      // 设定默认最大宽度
      const systemInfo = wx.getSystemInfoSync();
      const deciveWidth = systemInfo.screenWidth;

      // 默认参数
      maxWidth = maxWidth || deciveWidth;
      lineHeight = lineHeight || 20;

      // 校验参数
      if (typeof text !== 'string' || typeof x !== 'number' || typeof y !== 'number') {
        return;
      }

      // 字符串分割为数组
      const arrText = text.split('');
      // 当前字符串及宽度
      let currentText = '';
      let currentWidth;

      for (let letter of arrText) {
        currentText += letter;
        currentWidth = ctx.measureText(currentText).width;
        if (currentWidth > maxWidth) {
          ctx.fillText(currentText, x, y);
          currentText = '';
          y += lineHeight;
        }
      }
      if (currentText) {
        ctx.fillText(currentText, x, y);
      }
    },

  },
}
</script>

<style scoped>
  .button {
    margin-top: -200rpx;
    padding: 10rpx 50rpx;
  }
  .wrapper {
    z-index: 10;
    position: absolute;
    left: 0rpx;
    top: 150rpx;
    width: 100%;
    height: 1000rpx;
    border: 1rpx solid black;
    box-sizing: border-box;
    visibility: hidden;
  }
</style>
