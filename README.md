# 说明文档
    选题：7（WebGIS网站搭建）

### 工具使用
    Tomcat，Geoserver，Leaflet，GoogleMap，ajax

### 源码
    https://github.com/TianyouCheng/WMS_Service

### 可执行程序
    见源码（wmsmap.html），需部署Tomcat+Geoserver，科学上网以使用谷歌地图

### 附件
    WebGIS网站演示_程天佑.mp4

# 实现说明

### 1) 前端Web页面的地图窗口使用谷歌地图作为底图
> 基于Leaflet的地图瓦片服务初始化
```JavaScript
L.TileLayer.ChinaProvider = L.TileLayer.extend({

    initialize: function(type, options) { // (type, Object)
        var providers = L.TileLayer.ChinaProvider.providers;

        options = options || {}

        var parts = type.split('.');

        var providerName = parts[0];
        var mapName = parts[1];
        var mapType = parts[2];

        var url = providers[providerName][mapName][mapType];
        options.subdomains = providers[providerName].Subdomains;
        options.key = options.key || providers[providerName].key;

        if ('tms' in providers[providerName]) {
            options.tms = providers[providerName]['tms']
        }

        L.TileLayer.prototype.initialize.call(this, url, options);
    }
});
```
> 谷歌通用地图、谷歌影像地图、谷歌注记地图瓦片
```JavaScript
Google: {
        Normal: {
            Map: "//www.google.cn/maps/vt?lyrs=m@189&gl=cn&x={x}&y={y}&z={z}"
        },
        Satellite: {
            Map: "//www.google.cn/maps/vt?lyrs=s@189&gl=cn&x={x}&y={y}&z={z}",
            Annotion: "//www.google.cn/maps/vt?lyrs=y@189&gl=cn&x={x}&y={y}&z={z}"
        },
        Subdomains: []
    },

```
> 地图初始化及图层添加
```JavaScript
 var normalMap = L.tileLayer.chinaProvider("Google.Normal.Map", {
        maxZoom: 18,
        minZoom: 5,
      }),
      satelliteMap = L.tileLayer.chinaProvider("Google.Satellite.Map", {
        maxZoom: 18,
        minZoom: 5,
      }),
      routeMap = L.tileLayer.chinaProvider("Google.Satellite.Annotion", {
        maxZoom: 18,
        minZoom: 5,
      });

var baseLayers = {
      地图: normalMap,
      影像: satelliteMap,
      标注: routeMap,
    };
```

### 2）可以通过页面将空间数据（shapefile矢量+geotiff栅格）上传到服务器上
> 基于Tomcat的Geoserver部署

    略

> Ajax调用Geoserver Rest接口
```JavaScript
// 新建shp工作区
$.ajax({
    url: "http://localhost:8080/geoserver/rest/workspaces",
    dataType: "json",
    contentType: "application/json",
    type: "POST",
    //      async: false,
    //      processData:false,
    data: JSON.stringify({ workspace: { name: "myUploadShp" } }),
    success: function (data, textStatus) {
    //data可能是xmlDoc、jsonObj、html、text等等
    console.log(data);
    this; //调用本次ajax请求时传递的options参数
    },
    error: function (XMLHttpRequest, textStatus, errorThrown) {
    //通常情况下textStatus和errorThrown只有其中一个包含信息
    console.log(textStatus + errorThrown);
    this; //调用本次ajax请求时传递的options参数
    },
    headers: {
    Authorization: "Basic YWRtaW46Z2Vvc2VydmVy",
    },
});
```
### 3）可以在页面上基于谷歌底图叠加显示上传以后的空间数据
> 基于Rest接口的数据发布
```JavaScript
// 上传shp
$("#submitbtshp").on("click", function () {
    // 文件元素
    var file = $("#fileshp")[0].files[0];
    console.log(file);
    // 通过FormData将文件转成二进制数据
    var formData = new FormData();
    // 将文件转二进制
    formData.append("file", file);
    $.ajax({
    url: "http://localhost:8080/geoserver/rest/workspaces/myUploadShp/datastores/shp/file.shp?charset=GB2312",
    dataType: "json",
    contentType: "application/zip",
    processData: false,
    type: "PUT",
    async: false,
    data: formData,
    headers: {
        Authorization: "Basic YWRtaW46Z2Vvc2VydmVy",
        "Content-type": "application/zip",
    },
    success: function (data, textStatus) {
        //data可能是xmlDoc、jsonObj、html、text等等
        console.log(data);
        this; //调用本次ajax请求时传递的options参数
    },
    error: function (XMLHttpRequest, textStatus, errorThrown) {
        //通常情况下textStatus和errorThrown只有其中一个包含信息
        console.log(textStatus + errorThrown);
        this; //调用本次ajax请求时传递的options参数
    },
    });
    var shpname=file.name.substring(0, file.name.indexOf('.'));
    console.log("myUploadShp:"+shpname);
    //   把shp加载到地图上
        var wmsLayer = L.tileLayer.wms(
    "http://localhost:8080/geoserver/myUploadShp/wms",
    {
        layers: "myUploadShp:"+shpname, //需要加载的图层
        format: "image/png", //返回的数据格式
        transparent: true,
    }
    );
        wmsLayer.addTo(map);
});
```
> Shp矢量与Tiff栅格上传的区别
```JavaScript
// Tiff数据存储需使用coveragestores，栅格数据为datastores
$.ajax({
    url: "http://localhost:8080/geoserver/rest/workspaces/myUploadTif/coveragestores/tif/file.geotiff",
    ...
})
// 两种数据均使用wms进行发布，可以通过同一方式叠加地图
```