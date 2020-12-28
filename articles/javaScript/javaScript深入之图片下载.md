### 在不依赖后端的情况下，JS如何下载资源？

简单，创建临时`a`标签，手动触发点击事件。伪代码如下：

```js
    const fileName = new Date().getTime().toString();
    const a = document.createElement('a');
    a.href = [ 地址 ];
    a.download = `${fileName}.jpg`;
    a.target = 'download';
    a.click();
```

优点：简单，对大多数资源有效 缺点：对图片资源无效

### 为何对图片资源无效？

`a`标签的默认事件是打开资源链接。现代浏览器（以谷歌最新版为例）实现了当资源链接为一个图片时，会默认在新窗口打开该资源，而非下载。

### 那针对以上情况图片资源如何下载？

+

```js
    /**
 * 根据image生成canvas
 * */
function generateCanvas(img) {
    const canvas = document.createElement('canvas');
    canvas.width = img.width;
    canvas.height = img.height;
    const ctx = canvas.getContext('2d');
    ctx.drawImage(img, 0, 0, img.width, img.height);
    return canvas;
}

function generateImage(src, callback) {
    return new Promise((resolve, reject) => {
        const image = new Image();
        image.src = src + '?v=' + Math.random(); // 处理缓存
        image.crossOrigin = '*';  // 支持跨域图片
        function onLoadImage() {
            const canvas = generateCanvas(image);
            if (!canvas) {
                reject('canvas不存在，未绘制任何数据');
            }
            resolve(canvas)
        }

        image.onload = onLoadImage;
    })
}

function generateLinkElement(imageBlob) {
    const fileName = new Date().getTime().toString();
    const imageFile = new File([ imageBlob ], `${fileName}.jpg`);
    const a = document.createElement('a');
    a.href = URL.createObjectURL(imageFile);
    a.download = `${fileName}.jpg`;
    a.target = 'download';
    a.click();
}

function downloadImage(url) {
    generateImage(url).then(canvas => {
        canvas.toBlob(blob => {
            generateLinkElement(blob)
        }, 'image/jpeg', 90);
    })
}

downloadImage('http://192.168.8.61:8800/file/work/5fe40092d5279e64862e52f5/images/20201224125100817146.jpeg')
```
https://www.cnblogs.com/minigrasshopper/p/9042686.html