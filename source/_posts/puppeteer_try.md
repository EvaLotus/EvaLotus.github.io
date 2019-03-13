---
title: 「 puppeteer 」puppeteer初尝试
date: 2019-02-13 18:20:35
tags: puppeteer
---
上一篇中我们通过自己手写代码完成了抓取list页中的基本数据，可是沙沙又提出了新的需求，如何抓取到每个酒店的装修时间和客房数量呢？我让她自己去研究八爪鱼去了，但是如果是真的撩妹，这可不就前功尽弃了。

正好最近在研究反作弊中判断是否是webdriver，headless chrome，发现一个神器puppeteer，不需要学习python，用nodeJS就可以搞定，我的键盘已经等不及了，用了一天时间研究，分享给大家。

其实主要是async和await的理解不够深刻，多花费了时间，实际上聪明如你半个小时应该就可以学会。

<!-- more -->
#### puppeteer

```bash
# 建个项目安装puppeteer
npm i puppeteer -S
```

官方demo就很容易上手，再加上awesome-puppeteer中的[例子](https://medium.com/@e_mad_ehsan/getting-started-with-puppeteer-and-chrome-headless-for-web-scrapping-6bf5979dee3e)，很容易就可以实现自己的目标。

```js
const puppeteer = require('puppeteer');

(async () => {
  const conf = {
    // 还是携程上海五角场江湾地区的url
    workUrl: 'http://hotels.ctrip.com/hotel/shanghai2/zone368#ctm_ref=hod_hp_sb_lst',
    // 设置ua，不然ua中包含headless，会被识别出来，拒绝提供服务
    ua: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36',
    viewport: {
      width: 1920,
      height: 1080,
    },
  };
  const browserSetting = {
    // 默认是headless的模式打开的,改为false可以打开实际的chrome，方便我们查看
    // 但是设置为true会快很多
    headless: false,
    // 或者直接打开指定path的chrome，最好还是使用默认提供的chromium
    executablePath: 'C:/Program Files (x86)/Google/Chrome/Application/chrome.exe',
    // 设置后可以操作慢点方便调试
    // slowMo: 250,
    // 打开F12
    devtools: true,
  };
  const browser = await puppeteer.launch(browserSetting);
  const page = await browser.newPage();
  // 页面设置
  await page.setViewport(conf.viewport);
  await page.setUserAgent(conf.ua);
  // await page.emulate(conf.device);
  // 页面跳转
  await page.goto(conf.workUrl);
  // page.$$(sel);= document.querySelectorAll(sel)
  // page.$(sel);= document.querySelector(sel)
  // page.$eval(sel);
  // page.$$eval(sel);
  // const els = await page.$$eval('p', els => els);// 奇怪的是这样得到的els里的元素都是{}
  // 改为
  // const elsHtml = await page.$$eval('p', els => els.map(el => el.innerHTML));
  // console.log(elsHtml);
  // 但个人觉得可以直接js实现的就不必用 puppeteer api，记一堆api不如用好js
  const hotels = await page.evaluate(async () => {
    // 这里可以直接执行js代码了
    const resArr = [];
    let timer = null;
    // 注意此处的异步操作
    async function getRes() {
      return new Promise((resolve) => {
        function getData() {
          // return new Promise((resolve, reject) => {
          // 因为ctrip本来就有jQuery，所以可以直接使用
          const num = $('.hotel_item').length;
          for (var i = 0; i < num; i++) {
            const item = $('.hotel_item:eq(' + i + ')');
            const hotel = {
              name: `${item.find('.hotel_name a').attr('title')}`,
              address: `${item.find('.hotel_item_htladdress').text().replace(/地图|街景/g, '')}`,
              url: `${item.find('.hotel_name a').attr('href').replace(/\?.*/g, '')}`,
              rate: `${item.find('.hotel_value').text()}`,
              price: `${item.find('.J_price_lowList').text()}`,
            };
            resArr.push(hotel);
          }
          let $nextBtn = $('.c_down');
          if ($nextBtn.length) {
            $nextBtn.click();
            timer = setTimeout(getData, 1000);
            $nextBtn = null;
          } else {
            clearTimeout(timer);
            resolve(resArr);
          }
        }
        getData();
      });
    }
    // 加debugger可以在打开的chrome里调试js
    // debugger;
    return getRes();
  });
})();
```

以上功能都可以直接按上一篇在chrome snippet中实现，但是如果需要自动获取detail信息，就需要puppeteer来帮我们操作了。

```js
// 开始获取detail
async function getDetail(h) {
  const nh = h;
  await page.goto(`http://hotels.ctrip.com${h.url}`);
  nh.info = await page.evaluate(() => $('#htlDes>p')[0].childNodes[0].data);
  return nh;
}
// 数量太多测试时间太长，先测试4个试下
hotels.length = 4;
// 此处await不能使用forEach，await不能放在循环中，使用promise.all
// 参见http://es6.ruanyifeng.com/#docs/async
const promises = hotels.map(h => getDetail(h));
await Promise.all(promises);
console.log(hotels);

```

node直接写入csv文件

```js
// 将得到的结果写入csv文件
fs.writeFile('hotels.csv', hotels, function(err) {
  if (err) {
    return console.error(err);
  }
  // 得到csv文件会有乱码问题，可以找框架来直接转为csv文件，此处不赘述
});

```

#### 总结

以上只是puppet的最基本常用的功能，还可以做很多诸如：自动化测试，前端性能测试，异常监控等。

如果你在尝试过程中也对异步操作比较困惑，可以一起复习下async和await和promise，参见这篇（TODO async和await）。

puppeteer和Selenium/WebDriver的主要区别：

Selenium/WebDriver适用于多种浏览器

puppeteer主要作用于chromium，做到极致

我个人觉得API很多，看了也记不住，用到再去查。
