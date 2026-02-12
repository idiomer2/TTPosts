## ① HTML单页应用

用AI通过vibe coding生成




## ② 改造成PWA

1. html文件的head标签添加如下代码

```html
<!-- index.html -->

<html lang="zh-CN">
<head>


    <!-- PWA改造 -->
    <link rel="manifest" href="./manifest.json">
    <meta name="theme-color" content="#000000">
    <link rel="apple-touch-icon" href="./icon.png">
    <script>
        // 检查浏览器是否支持 Service Worker
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('./sw.js')
                    .then(registration => {
                        console.log('SW 注册成功，作用域: ', registration.scope);
                    })
                    .catch(err => {
                        console.log('SW 注册失败: ', err);
                    });
            });
        }
    </script>
 
 
</head>
```




