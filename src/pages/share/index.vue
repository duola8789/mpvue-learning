<template>
  <div class="container">
    <div class="button-wrapper">
       <cover-view class="button blue-button" @tap="saveHanlder">保存图片</cover-view>
    </div>
    <canvas canvas-id="myCanvas" style="width: 100%; height: 400px;" class="test" @tap="previewHandler"></canvas>
  </div>
</template>

<script>
import card from '@/components/card'

export default {
  data() {
    return {
      sizeInfo: {
        windowWidth: 0, // 屏幕可用宽度
        backgroundHeight: 0, // 背景图高度
        canvasHeight: 588, // canvas高度
        fontSizeRatio: 1, // 字体大小修正系数
        screenRatio: 1, // 屏幕尺寸系数（以375为基准）
      },
      filePath: '',
      text: ''
    }
  },
  computed: {
    // 屏幕中心X轴坐标
    centerPositionX() {
      return this.sizeInfo.windowWidth / 2;
    },
    // 文字最大宽度
    maxTextWidth() {
      const ratio = 0.8;
      return this.sizeInfo.windowWidth * ratio;
    },
  },
  mounted() {
    const self = this;
    // 获取路由参数
    const query = self.$root.$mp.query;
    self.text = query.info;
    wx.showLoading({
      title: '图片生成中',
      mask: true
    });
    // 同步获取系统信息
    const systemInfo = wx.getSystemInfoSync();
    // 获取屏幕宽度
    self.sizeInfo.windowWidth = systemInfo.windowWidth;
    // 如果是小尺寸机型，缩小字体大小
    if (self.sizeInfo.windowWidth < 375) {
      self.sizeInfo.fontSizeRatio = 0.8;
    }
    // 绘制canvas
    self.getCanvas();
  },

  onUnload() {
    this.filePath = '';
  },

  methods: {
    // 保存当前图片
    saveHanlder() {
      const self = this;
      wx.showLoading({
        title: '图片保存中',
        mask: true
      });
      if (self.filePath) {
        // 保存图片到系统相册。
        wx.saveImageToPhotosAlbum({
          filePath: self.filePath,
          success() {
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
      } else {
        wx.canvasToTempFilePath({
          canvasId: 'myCanvas',
          fileType: 'jpg',
          success: function (res) {
            // 获得图片临时路径
            self.filePath = res.tempFilePath;
            // 保存图片到系统相册。
            wx.saveImageToPhotosAlbum({
              filePath: self.filePath,
              success() {
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
        });
      }
    },

    // 预览图片
    previewHandler() {
      const self = this;
      if(self.filePath) {
        wx.previewImage({
          urls: [self.filePath]
        });
      } else {
        wx.canvasToTempFilePath({
          canvasId: 'myCanvas',
          fileType: 'jpg',
          success: function (res) {
            // 获得图片临时路径
            self.filePath = res.tempFilePath;
            wx.previewImage({
              urls: [self.filePath]
            });
          }
        });
      }
    },

    // 生成Canvas
    getCanvas() {
      const self = this;

      // 设定相对宽度等信息
      const left = (self.sizeInfo.windowWidth - 200) / 2;

      // 创建画布
      const ctx = wx.createCanvasContext('myCanvas');

      // 绘制头像
      const avatarPath = '/static/1.jpg';
      const avatarParam = {
        x: 80,
        y: 80,
        r: 40
      };
      self.drawAvatar(ctx, avatarPath, avatarParam);
      avatarParam.x = self.sizeInfo.windowWidth - avatarParam.x;
      self.drawAvatar(ctx, avatarPath, avatarParam);

      // 绘制空心矩形
      ctx.beginPath();
      ctx.setLineWidth(1);
      ctx.setStrokeStyle("red");
      ctx.rect(left, 142, 200, 30);
      ctx.stroke();

      // 绘制文字
      const textParam = {
        x: self.centerPositionX,
        y: 160,
        lineHeight: 30,
        maxWidth: 190,
      };
      this.drawTextMultiple(ctx, self.text, textParam);

      // 绘制文字
      const textParam2 = {
        x: self.centerPositionX,
        y: 205,
        fontSize: 24,
        lineHeight: 30,
        maxWidth: 190,
      };
      this.drawTextMultiple(ctx, '点击图片进行预览', textParam2);

      // 绘制网络图片
      const imgPath2 = 'https://wx.qlogo.cn/mmopen/vi_32/DYAIOgq83erFT3NNAGfQNKxlVt3gvbic2bTwGxsyRLBZkqxJDLtPFapfMFdSibtvp4uhT87Sia4pBXRtTibWibnkwzw/132';

      // 获取图片临时路径
      wx.getImageInfo({
        src: imgPath2,
        success(res) {
          ctx.drawImage(res.path, left, 220, 200, 150);
          ctx.draw(false, function () {});
        },
        fail(res) {
          console.log(res)
        },
        complete() {
          wx.hideLoading();
        }
      });
    },


    /**
     * 绘制头像
     * @param ctx canvas环境
     * @param path 头像路径
     * @param x 圆心坐标（默认值为屏幕中央）
     * @param y 圆心坐标
     * @param r 头像半径
     */
    drawAvatar(ctx, path, {x = this.centerPositionX, y, r = 30 }) {
      // 保存画布
      ctx.save();
      ctx.beginPath();
      // 起始弧度，在3点钟方向
      const sAngle = 0;
      // 终止弧度
      const eAngle = 2 * Math.PI;
      // 圆半径
      ctx.arc(x, y, r, sAngle, eAngle);
      // 剪切
      ctx.clip();
      // 绘制图片
      ctx.drawImage(path, x - r, y - r, r * 2, r * 2);
      // 恢复画布
      ctx.restore();
      return y + r;
    },

    /**
     * 绘制文字（可自动换行）
     * @param ctx canvas环境
     * @param text 要绘制的文字
     * @param x 文字X轴坐标（默认值为屏幕中央）
     * @param y 文字Y轴坐
     * @param maxWidth 最大宽度（默认为屏幕宽度*0.8）
     * @param maxLines 最大绘制行数（默认为100, 等于1时为单行绘制）
     * @param lineHeight 行间距（默认为20）
     * @param fontSize 文字字体大小（默认为12）
     * @param color 文字颜色（默认为#666）
     * @param textAlign 文字对齐方式（默认居中）
     * @param fontWeight 字体粗细。仅支持 normal, bold（默认normal）
     */
    drawTextMultiple(ctx, text, { x = this.centerPositionX, y, maxWidth = this.maxTextWidth, maxLines = 100, lineHeight = 20, fontSize = 12, color = '#666', textAlign = 'center', fontWeight = 'normal' }) {
      // 校验参数
      if (typeof text !== 'string' || typeof x !== 'number' || typeof y !== 'number') {
        throw new Error('绘制文字失败');
      }

      // 设置字体、颜色、对齐方式
      const fontFamily = 'Helvetica, Tahoma, Arial, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif';
      ctx.font = `${fontSize}px ${fontWeight} ${fontFamily}`;
      ctx.setFillStyle(color);
      ctx.setTextAlign(textAlign);

      // 字符串分割为数组
      const arrText = text.split('');

      // 当前字符串及宽度
      let currentText = '';
      let currentWidth;
      // 当前行数
      let currentLine = 0;

      // 单行绘制（能够节省性能，但是如果文字字数过多的情况下文字会被压缩）
      // if (maxLines === 1) {
      //   ctx.fillText(text, x, y, maxWidth);
      //   y += lineHeight;
      //   return y;
      // }

      // 对数组进行循环截取（可控制测试间隔）
      const interval = 1;
      for (let i = 0; i < arrText.length; i += interval) {
        currentText += arrText.slice(i, i + interval).join('');
        currentWidth = ctx.measureText(currentText).width;
        if (currentWidth > maxWidth) {
          ctx.fillText(currentText, x, y);
          currentText = '';
          y += lineHeight;
          currentLine += 1;
          // 如果已绘制的行数已达到限定绘制行数，后续文字放弃绘制
          if (maxLines && currentLine >= maxLines) {
            break;
          }
        }
      }

      // 剩余不足一行的文字，如果不限定最大绘制行数或者未达到最大绘制行数
      if (currentText && (!maxLines || currentLine <= maxLines)) {
        ctx.fillText(currentText, x, y);
        y += lineHeight;
        currentLine += 1;
      }
      return y;
    },
  },
}
</script>

<style>
  page {
    background: #000;
  }
  .button-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    box-sizing: border-box;
    margin-bottom: 50rpx;
  }
  .button {
    width: 100%;
    height: 100rpx;
    line-height: 100rpx;
    text-align: center;
  }
  .blue-button {
    background: cadetblue;
  }
  .test {
    box-sizing: border-box;
    background: #fff;
    border: 15rpx solid grey;
  }
</style>
