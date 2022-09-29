<template>
  <div class="baiduMapWrap">
    <div v-if="showBar == true" class="headerBar">
      <SelectOptions
        selectItemTitle="所属区域"
        selectItemTitleSize="12px"
        selectionWidth="100px"
        :selectItems="$store.getters.allDict.area_belong_type_opt"
        :initValue="areaIdInitValue"
        :getCheckedValue="getAreaIdInitValue"
      />
      <SelectOptions
        selectItemTitle="停车场"
        selectItemTitleSize="12px"
        selectionWidth="250px"
        :selectItems="$store.getters.allDict.project_select_type_opt"
        :initValue="areaIdInitValue"
        :getCheckedValue="getAreaIdInitValue"
      />
      <el-button
        type="primary"
        icon="el-icon-search"
        size="mini"
        style="margin-left: 10px"
        @click="onSearch"
        >搜索</el-button
      >
      <el-button type="info" icon="el-icon-back" size="mini" @click="onBack"
        >返回</el-button
      >
    </div>

    <baidu-map
      class="bm-view"
      :center="center"
      :zoom="zoom"
      @ready="handler"
      ak="TihvQXOB5ovgGVvhQm0oMOgwIZP4r5Xq"
      :scroll-wheel-zoom="true"
      @zoomend="onZoomend"
    >
      <div v-if="zoom >= 14">
        <div v-for="item in positionPointArray" :key="JSON.stringify(item)">
          <bm-marker
            :position="item.position"
            :dragging="false"
            animation="BMAP_ANIMATION_BOUNCE"
            @dragend="onDragend"
            @click="infoWindowOpen(item)"
            :icon="{
              url:
                item.currentParkingNumber >= 100
                  ? greenIcon
                  : item.currentParkingNumber >= 50 &&
                    item.currentParkingNumber < 100
                  ? orangeIcon
                  : item.currentParkingNumber >= 0 &&
                    item.currentParkingNumber < 50
                  ? redIcon
                  : '',
              size: { width: 40, height: 40 },
              opts: {
                imageSize: { width: 40, height: 40 },
              },
            }"
          >
          </bm-marker>
        </div>
        <bm-info-window
          :position="currentInfo.position"
          :show="show"
          @close="infoWindowClose"
          @open="infoWindowOpen"
          :opts="{
            width: 760,
            height: 260,
            title: '信息窗口标题',
          }"
          ><div class="infoWindow">
            <div class="left">
              <p>{{ currentInfo.carParkName }}</p>
              <p>地址:{{ currentInfo.address }}</p>
              <p>
                车位:{{ currentInfo.currentParkingNumber }}/{{
                  currentInfo.allParkingNumber
                }}
              </p>
            </div>
            <div class="right">
              <img
                @click="onViewImage"
                class="image"
                :src="currentInfo.imgUrl"
              />
            </div></div
        ></bm-info-window>
      </div>

      <bm-navigation anchor="BMAP_ANCHOR_TOP_RIGHT"></bm-navigation>
      <bm-local-search
        :keyword="keyword"
        :auto-viewport="true"
        :pageCapacity="3"
        @searchcomplete="onSearchcomplete"
      ></bm-local-search>
      <bm-geolocation
        anchor="BMAP_ANCHOR_BOTTOM_RIGHT"
        :showAddressBar="true"
        :autoLocation="true"
        @locationSuccess="onlocationSuccess"
      ></bm-geolocation>
    </baidu-map>
    <div v-if="keyword == ''" class="tip">
      <div>左键单击选择定位区域,左键按住不放拖动定位</div>
    </div>
    <el-dialog
      title="车场图片"
      :visible.sync="dialogVisible"
      width="50%"
      :before-close="handleClose"
    >
      <div class="imgContainer">
        <img class="image" :src="currentInfo.imgUrl" />
      </div>
    </el-dialog>
  </div>
</template>

