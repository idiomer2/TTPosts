## ① HTML单页应用

用AI通过vibe coding生成

## ② 改造成PWA

1. html文件的head标签添加如下代码

```html
<!-- ./index.html -->

<html lang="zh-CN">
<head>


    <!-- PWA改造 -->
    <link rel="manifest" href="./manifest.json">
    <link rel="apple-touch-icon" href="./icon.png">
    <meta name="theme-color" content="#000000">
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

2. 同目录下新建manifest.json和sw.js
```json
// manifest.json
{
    "name": "密码管理器",
    "short_name": "密码管理器",
    "description": "密码管理器纯HTML单页应用",
    "start_url": "/mypass/index.html",
    "display": "standalone",
    "background_color": "#ffffff",
    "theme_color": "#2196f3",
    "orientation": "portrait",
    "icons": [
      {
        "src": "/mypass/icon.png",
        "sizes": "128x128",
        "type": "image/png"
      }
    ]
}
```

```javascript
const CACHE_NAME = 'pwa-cache-v1';
// 需要缓存的文件列表
const urlsToCache = [
  '/mypass/index.html',
  '/mypass/icon.png',
  '/mypass/favicon.ico',
  'https://cdn.bootcdn.net/ajax/libs/vue/3.3.4/vue.global.prod.min.js',
  'https://cdn.bootcdn.net/ajax/libs/crypto-js/4.1.1/crypto-js.min.js',
  '/mypass/tailwindcss_3.4.17.min.css'
];
// 1. 安装事件：Service Worker 安装时触发，缓存指定文件
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  );
});
// 2. 拦截请求：每次网页发起请求时触发
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // 如果缓存里有，直接返回缓存（实现离线访问）
        if (response) {
          return response;
        }
        // 如果缓存没有，就去网络请求
        return fetch(event.request);
      })
  );
});
// 3. 激活事件：用于清理旧缓存（版本更新时）
self.addEventListener('activate', event => {
  const cacheWhitelist = [CACHE_NAME];
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

3. 当前目录添加web图标（favicon.ico）和app图标（icon.png）。其中app图标的分辨率不能太小，建议至少384x384
4. 最后起个WEB服务能访问当前目录下的这些资源即可（mypass/\*）


## ③ tailwindcss资源优化

如果用的是tailwindcss，且未指定版本或主版本为3或以上，那么web页面打开时会比较，因为版本3.x开始采用js扫描html，然后动态添加css样式，而非以前一整个css静态文件。因此，官方建议开发时直接引用`<script src="https://cdn.tailwindcss.com/3.4.17"></script>`，上生产的话，就要手动生成css静态文件引用，具体如下：

1. 安装nodejs(自带npm和npx)
2. 在html的目录下，执行`npm install -D tailwindcss@3.4.17 `生成本地依赖
3. 再执行`npm install` 实际安装tailwindcss
4. 修改步骤2生成的`tailwind.config.js`文件，将`content: [],`改为`content: ["./*.html"],`以扫描当前目录的所有html
5. 执行`npx tailwindcss -o ./tailwindcss_3.4.17.min.css --minify`生成tailwindcss_3.4.17.min.css文件
6. 在html文件中引用 `<link href="./tailwindcss_3.4.17.min.css" rel="stylesheet">`，并删除原来的`<script src="https://cdn.tailwindcss.com/3.4.17">
7. 在sw.js中添加tailwindcss_3.4.17.min.css的缓存

这样，web页面打开时就会快很多；PWA应用也能缓存所有静态资源，打开速度也明显加快，且可以断网使用*★,°*:.☆(￣▽￣)/$:*.°★* 。