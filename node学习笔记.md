<!--
  -- --------------------------------------------------------
  -- @file node学习笔记.md
  -- @author jinxuan_lin <jinxuan_lin@kingdee.com>
  -- @date 2019-01-11 14:00:36
  -- @last_modified_by jinxuan_lin <jinxuan_lin@kingdee.com>
  -- @last_modified_date 2019-01-11 14:22:35
  -- @copyright (c) 2019 @yfe/weekly-report
  -- --------------------------------------------------------
 -->

# nodeJs 学习笔记

## 内置模块

### 资源压缩 -zlib

浏览器通过在 http 请求头部加上 Accept-Encoding,告诉服务可以用 gzip,或者 deflate 算法压缩资源,
服务器可以对资源进行压缩，再返回给浏览器，一次节省流量，加快访问速度。

```
Accept-Encoding: gzip,defalte
```

#### 简单的压缩和解压缩

```
// 压缩
var fs = require('fs');
var zlib = require('zlib');
var gzip = zlib.createGzip();

var inFile = fs.createReadStream('file.txt');
var out = fs.createWriteStream('file.txt.gz');

inFile.pipe(gzip).pipe(out);

// 解压缩
var fs = require('fs');
var zlib = require('zlib');

var gunzip = zlib,createGunzip();

var inFile = fs.createReadStream('file.txt.gz');
var outFile = fs.createWriteStream('file.txt');

inFile.pipe(gunzip).pipe(outFile);
```
