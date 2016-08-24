---
layout: post
title: XMLHttpRequest Upload 及 CORS 研究
date: 2011-08-02 03:08:50
categories: frontend
tags: js, upload
---

首先，在开始这篇 Blog 之前，我并不知道研究能否成功。

虽然 SWFUpload 给了很多方便，但是，由于浏览器和 Flash 本身的毛病，SWFUpload 也带入了不少的 Accessibility 的 bug。所以，老大让我研究 HTML5 的 file input 多重上传及进度反馈。

## 基础

除了 IE，其他内核的浏览器都支持 file input 的 `multiple` 属性，也就是可以多选文件；除了 IE，其他内核的浏览器都支持 `XMLHttpRequest` 对象，而上传文件及进度反馈又必须要靠它。所以，在这里，只针对 IE 内核以外的浏览器进行测试。

## HTML

搞个最简单的 HTML 页面：

```html
<input type="file" id="file" name="file" />
<button id="upload">Upload</button>
```

## JS

为了之后的整合，分开的 JS 片段自成 `function`。

### 获取 file 对象

如果给了 `multiple` 属性，`files` 会大于 `1`。因为目前的研究重点是上传，先取第一个，`file` 为内置的 `File` 类的实例，`file instanceof File ---> true`。

```js
var getFile = function() {
  return document.getElementById("file").files[0];
};
```

### 准备上传用的 XHR 对象

```js
var prepareXhr = function(file) {
  var xhr = new XMLHttpRequest();
  xhr.upload.onprogress = function(e) {
    console.info("progress", (e.loaded / file.size * 100).toFixed(0) + "%");
  };
  xhr.onreadystatechange = function() {
    if (xhr.readyState == 4) {
      if (xhr.status == 200 || xhr.status == 304) {
        alert("uploaded - " + xhr.responseText);
      } else if (xhr.status === 0) {
        alert("upload aborted");
      } else {
        alert("upload exception with status - " + xhr.status);
      }
    }
  };
  xhr.open("POST", "submit.jsp", true);
  return xhr;
};
```

### 上传

