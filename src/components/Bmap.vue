

<template>
  <div id="container"></div>
</template>
  <script setup>
import { onMounted, reactive, ref } from "vue";
import markers from "/markers.png";
import closeIcon from "/redFood.png";
import yiqingp from "/yiqingp.png";

const state = reactive({
  InfoWindow: null,
  marks: [
    {
      pos: {
        lnt: 116.404,
        lat: 39.915
      },
      data: [
        {
          key: "张三",
          value: 25
        },
        {
          key: "李四",
          value: 30
        },
        {
          key: "王五",
          value: 33
        },
        {
          key: "赵六",
          value: 43
        }
      ]
    },
    {
      pos: {
        lnt: 116.404,
        lat: 39.900
      },
      data: [
        {
          key: "慕容复",
          value: 25
        },
        {
          key: "乔峰",
          value: 25
        },
        {
          key: "段誉",
          value: 25
        }
      ]
    }
  ]
});

const setCustomControl = () => {
  function ZoomControl() {
    // 设置默认停靠位置和偏移量
    this.defaultAnchor = BMAP_ANCHOR_TOP_LEFT;
    this.defaultOffset = new BMap.Size(10, 10);
  }
  // 通过JavaScript的prototype属性继承于BMap.Control
  ZoomControl.prototype = new BMap.Control();
  ZoomControl.prototype.initialize = function(map) {
    // 创建一个DOM元素
    var div = document.createElement("div");
    // 添加文字说明
    div.appendChild(document.createTextNode("放大2级"));
    // 设置样式
    div.style.cursor = "pointer";
    div.style.border = "1px solid gray";
    div.style.backgroundColor = "white";
    // 绑定事件，点击一次放大两级
    div.onclick = function(e) {
      map.zoomTo(map.getZoom() + 2);
    };
    const mapContainer = map.getContainer();
    // 放进地图容器中
    mapContainer.appendChild(div);
    // 并返回空间DOM
    return div;
  };
  return ZoomControl;
};

const addMarker = (point, map, data) => {
  console.log(this, 'addMarker')
  // 创建图标对象
  var myIcon = new BMap.Icon("/markers.png", new BMap.Size(23, 25), {
    // 指定定位位置。
    // 当标注显示在地图上时，其所指向的地理位置距离图标左上
    // 角各偏移10像素和25像素。您可以看到在本例中该位置即是
    // 图标中央下端的尖角位置。
    anchor: new BMap.Size(10, 25),

    imageSize: new BMap.Size(23, 25)
    // 设置图片偏移。
    // 当您需要从一幅较大的图片中截取某部分作为标注图标时，您
    // 需要指定大图的偏移位置，此做法与css sprites技术类似。
    // imageOffset: new BMap.Size(0, 0 - index * 25)   // 设置图片偏移
  });

  var marker = new BMap.Marker(point, { icon: myIcon, title: "我是title" });
  // 百度地图启动拖动，  // 启动 会导致百度地图弹窗点击两次才生效，因为点击第一次的时候会触发拖动效果
  // marker.enableDragging();
  // marker.addEventListener("dragend", function(e) {
  // });
  marker.addEventListener("click", function(e) {
    console.log(e, this, "marker-click");
    if (state.InfoWindow) {
      state.InfoWindow.close();
    }
    openInfoWindow(point, map, data);
  });
  map.addOverlay(marker);
};

// 自定义信息窗体显示内容
const getInfoBoxContent = data => {
  var headStr = "我是头部";
  var bodyStr = "";
  for (var i = 0; i < data.length; i++) {
    bodyStr +=
      '<div class="div-factor">' +
      data[i].key +
      "：" +
      data[i].value +
      "</div>";
  }
  var footStr =
    '<div class="div-btn"><a onclick="onInfoWindowClick()">查看详情></a></div>';
  var html = `<div class="infoBoxContent">${headStr}${bodyStr}${footStr}</div>`;
  return html;
};

