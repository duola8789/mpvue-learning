<template>
  <div class="container">
    <canvas canvas-id="shareCanvas" class="share-canvas" :style="{width: '100%', height: sizeInfo.canvasHeight + 'px', zIndex: 2}"></canvas>
    <cover-view class="save-button" @click="saveImage" :disabled="isDisabled">保存图片</cover-view>
  </div>
</template>

<script>
  // import API from '@/network/api.url';

  const PAGE = 'share';

  const CODE = {
    SUCCESS: '10000',
    PARTICIPATED: '40007',
  };

  //TODO: 1 图片生成时间长，因为文字字数不同导致的函数不固定
  //TODO: 2 是否有新版视觉稿
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
        filePath: '', // 生成图片的路径
        // TODO： 假数据及字段名
        userInfo: {
          nickName: '多拉斯基周',
          url: 'https://wx.qlogo.cn/mmopen/vi_32/DYAIOgq83erFT3NNAGfQNKxlVt3gvbic2bTwGxsyRLBZkqxJDLtPFapfMFdSibtvp4uhT87Sia4pBXRtTibWibnkwzw/132',
          tempPath: '',
          width: '',
          height: '',
        },
        promoterInfo: {
          nickName: '2222222',
          url: 'https://wx.qlogo.cn/mmopen/vi_32/DYAIOgq83erFT3NNAGfQNKxlVt3gvbic2bTwGxsyRLBZkqxJDLtPFapfMFdSibtvp4uhT87Sia4pBXRtTibWibnkwzw/132',
          tempPath: '',
          width: '',
          height: '',
        },
        // TODO： 假数据及字段名
        contractInfo: {
          matchDescription: '2018-06-05 足球友谊赛 "意大利VS荷兰" 的比赛中，我',
          target: '10个金币',
          promoterExpect: '压意大利赢，你敢应战吗？',
        },
        QRCode: {
          url: '',
          tempPath: '',
          width: '',
          height: '',
        },
        isDisabled: true
      }
    },
    computed: {
      // 屏幕中心X轴坐标
      centerPositionX() {
        return this.sizeInfo.windowWidth / 2
      },
      // 文字最大宽度
      maxTextWidth() {
        const ratio = 0.8;
        return this.sizeInfo.windowWidth * ratio
      },
      title() {
        return `${this.userInfo.nickName}的竞猜`
      },
      promiseArray() {
        return [this.userInfo, this.promoterInfo, this.QRCode]
      }
    },

    mounted() {
      const self = this;

      // TODO: 所有UI提示是否替换
      wx.showLoading({
        title: '图片加载中',
        mask: true
      });

      self.init();

      // 获取二维码
      self.getQRCode().then(self.getTempPath)
        .then((resArr) => {
          // 取出临时路径
          self.promiseArray.map((item, index) => {
            item.tempPath = resArr[index].path;
            item.width = resArr[index].width;
            item.height = resArr[index].height;
          });
          // 开始画图
          self.getCanvas()
        })
        .catch(e => {
          // TODO: 失败提示
          // self.$store.commit('failCallBack', {
          //   content: e,
          //   page: PAGE,
          //   request: 'getImageTempPath',
          // });
          wx.hideLoading();
          self.isDisabled = true;
          console.error(e)
        });


    },
    methods: {
      init() {
        const self = this;

        // 同步获取系统信息
        const systemInfo = wx.getSystemInfoSync();
        // 获取屏幕宽度
        self.sizeInfo.windowWidth = systemInfo.windowWidth;
        // 如果是小尺寸机型，缩小字体大小
        if (self.sizeInfo.windowWidth < 375) {
          self.sizeInfo.fontSizeRatio = 0.8;
        }
        // 以375为基准
        self.sizeInfo.screenRatio = Number((self.sizeInfo.windowWidth / 375).toFixed(2));
        // 修正canvas尺寸
        self.sizeInfo.canvasHeight = systemInfo.windowHeight + 10;
      },

      // 获取网络图片临时路径
      getTempPath() {
        const self = this;
        return Promise.all(self.promiseArray.map(obj => {
          return new Promise(((resolve, reject) => {
            wx.getImageInfo({
              src: obj.url,
              success(res) {
                resolve({
                  path: res.path,
                  width: res.width,
                  height: res.height
                });
              },
              fail(e) {
                reject(new Error(`获取头像信息失败：${e}`));
              }
            })
          }))
        }));
      },

      // 获取二维码
      getQRCode() {
        const self = this;
        // return new Promise((resolve, reject) => {
        //   wx.request({
        //     url: API.query,
        //     data: {
        //       // contractId: self.contractInfo.contractId,
        //     },
        //     success(ret) {
        //       const wxRet = ret.data;
        //       if (wxRet.retcode === CODE.SUCCESS) {
        //         resolve(true);
        //       } else {
        //         reject(new Error(`获取二维码失败：${wxRet.retdesc}`));
        //       }
        //     },
        //     fail(e) {
        //       reject(new Error(`获取二维码失败：${e}`));
        //     }
        //   })
        // });
        return new Promise(((resolve, reject) => {
          self.QRCode.url = 'https://wx.qlogo.cn/mmopen/vi_32/DYAIOgq83erFT3NNAGfQNKxlVt3gvbic2bTwGxsyRLBZkqxJDLtPFapfMFdSibtvp4uhT87Sia4pBXRtTibWibnkwzw/132',
            resolve()
        }));
      },

      // 绘制头像
      drawCanvasAvatar(ctx, avatarPath) {
        const self = this;
        // 头像在背景图中的位置
        const backgroundRatio = 0.54;
        // 保存画布
        ctx.save();
        ctx.beginPath();
        // 圆心中点坐标
        const x = self.centerPositionX;
        const y = Math.round(self.sizeInfo.backgroundHeight * backgroundRatio);
        // 圆半径
        const r = 30 * self.sizeInfo.screenRatio;
        // 起始弧度，在3点钟方向
        const sAngle = 0;
        // 终止弧度
        const eAngle = 2 * Math.PI;
        // 画圆
        ctx.arc(x, y, r, sAngle, eAngle);
        // 剪切
        ctx.clip();
        // 绘制图片
        // 圆形坐标是圆心的坐标，图片的坐标是图片左上角的坐标，两个坐标系之间差了一个半径长度
        ctx.drawImage(avatarPath, x - r, y - r, r * 2, r * 2);
        // 恢复画布
        ctx.restore();
        // 返回底部尺寸
        return y + r
      },

      // 绘制背景
      drawBackground(ctx) {
        const self = this;
        const path = '/static/share-bg-1.jpg';
        // 原始图像的高宽比
        const ratio = 0.88;
        // 图像的左上角在目标canvas上X轴/Y轴的位置
        const dx = 0;
        const dy = 0;
        // 绘制图像的宽度，允许对绘制的图像进行缩放
        const dWidth = self.sizeInfo.windowWidth;
        const dHeight = Math.round(dWidth * ratio);
        // 背景图高度
        self.sizeInfo.backgroundHeight = dHeight;
        // 绘制图像到画布
        ctx.drawImage(path, dx, dy, dWidth, dHeight);
      },

      // 生成Canvas
      getCanvas() {
        const self = this;
        // 创建画布
        const ctx = wx.createCanvasContext('shareCanvas');

        // 绘制背景图
        self.drawBackground(ctx);

        // 绘制头像
        const avatarBottomPosition = self.drawCanvasAvatar(ctx, self.userInfo.tempPath);

        // 绘制昵称标题
        const titleParam = {
          y: Math.round(avatarBottomPosition + 18 * self.sizeInfo.screenRatio),
          color: '#f99191',
          fontSize: 10 * self.sizeInfo.fontSizeRatio,
          lineHeight: 14,
          maxLines: 2,
          maxWidth: 92
        };
        self.drawTextMultiple(ctx, self.title, titleParam);

        // 绘制比赛信息
        const matchDescription = self.contractInfo.matchDescription;
        const matchDescriptionParam = {
          y: Math.round(self.sizeInfo.backgroundHeight + 35 * self.sizeInfo.screenRatio),
        };
        const descriptionBottomPosition = self.drawTextMultiple(ctx, matchDescription, matchDescriptionParam);

        // 绘制比赛赌注
        const matchTarget = self.contractInfo.target;
        const matchTargetParam = {
          y: Math.round(descriptionBottomPosition + 20 * self.sizeInfo.screenRatio),
          color: '#ff0018',
          fontSize: 24,
          lineHeight: 28,
        };
        const targetBottomPosition = self.drawTextMultiple(ctx, matchTarget, matchTargetParam);

        // 绘制比赛内容
        const matchExpect = self.contractInfo.promoterExpect;
        const matchExpectParam = {
          y: Math.round(targetBottomPosition + 5 * self.sizeInfo.screenRatio),
        };
        const expectBottomPosition = self.drawTextMultiple(ctx, matchExpect, matchExpectParam);

        // 绘制二维码
        const QRImageRatio = Number((self.QRCode.height / self.QRCode.width).toFixed(2));// 二维码宽高比
        const dWidth = 90 * self.sizeInfo.screenRatio;
        const dHeight = Math.round(dWidth * QRImageRatio);
        const dx = self.centerPositionX - dWidth * 0.5;
        const dy = Math.round(expectBottomPosition + 10 * self.sizeInfo.screenRatio);
        ctx.drawImage(self.QRCode.tempPath, dx, dy, dWidth, dHeight);

        // 根据最终元素的位置修正canvas的尺寸
        const finalBottomPosition = dy + dHeight;
        const distance = finalBottomPosition - self.sizeInfo.canvasHeight;
        if (distance >= 0) {
          self.sizeInfo.canvasHeight = self.sizeInfo.canvasHeight + distance + 20
        }

        // 全部绘制
        ctx.draw(false, function () {
          self.isDisabled = false;
          wx.hideLoading();
        });
      },

      // 将图片保存到相册
      saveImage() {
        console.log(123)
        const self = this;
        // 生成图片
        wx.showLoading({
          title: '图片加载中',
          mask: true
        });
        wx.canvasToTempFilePath({
          canvasId: 'shareCanvas',
          fileType: 'jpg', // 默认为png
          success(res) {
            // 获得图片临时路径
            self.filePath = res.tempFilePath;
            // 保存图片到系统相册
            wx.saveImageToPhotosAlbum({
              filePath: self.filePath,
              success(saveRes) {
                wx.hideLoading();
                wx.showToast({
                  title: '图片保存成功',
                  icon: 'success'
                })
              },
              fail(saveErr) {
                // TODO：fail提示
                wx.hideLoading();
              },
            })
          },
          fail(error) {
            // TODO：fail提示
            wx.hideLoading();
          }
        })
      },

      // 绘制单行文字
      drawTextSingle(ctx, text, { x = this.centerPositionX, y, maxWidth = this.maxTextWidth, fontSize = 12, color = '#666', textAlign = 'center' } = {}) {
        // 校验参数
        if (typeof text !== 'string' || typeof x !== 'number' || typeof y !== 'number') {
          return;
        }
        // 设置字体、颜色、对齐方式
        ctx.setFontSize(fontSize);
        ctx.setFillStyle(color);
        ctx.setTextAlign(textAlign);
        ctx.fillText(text, x, y, maxWidth);
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
       */
      drawTextMultiple(ctx, text, { x = this.centerPositionX, y, maxWidth = this.maxTextWidth, maxLines = 100, lineHeight = 20, fontSize = 12, color = '#666', textAlign = 'center' }) {

        // 校验参数
        if (typeof text !== 'string' || typeof x !== 'number' || typeof y !== 'number') {
          return;
        }

        // 设置字体、颜色、对齐方式
        ctx.setFontSize(fontSize);
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
        // const interval = 2;
        // for(let i = 0; i < arrText.length; i += interval) {
        //   currentText += arrText.slice(i, i + interval).join('');
        //   currentWidth = ctx.measureText(currentText).width;
        //   if (currentWidth > maxWidth) {
        //     ctx.fillText(currentText, x, y);
        //     currentText = '';
        //     y += lineHeight;
        //     currentLine++;
        //     // 如果已绘制的行数已达到限定绘制行数，后续文字放弃绘制
        //     if (maxLines && currentLine >= maxLines) {
        //       break;
        //     }
        //   }
        // }
        for (let letter of arrText) {
          currentText += letter;
          currentWidth = ctx.measureText(currentText).width;
          if (currentWidth > maxWidth) {
            ctx.fillText(currentText, x, y);
            currentText = '';
            y += lineHeight;
            currentLine++;
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
          currentLine++;
        }

        return y;
      },

    },
  }
</script>

<style scoped>
  .container {
    box-sizing: border-box;
    padding: 0 0 110rpx 0;
  }
  .share-canvas {
    position: relative;
    box-sizing: inherit;
    width: 100%;
    z-index: 1;

  }
  .save-button {
    box-sizing: inherit;
    position: fixed;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 100rpx;
    line-height: 100rpx;
    text-align: center;
    font-size: 36rpx;
    color: #FFF;
    background: #FF3D54;
    font-family: PingFangSC-Semibold sans-serif;
    z-index: 10;
  }
</style>