有两种方式，一种是 [AjaxUpload](http://valums.com/ajax-upload/) 使用的方式，采用 `application/octet-stream` 作为 `request header` 的 `Content-Type`。这种方式的比较简单，但是只能 send file 对象，其他的附加数据，甚至包括 `filename` 都无法加到 post 数据。所以加个 `X-File-Name` 的 `request header` 来让 server 端获得 `filename`。

```js
var doUpload = function() {
  var file = getFile();
  if (!file) {
    return;
  }

  var xhr = prepareXhr(file);
  xhr.setRequestHeader("Content-Type", "application/octet-stream");
  xhr.setRequestHeader("X-File-Name", file.name);
  xhr.send(file);
};
```

FF4 引入了 `FormData`，可以方便的把文件和其他数据组装起来，这种情况下 XHR 会自动检测到 `Content-Type`，不需要设置。

```js
var fromData = new FormData();
fromData.append("filedata", file);
fromData.append("filename", file.name);
xhr.send(fromData);
```

另外一种方式是，使用 `multipart/form-data` 为 `Content-Type`，但这个比较复杂，需要对文件**内容**进行前端编码，注意是文件内容，所以，这需要在 JS 端就能把文件的内容读出来，于是用到新的 JS API - [FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)。

```js
var readFile = function(file, callback) {
  var reader = new FileReader();
  reader.onload = function(e) {
    if (callback) {
      callback(e.target.result);
    }
  };
  reader.onerror = function(e) {
    alert("file read error");
  };
  reader.readAsBinaryString(file);
};
```

有了文件数据，需要手工组装 MIME 对象（可以参考 `FormData` 时的 Post），同时设置的 `request header`：

```js
readFile(file, function(fileData) {
  var boundary = "-------xxxxxxxxx";

  xhr.setRequestHeader("Content-Type", "multipart/form-data, boundary=" + boundary);// simulate a file MIME POST request
  xhr.setRequestHeader("Content-Length", file.size);

  xhr.sendAsBinary = xhr.sendAsBinary || function(datastr) {
    var ords = Array.prototype.map.call(datastr, function(x) {
        return x.charCodeAt(0) &amp; 0xFF;
      }),
      unit8Arr = new Uint8Array(ords);

    this.send(unit8Arr.buffer);
  };
  xhr.sendAsBinary(["--" + boundary,
    "Content-Disposition: form-data; name="filedata"; filename="" + file.name + """,
    "Content-Type: " + file.type + "rn",// extra "rn"
    fileData,
    "--" + boundary,
    "Content-Disposition: form-data; name="filename"rn",// extra "rn"
    file.name,
    "--" + boundary + "--"].join("rn"));// forming a multipart MIME manually
});
```

为了方便测试 把 HTML 改成：

```html
<input type="file" id="file" name="file" />
<button id="upload-octet">upload-octet</button>
<button id="upload-formdata">upload-formdata</button>
<button id="upload-manual">upload-manual</button>
```

并在 `window.onload` 时初始化：

```js
document.getElementById("upload-octet").onclick = function() {
  doUpload("octet");
};
document.getElementById("upload-formdata").onclick = function() {
  doUpload("formdata");
};
document.getElementById("upload-manual").onclick = function() {
  doUpload("manual");
};
```

## 后台代码

### application/octet-stream

依赖 `apache-commons-io`

```java
String contentType = request.getHeader("Content-Type");

File uploaderFolder = new File(request.getServletContext().getRealPath("/fileuploader/uploadfolder"));
if (!uploaderFolder.exists()) {
  uploaderFolder.mkdir();
}
InputStream fis = null;
FileOutputStream fos = null;

try {
  fis = request.getInputStream();
  fos = new FileOutputStream(new File(uploaderFolder, request.getHeader("X-File-Name")));
  IOUtils.copy(fis, fos);
  response.setStatus(HttpServletResponse.SC_OK);
  fos.close();
  fis.close();
} catch (Exception ex) {
  throw ex;
} finally {
  if (fis != null) {
    fis.close();
  }
  if (fos != null) {
    fos.close();
  }
}
out.print(contentType);
```

### multipart/form-data

依赖 `apache-commons-fileupload`

```java
String contentType = request.getHeader("Content-Type");

File uploaderFolder = new File(request.getServletContext().getRealPath("/fileuploader/uploadfolder"));
if (!uploaderFolder.exists()) {
  uploaderFolder.mkdir();
}

ServletFileUpload upload = new ServletFileUpload(new DiskFileItemFactory());
List items = upload.parseRequest(request);
Iterator iter = items.iterator();

FileItem fileItem = null;
String filename = null;
while (iter.hasNext()) {
  FileItem item = (FileItem) iter.next();

  if (item.isFormField()) {
    if ("filename".equalsIgnoreCase(item.getFieldName())) {
      filename = item.getString();
    }
  } else {
    fileItem = item;
  }
}
fileItem.write(new File(uploaderFolder, filename));

out.print(contentType);
```

以上两段代码，可以用 `if ("application/octet-stream".equals(contentType)) ... else ...` 整合在 `submit.jsp` 中。

## CORS

写了这么多，终于到重点了 - **Cross Origin Resource Sharing**，其实 CORS 无非就是 AJAX 跨域调用的问题，以下情况会产生跨域：

1. 不同域名 `http://www.xxx.com` v.s. `http://www.yyy.com`
2. 同域名，不同端口 `http://www.xxx.com` v.s. `http://www.xxx.com:8080`
3. 同域名，不同协议 `http://www.xxx.com` v.s. `https://www.xxx.com`
4. 域名和 IP，即使域名对应的是 ip `http://www.xxx.com` v.s. `http://10.224.170.168`
5. 主域名相同，子域名不同 `http://www.xxx.com` v.s. `http://sub.xxx.com`

这样，本地测试要模拟跨域就很简单了 - 只需要 host 里面对 `127.0.0.1` 发派两个不同的域名即可。

从 Firebug 的「Net」标签页可以看到，当 CORS 时，一个 AJAX 会分成两个请求发出。

我做了一个很简单的 POST 请求，结果如下：

![](images/posts/cors_ajax_2_calls.png)

两次请求的区别，已经圈出来了。主要的区别是：

* 第一次请求类型为 `OPTION`，而真正的请求为 `POST`
* `OPTION` 请求没有 `Content-Type` 后台取出来为 `null` - 估计 这可以作为 `OPTION` 请求的一个辨别方式

所以，现在看来，CORS Upload via AJAX 理论上是可以实现的，但是...目前只测了 FF5 的情况。对三种上传的方案，也没有具体深入的研究。

## 参考

* [W3C XMLHttpRequest Level 2](http://www.w3.org/TR/XMLHttpRequest2/)
* [Mozilla Developer - Using XMLHttpRequest](https://developer.mozilla.org/En/Using_XMLHttpRequest)
* [HTML5 Powered AJAX Upload](http://blog.new-bamboo.co.uk/2010/7/30/html5-powered-ajax-file-uploads)
* [New Tricks in XMLHttpRequest2](http://www.html5rocks.com/en/tutorials/file/xhr2/)
* [W3C CORS](http://www.w3.org/TR/cors/)
* [XHR2 & Cross-Origin Resource Sharing](http://tiffanybbrown.com/xhr2/)
* [How to upload arbitrary file contents cross-domain](http://blog.kotowicz.net/2011/04/how-to-upload-arbitrary-file-contents.html)
