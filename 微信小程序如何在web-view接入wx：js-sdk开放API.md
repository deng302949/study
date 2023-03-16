# 微信小程序如何在web-view接入wx：js-sdk开放API

- 需在script标签引入JS-SDK文件
- 需在微信小程序后台配置
  - 开发设置->  服务器域名中request合法域名写入API备案域名
  - 开发设置->  业务域名中配置webview中前端路径域名
- 需在微信公众号处配置
  - 公众号-> 功能设置 ， 设置业务域名 同上（开发设置业务域名)
  - 公众号-> 功能设置 ， 设置js安全域名 同上（开发设置服务器域名)

- 需服务器给出相应的信息[timestamp、nonceStr、signature]
  验证signature：https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign

  - 需传入当前页面的路径换取（备注： 此路径需在JS安全域名配置）

  ```
  wx.config({
      debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
      appId: 'wx1189ad3bae357bcf', // 必填，公众号的唯一标识
      timestamp: timestamp, // 必填，生成签名的时间戳
      nonceStr: noncestr, // 必填，生成签名的随机串
      signature: signature, // 必填，签名
      jsApiList: ['scanQRCode', 'chooseImage'], // 必填，需要使用的JS接口列表（扫一扫接口）
    });
  ```

- react代码如下
  ```
  // r为结果
  // 以下为调用方式， callback 取回调数据， type 调用的功能 scanQRCode、chooseImage
  // wxGetQRcode((r) => {
  //   console.log(r);
  // }, 'scanQRCode' || 'chooseImage');
  const wxGetQRcode = async (callback, type) => {
    // https://storagets.gonest.cn/storageMange/homePage
    let res;
    try {
      res = await Http.post(
        `${API_HOST}/wxSign/getSignature?url=${window.location.href}`,
      );
    } catch (e) {
      // alert('调用扫码前获取参数错误');
    }
    // sdk未注入
    if (!wx || !res.data) return;
    const { noncestr, signature, timestamp } = res.data;
    wx.config({
      debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
      appId: 'wx1189ad3bae357bcf', // 必填，公众号的唯一标识
      timestamp: timestamp, // 必填，生成签名的时间戳
      nonceStr: noncestr, // 必填，生成签名的随机串
      signature: signature, // 必填，签名
      jsApiList: ['scanQRCode', 'chooseImage'], // 必填，需要使用的JS接口列表（扫一扫接口）
    });
    if (!type) {
      return;
    }
    wx.ready(function () {
      type === 'scanQRCode' &&
        wx.scanQRCode({
          needResult: 1, // 默认为0，扫描结果由微信处理，1则直接返回扫描结果，
          scanType: ['qrCode', 'barCode'], // 可以指定扫二维码还是一维码，默认二者都有
          success: function (res) {
            setTimeout(() => {
              callback(res);
            }, 1000);
          },
          error: function (err) {
            alert('打开摄像头错误');
          },
        });
      type === 'chooseImage' &&
        wx.chooseImage({
          count: 1, // 默认9
          sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
          sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
          success: function (res) {
            setTimeout(() => {
              callback(res);
            }, 1000); // 返回选定照片的本地 ID 列表，localId可以作为 img 标签的 src 属性显示图片
          },
        });
    });
    wx.error((err) => {
      console.log('初始化错误！');
      // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
    });
  };
  ```

  **备注: ** 需要在首页 执行一次config文件

  JS-SDK： https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html