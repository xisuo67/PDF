# 前端生成pdf文件之pdfmake.js
pdfmake.js是一个简单的生成pdf文件的插件。

pdfmake.js https://files.cnblogs.com/files/s313139232/pdfmake.min.js  

参考：https://www.cnblogs.com/pheye/p/5661933.html


## pdfmake的基本使用方法  
1.包含以下两个文件  
```JS
    <script src="build/pdfmake.min.js"></script>  
    <script src="build/vfs_fonts.js"></script>  
```
2.在JS代码中声明一个Document-definition对象，这个是pdfmake自己的术语。简单点说，就是创建一个至少包含content属性的对象。然后就可以调用pdfMake的方法导出PDF，具体见如下代码：
```JS
    <script type="text/javascript">
    //创建Document-definition对象 
    var dd = {
    content: [
    'One paragraph',
    'Another paragraph, this time a little bit longer to make  sure, this line will be divided into at least two lines'
    ]
    };
    //导出PDF
    pdfMake.createPdf(dd).download();
    </script>
```
## 插入图片  

在插入图片方面，jsPDF要求先将图片转换成Data URL才行，而pdfmake允许直接指定路径，看起来是很方便，但这是有条件的，必须是以node.js作为服务器，或者将图片放到vfs_fonts.js中，所以总的来说，用处不大，还是一样得将图片转换成Data URL形式才行。

为解决此问题，我写了一个ImageDataURL的函数对象，可同时传入多个图片地址。在图片都加载完成后，ImageDataURL.oncomplete将被触发，在回调中通过this.imgdata取出各个图片的Data URL，根据pdfmake的要求组织下，就可正确生成pdf了。

ImageDataURL的原理是通过H5的canvas标签，将图片绘制在canvas上，然后通过canvas的toDataURL得到图像的Data URL。使用时请注意浏览器兼容性问题。

以下为将sampleImage.jpg, sampleage.jpg, sampleImage.jpg依次写入PDF的例子，测试时sampleage.jpg不存在，PDF直接忽略。  
```HTML
    <!DOCTYPE html>
    <html lang="zh-CN">
        <head>
        <meta charset="utf-8">
        <title>my second export PDF</title>
        <script src="build/pdfmake.min.js"></script>
        <script src="build/vfs_fonts.js"></script>
        <script>
            function down() {
                var x = new ImageDataURL(["sampleImage.jpg", "samplage.jpg", "sampleImage.jpg"]);
                x.oncomplete = function() {
                    var imgs = new Array();
                    console.log("complete");
                    for (key in this.imgdata) {
                        if (this.imgdata[key] == this.emptyobj){
                            continue;
                        }  
                        imgs.push({image:this.imgdata[key]});
                    }
                    var dd = {
                        content: [
                        'Title',
                        imgs,
                        ],
                    };
                    pdfMake.createPdf(dd).download();
                }
            }
        </script>
        </head>
        <body>
        <button onclick="down()">下载</button>
        <script>
        //urls必须是字符串或字符串数组
        function ImageDataURL(urls) {
            this.completenum = 0;
            this.totalnum = 0;
            this.imgdata = new Array();
            this.emptyobj = new Object();
            this.oncomplete = function(){};
            this.getDataURL = function(url, index) {
                var c = document.createElement("canvas");
                var cxt = c.getContext("2d");
                var img = new Image();
                var dataurl;
                var p;
                p = this;
                img.src = url;
                img.onload = function() {
                    var i;
                    var maxwidth = 500;
                    var scale = 1.0;
                    if (img.width > maxwidth) {
                        scale = maxwidth / img.width;
                        c.width = maxwidth;
                        c.height = Math.floor(img.height * scale);
                    } else {
                        c.width= img.width;
                        c.height= img.height;
                    }
                    cxt.drawImage(img, 0, 0, c.width, c.height);
                    p.imgdata[index] = c.toDataURL('image/jpeg');
                    for (i = 0; i < p.totalnum; ++i) {
                        if (p.imgdata[i] == null)
                            break;
                    }
                    if (i == p.totalnum) {
                        p.oncomplete();
                    }
                };
                img.onerror = function() {
                    p.imgdata[index] = p.emptyobj;
                    for (i = 0; i < p.totalnum; ++i) {
                        if (p.imgdata[i] == null)
                            break;
                    }
                    if (i == p.totalnum) {
                        p.oncomplete();
                    }
                };
            }
            if (urls instanceof Array) {
                this.totalnum = urls.length; 
                this.imgdata = new Array(this.totalnum);
                for (key in urls) {
                    this.getDataURL(urls[key], key);
                }
            } else {
                this.imgdata = new Array(1);
                this.totalnum = 1;
                this.getDataURL(urls, 0);
            }
        }
        </script>
        </body>
    </html>
```