// 自定义信息窗口
const openInfoWindow = (point, map, data) => {
  const openInfoHtml = getInfoBoxContent(data);
  state.InfoWindow = new BMapLib.InfoBox(map, openInfoHtml, {
    boxStyle: {
      backgroundImage: "url('/yiqingp.png')",
      backgroundSize: "100% 100%",
      width: "300px",
      height: "250px"
    },
    closeIconMargin: "8px 8px 0 0",
    enableAutoPan: false,
    alignBottom: false,
    closeIconUrl: closeIcon
  });
  state.InfoWindow.open(point);
};

const addPolyline = map => {
  var polyline = new BMap.Polyline(
    [new BMap.Point(116.399, 39.91), new BMap.Point(116.405, 39.92)],
    { strokeColor: "blue", strokeWeight: 6, strokeOpacity: 0.5 }
  );
  console.log(polyline);

  map.addOverlay(polyline);
};

// 添加自定义覆盖物
const addCustomOverlay = () => {
  // 定义自定义覆盖物的构造函数
  function SquareOverlay(center, length, color) {
    this._center = center;
    this._length = length;
    this._color = color;
  }
  // 继承API的BMap.Overlay
  SquareOverlay.prototype = new BMap.Overlay();
  // 实现初始化方法
  SquareOverlay.prototype.initialize = function(map) {
    // 保存map对象实例
    this._map = map;
    // 创建div元素，作为自定义覆盖物的容器
    var div = document.createElement("div");
    div.style.position = "absolute";
    // 可以根据参数设置元素外观
    div.style.width = this._length + "px";
    div.style.height = this._length + "px";
    div.style.background = this._color;
    // 将div添加到覆盖物容器中
    map.getPanes().markerPane.appendChild(div);
    // 保存div实例
    this._div = div;
  };

  // 实现绘制方法
  SquareOverlay.prototype.draw = function() {
    // 根据地理坐标转换为像素坐标，并设置给容器
    var position = this._map.pointToOverlayPixel(this._center);

    this._div.style.left = position.x - this._length / 2 + "px";
    this._div.style.top = position.y - this._length / 2 + "px";
  };

  // 实现显示方法
  SquareOverlay.prototype.show = function() {
    if (this._div) {
      this._div.style.display = "";
    }
  };
  // 实现隐藏方法
  SquareOverlay.prototype.hide = function() {
    if (this._div) {
      this._div.style.display = "none";
    }
  };

  // 添加自定义方法
  SquareOverlay.prototype.toggle = function() {
    if (this._div) {
      if (this._div.style.display == "") {
        this.hide();
      } else {
        this.show();
      }
    }
  };
  return SquareOverlay;
};

onMounted(() => {
  const map = new BMap.Map("container");
  const pointCenter = new BMap.Point(116.404, 39.915);
  map.centerAndZoom(pointCenter, 15);
  console.log(state.marks, 'marks')
  state.marks.forEach(item => {
    const point = new BMap.Point(item.pos.lnt, item.pos.lat);
    // 创建标注
    addMarker(point, map, item.data);
  });

  // console.log(BMapLib, "BMapLib");
  // 添加折线
  // addPolyline(map);

  // 自定义覆盖物
  // const customOverlay = addCustomOverlay();

  // const mySquare = new customOverlay(map.getCenter(), 100, "red");

  // map.addOverlay(mySquare);

  //   // 添加自定义覆盖物
  // var mySquare = new SquareOverlay(map.getCenter(), 100, "red");
  // map.addOverlay(mySquare);

  // map.addControl(new BMap.NavigationControl());

  // const myZoomCtrl = setCustomControl();
  // const myZoomCtrl2 = new myZoomCtrl();
  // console.log(myZoomCtrl2, "zn-myZoomCtrl2");
  // // 添加到地图当中
  // map.addControl(myZoomCtrl2, map);
});
</script>
  <style scoped lang="scss">
#container {
  width: 100%;
  height: 100%;
}
</style>
  