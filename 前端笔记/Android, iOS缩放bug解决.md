## 场景复现

1. 用的是cordova hybrid应用
2. 业务需求是某一个页面需要支持缩放（默认所有的都是不支持缩放的，看下面配置）
3. 初始的设置是  
`<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=no">`

## 解决方案
1. 用hammer.js手势库控制css scale 最终放弃了，因为要计算各种东西）。
2. 进入页面的时候，设置允许缩放，退出页面的时候，再设置回不允许缩放。

## 遇见的bug及解决方案
1. 在Andriod上，user-scalable=yes不生效。添加下面代码即可，一行也不能少。
```java
WebSettings settings = ((WebView)appView.getEngine().getView()).getSettings();
settings.setUseWideViewPort(true);
settings.setLoadWithOverviewMode(true);
settings.setSupportZoom(true);
settings.setBuiltInZoomControls(true);
settings.setDisplayZoomControls(false);
```
2. 在IOS上，如果以放大状态退出这个页面，外面还是放大状态，且没办法缩小。需要在退出之前先还原到初始大小(通过设置maximum-scale=1)。
```js
addZoomSupport() {
    let metaList = document.getElementsByTagName('meta');
    let viewportMeta = Array.from(metaList).filter(meta => {
        return meta.name === 'viewport';
    })[0];
    viewportMeta.content = "width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=yes";
}

removeZoomSupport() {
    let metaList = document.getElementsByTagName('meta');
    let viewportMeta = Array.from(metaList).filter(meta => {
        return meta.name === 'viewport';
    })[0];
    viewportMeta.content = "width=device-width, initial-scale=1.0, maximum-scale=1.0, viewport-fit=cover, user-scalable=no";
}
```

