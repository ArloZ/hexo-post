---
title: tornado+nginx上传视频文件
date: 2014-06-27 17:04:36
category: tornado
tags: [tornado,upload-file,jquery]
---

由于tornado通过表达上传的数据最大限制在100M，所以如果需要上传视屏文件的情况在需要通过其他方式实现，
此处采用nginx的[nginx-upload-module](http://www.grid.net.ru/nginx/upload.en.html)和[jQuery-File-Upload](https://github.com/blueimp/jQuery-File-Upload)实现。

### 1.编译安装nginx-upload-module

- 下载nginx-1.5.8
- 下载nginx-upload-module2.0
- 由于nginx-upload-module不支持最新版的nginx，直接编译会出错，需要打补丁 [davromaniak.txt](http://paste.davromaniak.eu/index.php?show=110)

``` bash
$ tar xzf nginx-1.5.8
$ tar xzf nginx_upload_module-2.0.12.tar.gz
$ cd nginx_upload_moule-2.0.12
$ patch ngx_http_upload_module.c davromaniak.txt
$ cd ../nginx-1.5.8
$ ./configure --add-module=../nginx_upload_moule-2.0.12
$ make & sudo make install
```


### 2.配置nginx的upload-module
```
upstream tornadoserver{
    server 127.0.0.1:8001;
}
server {
listen       8080;
server_name  localhost;
location / {
    proxy_pass_header Server;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_pass http://tornadoserver;
}
client_max_body_size 4G;
client_body_buffer_size 1024k;

if ($host !~* ^(localhost) ) {
}
location = /uploads {
    if ($request_method = OPTIONS) {
        add_header Pragma no-cache;
        add_header X-Content-Type-Options nosniff;

        # Access control for CORS
        add_header Access-Control-Allow-Origin "http://localhost";
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "cache-control, content-range, accept,\
            origin, session-id, content-disposition, x-requested-with, ctent-type,\
            content-description, referer, user-agent";
        add_header Access-Control-Allow-Credentials "true";
        # 10 minute pre-flight approval
        add_header Access-Control-Max-Age 600;
        return 204;
    }
    if ($request_method = POST) {
        add_header Pragma no-cache;
        add_header X-Content-Type-Options nosniff;
        #add_header Cache-control "no-story, no-cache, must-revalidate";

        # Access control for CORS
        add_header Access-Control-Allow-Origin "http://localhost";
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "cache-control, content-range, accept,\
                origin, session-id, content-disposition, x-requested-with,\
                content-type, content-description, referer, user-agent";
        add_header Access-Control-Allow-Credentials "true";
        # 10 minute pre-flight approval
        add_header Access-Control-Max-Age 600;

        upload_set_form_field $upload_field_name.name "$upload_file_name";
        upload_set_form_field $upload_field_name.content_type "$upload_content_type";
        upload_set_form_field $upload_field_name.path "$upload_tmp_path";

        upload_pass_form_field "^X-Progress-ID$|^authenticity_token$";
        upload_cleanup 400 404 499 500-505;
    }

    upload_pass @fast_upload_endpoint;

    # {a..z} not usefull when use zsh
    # mkdir {1..9} {a..z} {A..Z}
    upload_store /tmp/uploads 1;

    # set permissions on the uploaded files
    upload_store_access user:rw group:rw all:r;
    }
    location @fast_upload_endpoint {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass_header 'Access-Control-Allow-Origin';
        proxy_pass http://tornadoserver;
    }
}
```

### 3.demo页面

``` javascript
<!DOCTYPE HTML>
<html>
<head> <meta charset="utf-8">
    <title>jQuery File Upload Example</title>
    <style>
    .bar {
        height: 18px;
        background: green;
    }
</style></head>
<body>
    <input id="fileupload" type="file" name="files[]" data-url="uploads" multiple>
    <script src="{{static_url('js/jquery-1.10.2.min.js')}}"></script>
    <script src="/static/js/vendor/jquery.ui.widget.js"></script>
    <script src="/static/js/jquery.iframe-transport.js"></script>
    <script src="/static/js/jquery.fileupload.js"></script>
    <script>
        $(function () {
            $('#fileupload').fileupload({
                dataType: 'json',
                done: function (e, data) {
                    $.each(data.result.files, function (index, file) {
                       $('<p/>').text(file.name+"  "+file.size).appendTo(document.body);
                    });
                },
                progressall: function (e, data) {
                    var progress = parseInt(data.loaded / data.total * 100, 10);
                    $('#progress .bar').css( 'width', progress + '%');
                }
            });
        });
    </script>
    <div id="progress">
        <div class="bar" style="width: 10%;"></div>
    </div>
</body>
</html>
```

### 4.tornado处理
当nginx的upload-module完成视频文件的传输之后，其会设置表单数据，并转发给后台tornado服务器处理。

通过如下方式获得相关参数：

``` python
name = self.get_argument("files[].name","")
content_type = self.get_argument("files[].content_type","")
oldpath = self.get_argument("files[].path","")
size = os.path.getsize(oldpath)
files = []
files.append({"name":name,"type":content_type,"size":size})
ret = {"files":files}
self.write(tornado.escape.json_encode(ret))
```
