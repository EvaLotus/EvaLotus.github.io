---
title: 「 HTTP 」通过axios来理解http请求
date: 2017-05-18 17:46:53
tags: HTTP
---

刚开始做小程序对接的时候，很多人一上来就问为啥这个接口请求不成功，完全都不知道自己看看HTTP的请求信息，因为 `wx.request` 没有设置`content-type`，默认的 `content-type` 是 `application/json`。但是我们平常后端接受的一般都是 `application/x-www-form-urlencoded`。用惯了jQuery的$.ajax今天来说说请求的细节。
<!-- more -->
#### Request Payload& FormData

1. Request Payload

对应content-type：application/json，**axios默认使用的是此content-type**

```
POST /some-path HTTP/1.1
Content-Type: application/json
Request Payload:
view parsed: { "foo" : "bar", "name" : "John" }
view source：{ "foo" : "bar", "name" : "John" }
没有viewencode，因为不encode
```

content-type：application/json上传数据可以更多样，可以直接传递数组，对象
`{ "foo" : "bar", "name" : "John"，goods:['fish','beef'] }`

**后台处理**

对于 Request Payload 请求， 必须加`@RequestBody`才能将请求正文解析到对应的 bean 中，且只能通过 request.getReader\(\) 来获取请求正文内容

```
@PostMapping("/accounts")
public ResponseEntity<?> createAccount(@RequestBody SysAccount account) {
```

1. FormData

**常见的content-type一般都是application/x-www-form-urlencoded**，这是jQuery默认的content-type

对应application/x-www-form-urlencoded：正文请求类似get url请求

```
POST /some-path HTTP/1.1
Content-Type: application/x-www-form-urlencoded
FormData:
view source:foo=bar&name=John
view source中会encode之后直接上送
```

**后台处理**

对于 Form Data 请求，无需任何注解，springmvc 会自动使用 MessageConverter 将请求参数解析到对应的 bean，且通过 request.getParameter\(...\) 能获取请求参数

```
@PostMapping("/accounts")
public ResponseEntity<?> createAccount(SysAccount account) {
```

前端可以通过qs来将request payload转为form data上送

```js
import 'Qs';
// axios默认将Content-type设为`application/json`
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
axios.post(url, { data: Qs.stringify(para) }).then((res) => {
resolve(res);
}).catch((err) => {
})
```

Qs.stringify和JSON.stringify的区别

```js
Qs.stringify( {name:'hehe',age:10})=> name=hehe&age=10
JSON.stringify( {name:'hehe',age:10})=>"{"a":"hehe","age":10}"
```

#### 上传文件

需要在header中设置`{ 'Content-Type': 'multipart/form-data' }`

```
POST /some-path HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryP1hWgP9UXLkUFJEd

Request Payload

------WebKitFormBoundaryP1hWgP9UXLkUFJEd
Content-Disposition: form-data; name="id"

12345
------WebKitFormBoundaryP1hWgP9UXLkUFJEd # 通过boundary来分隔各个请求参数
Content-Disposition: form-data; name="ips"

["1.2.3.1","1.2.3.4"] //注意在formData中无法上送实际list，都会被转为string
------WebKitFormBoundaryP1hWgP9UXLkUFJEd
Content-Disposition: form-data; name="file"; filename="my.file"
Content-Type: text/plain

------WebKitFormBoundaryP1hWgP9UXLkUFJEd--
```



调用示例：

```js
const formData = new FormData();
formData.append('file', file);
formData.append('id', id);
formData.append('ips', JSON.stringify(['1.2.1.1','2.1.2.1']));//后台还是需要配合解析
axios({
method: 'post',
url,
data: formData,
headers: { 'Content-Type': 'multipart/form-data' },//需要设置Content-Type
}).then((res) => {
}).catch((err) => {
});
```

#### DELETE的使用

可以看到axios中传参config中可以使用data和params

```js
// `params` are the URL parameters to be sent with the request
// Must be a plain object or a URLSearchParams object
//
params: {
ID: 12345
},
// `data` is the data to be sent as the request body
// Only applicable for request methods 'PUT', 'POST', and 'PATCH'
// When no `transformRequest` is set, must be of one of the following types:
// - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
// - Browser only: FormData, File, Blob
// - Node only: Stream, Buffer
// data传参是在request body中，是向服务器发送资源，只支持PUT，POST，PATCH。因为GET只支持在params中传参，显示时是Query String Parameters

// DELETE也只支持在params的原因是一般都是只上送个id来删除，没有必要在request body中传递，直接在Query String Parameters中传递即可
data: {
firstName: 'Fred'
},
```

浏览器中显示

```
POST /some-path HTTP/1.1
Content-Type: application/x-www-form-urlencoded
---------------Query String Parameters-----------------
拼接在url后面的就是此种
view source:
id:12345
view parsed:
id=12345
-----------FormData-----------------
view parsed:
foo:bar
name:Eva
view source：
foo=bar&name=Eva
```

GET请求对应Query String Parameters，是没有对应的Content-type的

或对应Content-type：text/plain

在request body中传递是对应Request Payload或FormData

#### 支持CSRF攻击

```
// 设置CSRFToken
axios.defaults.xsrfCookieName = 'csrftoken';
axios.defaults.xsrfHeaderName = 'X-CSRFToken';
```

#### 关于urlencode

前文提到FormData对应的Content-type为application/x-www-form-urlencoded以及GET请求对应的Query String Parameters，此类请求中上送参数都会被浏览器自动encode后上送

但是需要注意的是有时候浏览器encode的并不是我们想要的，比如下面这个陈年老坑

https://mi.com?id=1+2+3
实际后台收到的参数是id=1 2 3,上送时+变成了空格，所以需要前端`encodeURIComponent('1+2+3')=&gt;1%2B2%2B3`再上送
实际上会encode成https://mi.com?id=1 2 3，参数还是id=1 2 3

总之遇上+号时，前端最好encodeURIComponent后上送

参考文献：
[https://fed.renren.com/2018/02/03/http-request/](https://fed.renren.com/2018/02/03/http-request/)