<script>
import BaiduMap from "vue-baidu-map/components/map/Map";
import BmMarker from "vue-baidu-map/components/overlays/Marker";
import BmLabel from "vue-baidu-map/components/overlays/Label";
import BmInfoWindow from "vue-baidu-map/components/overlays/InfoWindow";
import { BmLocalSearch, BmNavigation, BmGeolocation } from "vue-baidu-map";
import position from "@/assets/images/parkingicon.png";
import REDICON from "@/assets/images/red.png";
import ORANGEICON from "@/assets/images/orange.png";
import GREENICON from "@/assets/images/green.png";
import SelectOptions from "@/components/SelectOptions";
import axios from "axios";
export default {
  name: "Map",
  components: {
    BaiduMap,
    BmMarker,
    BmLocalSearch,
    BmNavigation,
    BmGeolocation,
    BmLabel,
    BmInfoWindow,
    SelectOptions,
  },
  props: {
    showBar: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      redIcon: REDICON,
      greenIcon: GREENICON,
      orangeIcon: ORANGEICON,
      show: false,
      dialogVisible: false,
      center: { lng: 104.072745, lat: 30.578994 },
      city: {
        name: "成都市",
      },
      regionList: [
        {
          regionName: "锦江区",
          carParkNum: 18,
        },
        {
          regionName: "青羊区",
          carParkNum: 28,
        },
        {
          regionName: "金牛区",
          carParkNum: 8,
        },
        {
          regionName: "武侯区",
          carParkNum: 32,
        },
        {
          regionName: "成华区",
          carParkNum: 12,
        },
        {
          regionName: "龙泉驿区",
          carParkNum: 33,
        },
        {
          regionName: "青白江区",
          carParkNum: 28,
        },
        {
          regionName: "新都区",
          carParkNum: 19,
        },
        {
          regionName: "温江区",
          carParkNum: 29,
        },
        {
          regionName: "双流区",
          carParkNum: 30,
        },
        {
          regionName: "郫都区",
          carParkNum: 15,
        },
        {
          regionName: "新津区",
          carParkNum: 13,
        },
      ],
      positionPointArray: [
        {
          position: { lng: 104.066575, lat: 30.649507 },
          allParkingNumber: "100",
          currentParkingNumber: "10",
          address:
            "桂溪街道78号桂溪街道78号桂溪街道78号桂溪街道78号桂溪街道78号桂溪街道78号",
          carParkName: "华西医院停车场",
          imgUrl:
            "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fblog.oneapm.com%2Fcontent%2Fimages%2F2016%2F01%2F-----2016-01-06---11-00-23.png&refer=http%3A%2F%2Fblog.oneapm.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1663819837&t=d7e930a6abc18866a0ba89dfbf9bb126",
        },
        {
          position: { lng: 104.047184, lat: 30.671497 },
          allParkingNumber: "110",
          currentParkingNumber: "80",
          address: "桂溪街道28号",
          carParkName: "四川省人民医院停车场",
          imgUrl:
            "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fblog.oneapm.com%2Fcontent%2Fimages%2F2016%2F01%2F-----2016-01-06---11-00-23.png&refer=http%3A%2F%2Fblog.oneapm.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1663819837&t=d7e930a6abc18866a0ba89dfbf9bb126",
        },
        {
          position: { lng: 104.048812, lat: 30.665174 },
          allParkingNumber: "150",
          currentParkingNumber: "10",
          address: "桂溪街道8号",
          carParkName: "四川省骨科医院停车场",
          imgUrl:
            "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fblog.oneapm.com%2Fcontent%2Fimages%2F2016%2F01%2F-----2016-01-06---11-00-23.png&refer=http%3A%2F%2Fblog.oneapm.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1663819837&t=d7e930a6abc18866a0ba89dfbf9bb126",
        },
        {
          position: { lng: 104.076618, lat: 30.70513 },
          allParkingNumber: "78",
          currentParkingNumber: "30",
          address: "桂溪街道7号",
          carParkName: "成都大学附属医院停车场",
          imgUrl:
            "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fblog.oneapm.com%2Fcontent%2Fimages%2F2016%2F01%2F-----2016-01-06---11-00-23.png&refer=http%3A%2F%2Fblog.oneapm.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1663819837&t=d7e930a6abc18866a0ba89dfbf9bb126",
        },
        {
          position: { lng: 104.058242, lat: 30.59617 },
          allParkingNumber: "40",
          currentParkingNumber: "30",
          address: "桂溪街道12号",
          carParkName: "成都市第一人民医院停车场",
          imgUrl:
            "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fblog.oneapm.com%2Fcontent%2Fimages%2F2016%2F01%2F-----2016-01-06---11-00-23.png&refer=http%3A%2F%2Fblog.oneapm.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1663819837&t=d7e930a6abc18866a0ba89dfbf9bb126",
        },
        {
          position: { lng: 104.076618, lat: 30.70513 },
          allParkingNumber: "120",
          currentParkingNumber: "112",
          address: "桂溪街道23号",
          carParkName: "成都大学附属医院停车场",
          imgUrl:
            "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fblog.oneapm.com%2Fcontent%2Fimages%2F2016%2F01%2F-----2016-01-06---11-00-23.png&refer=http%3A%2F%2Fblog.oneapm.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1663819837&t=d7e930a6abc18866a0ba89dfbf9bb126",
        },
        {
          position: { lng: 104.125172, lat: 30.751672 },
          allParkingNumber: "120",
          currentParkingNumber: "112",
          address: "桂溪街道98号",
          carParkName: "西部战区总医院停车场",
          imgUrl:
            "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fblog.oneapm.com%2Fcontent%2Fimages%2F2016%2F01%2F-----2016-01-06---11-00-23.png&refer=http%3A%2F%2Fblog.oneapm.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1663819837&t=d7e930a6abc18866a0ba89dfbf9bb126",
        },
      ],
      mapIcon: position,
      zoom: 9,
      keyword: "",
      address: "",
      plyRef: "",
      mapRef: "",
      geocoderRef: "",
      currentAddress: "",
      currentInfo: {},
      areaIdInitValue: "",
      projectIdInitValue: "",
    };
  },
  methods: {
    getAreaIdInitValue(value) {
      this.areaIdInitValue = value;
      console.log(value);
    },
    getProjectIdInitValue(value) {
      this.projectIdInitValue = value;
      console.log(value);
    },
    onViewImage() {
      this.dialogVisible = true;
    },
    handleClose(done) {
      done();
    },
    infoWindowOpen(info) {
      if (info.position) this.currentInfo = info;
      this.show = true;
    },
    infoWindowClose() {
      this.show = false;
    },
    onClick(event) {
      this.center = event.point;
    },
    onDragend(event) {
      this.center = event.point;
      this.zoom = this.mapRef.Na;
    },
    onlocationSuccess(event) {
      this.center = event.point;
    },
    onSearch() {
      this.keyword = this.address;
      this.zoom = this.mapRef.Na;
    },
    onSearchcomplete(results) {
      if (!results) return;
      this.center = results.Kr[0].point;
    },
    onBack() {
      this.$router.back();
    },
    onComfirm() {
      var point = new BMap.Point(this.center.lng, this.center.lat);
      this.geocoderRef.getLocation(point, (results) => {
        this.currentAddress = results.address;
        if (this.currentAddress && this.center) {
          this.$router.push({
            path: "/carParkManage/carParkSet/addCarPark",
            name: "AddCarPark",
            params: { address: this.currentAddress, position: this.center },
          });
        }
      });
    },
    handler({ BMap, map }) {
      this.zoom = 12;
      this.mapRef = map;
      this.center.lng = 104.072745;
      this.center.lat = 30.578994;
      this.mapRef.clearOverlays();
      this.geocoderRef = new BMap.Geocoder();
      let geolocation = new BMap.Geolocation();
      geolocation.getCurrentPosition((event) => {
        console.log("event", event);
      });
    },
    onZoomend() {
      this.zoom = this.mapRef.Na;
      // console.log("当前缩放级别", this.mapRef.Na);
      if (this.mapRef.Na >= 14) this.commonClearLabel();
      if (this.mapRef.Na < 14) {
        this.commonClearLabel();
        this.drawRegionList(12, 60);
        this.initLabelContent();
      }
    },
    drawRegionList(fontSize, labelSize) {
      this.regionList.forEach((regionItem) => {
        this.addRegionLabel(
          this.city.name,
          regionItem.regionName,
          regionItem.carParkNum,
          fontSize,
          labelSize
        );
      });
    },
    async addRegionLabel(
      cityName,
      regionName,
      carParkNum,
      fontSize,
      labelSize
    ) {
      let point = await this.getReigonLocation(cityName, regionName);
      const labelDocument = ` <div class="labelFomatterRef" data-name=${regionName} 
          style='width:65px;
          height: 65px; 
          border-radius: 35px;
          background:#05ae67;
          position:absolute;
          display:flex;
          justify-content:center;
          align-content: center;
          flex-wrap:wrap;'>
              <span style='display:block;width:100%;text-align:center;'>${regionName}</span>
              <span style='display:block;width:100%;text-align:center;font-size:8px;'>${carParkNum}</span>
          </div>`;
      //labelDocument为“label对象”的子DOM元素;
      let label = new BMap.Label(labelDocument, {
        position: new BMap.Point(point.lng, point.lat),
        offset: new BMap.Size(-10, -40),
      });
      label.labelTag = true;
      //接收DOM和定位,并标记这个覆盖物为label,便于后续识别清除;
      label.setStyle({
        color: "white",
        width: `${labelSize}px`,
        height: `${labelSize}px`,
        fontSize: `${fontSize}px`,
        borderRadius: "30px",
        display: "flex",
        justifyContent: "center",
        alignItems: "center",
      });

      label.addEventListener("click", (event) => {
        this.zoom = 14;
        this.center.lng = point.lng;
        this.center.lat = point.lat;
      });
      this.mapRef.addOverlay(label);
      //将label覆盖物追加到mapRef示例上进行渲染;
      //hover事件绑定在了label的子元素labelDocument原生标签身上,而没有直接绑定在label身上,其原因是因为label上仅支持了onmouseover与onmouseout;
      //由于是父子组件结构、此时事件onmouseover进入到子组件时会冒泡到父组件导致多次触发事件,视觉上就是闪烁;
      //此时改用onmouseenter与onmouseleave绑定子元素labelDocument原生标签即可解决该问题;
    },

    initLabelContent() {
      setTimeout(() => {
        const labelFomatterSets =
          document.getElementsByClassName("labelFomatterRef");
        if (labelFomatterSets.length == 0) return;
        const labelFomatterRefs = Array.prototype.slice.call(labelFomatterSets);
        labelFomatterRefs.forEach((item) => {
          item.onmouseenter = () => {
            item.style.background = "rgb(250, 87, 65)";
            let boundaryRef = new BMap.Boundary();
            boundaryRef.get(item.dataset.name, (res) => {
              if (!res.boundaries.length) return;
              this.plyRef = new BMap.Polygon(res.boundaries[0], {
                strokeWeight: 2,
                fillColor: "green",
                strokeColor: "green",
                fillOpacity: 0.07,
                strokeStyle: "solid",
              });
              this.plyRef.polygonTag = true;
              this.mapRef.addOverlay(this.plyRef);
            });
          };
          item.onmouseleave = () => {
            item.style.background = "#05ae67";
            this.commonClearPolygon();
          };
        });
      }, 100);
      //等待labelFomatterSets挂载完毕后、微任务中执行;
    },

    getReigonLocation(cityName, regionName) {
      return axios
        .get("/baiduMapAPI/place/v2/search", {
          params: {
            query: regionName,
            region: cityName,
            output: "json",
            city_limit: true,
            ak: "TihvQXOB5ovgGVvhQm0oMOgwIZP4r5Xq",
          },
        })
        .then((res) => {
          let location = res.data.results[0].location;
          return Promise.resolve(location);
        });
    },
    //此方法跨域cookie泄漏,后续调整为走后端接口;
    commonClearLabel() {
      let allOverlay = this.mapRef.getOverlays();
      allOverlay.map((overlayItem) => {
        if (overlayItem.labelTag) this.mapRef.removeOverlay(overlayItem);
      });
    },

    commonClearPolygon() {
      let allOverlay = this.mapRef.getOverlays();
      allOverlay.map((overlayItem) => {
        if (overlayItem.polygonTag) this.mapRef.removeOverlay(overlayItem);
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.baiduMapWrap {
  width: 100%;
  height: 100%;
  .headerBar {
    width: 100%;
    height: 30px;
    margin-top: 15px;
    margin-bottom: 15px;
    display: flex;
    flex-wrap: nowrap;
    align-items: center;
    justify-content: left;
    .box {
      white-space: nowrap;
    }
  }
  .tip {
    height: 100px;
    display: flex;
    flex-wrap: nowrap;
    justify-content: center;
    align-items: center;
    color: rgb(250, 87, 65);
  }
  .bm-view {
    width: 100%;
    height: 770px;
  }
  .infoWindow {
    /* height: 130px; */
    width: 320px;
    /* background: red; */
    display: flex;
    .left {
      width: 60%;
      /* background: rgb(125, 125, 125); */
      padding: 3px;
    }
    .right {
      width: 40%;
      /* background: rgb(37, 152, 76); */
      padding: 3px;
      display: flex;
      align-items: center;
      justify-content: center;
      .image {
        width: auto;
        height: auto;
        max-width: 100%;
        max-height: 100%;
      }
    }
  }
  .imgContainer {
    height: 400px;
    display: flex;
    align-items: center;
    justify-content: center;
    .image {
      width: auto;
      height: auto;
      max-width: 100%;
      max-height: 100%;
    }
  }
}
</style>
