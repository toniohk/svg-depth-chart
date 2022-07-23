<template>
  <div class="chart-container">
    <svg :width="options.width" :height="options.height" @mousemove="onMouseMove">
      <g class="chart-body">
        <g class="y axis">
          <g v-for="item in yAxis" :key="item.id" :transform="item.transform" stroke-opacity="1">
            <line class="line-horizontal " :x1="options.width" x2="0" y1="0" y2="0" style="stroke: rgb(218, 218, 218);"></line>
            <text :x="10" y="0" dy="3px" fill="#666" font-size="10px" text-anchor="start">{{ item.value }}</text>
          </g>
        </g>
        <g class="x axis">
          <g v-for="item in xAxis" :key="item.id" :transform="item.transform">
            <line class="line-vertical " x1="0" x2="0" :y1="options.height - options.yAxisGridSpace" y2="0" style="stroke: rgb(218, 218, 218);"></line>
            <text x="0" :y="options.height - options.yAxisGridSpace + 5" dy="10px" fill="#666" font-size="10px" text-anchor="middle">{{ item.value }}</text>
          </g>
        </g>
        <g class="dataset-left">
          <rect v-for="item in buyDepth" :key="item.id" :style="{ fill: '#bdd3e6', stroke: '#6b7f91', fillOpacity: '0.5', strokeDasharray: `${item.strokeDashLen} ${item.strokeGapLen} 10000` }" :x="item.x" :y="item.y" :width="item.width" :height="item.height"></rect>
        </g>
        <g class="dataset-right">
          <rect v-for="item in sellDepth" :key="item.id" :style="{ fill: '#cce6bd', stroke: '#526546', fillOpacity: '0.5', strokeDasharray: `${item.strokeDashLen} ${item.strokeGapLen} 10000` }" :x="item.x" :y="item.y" :width="item.width" :height="item.height"></rect>
        </g>
        <g class="pointer">
          <line :transform="'translate(' + pointer.x + ', 0)'" class="line-vertical" x1="0" x2="0" :y1="options.height - options.yAxisGridSpace" y2="0" style="stroke: rgb(0, 0, 0); stroke-dasharray: 3;"></line>
          <text :x="pointer.x" y="0" dy="10px" font-size="10px" fill="#666" text-anchor="middle">{{ pointer.value.price }}</text>
          <line :transform="'translate(0, ' + pointer.y + ')'" class="line-vertical" :x1="options.width" x2="0" y1="0" y2="0" style="stroke: rgb(0, 0, 0); stroke-dasharray: 3;"></line>
          <text :x="options.width - 30" :y="pointer.y" dy="10px" fill="#666" font-size="10px" text-anchor="middle">{{ pointer.value.amount }}</text>
        </g>
      </g>
    </svg>
  </div>
</template>

<script>

import {onBeforeMount, ref} from "vue";

const defaultDayOptions = {
  // basic configuration
  width: 780, // depth map visible width
  height: 540, // depth map visible height
  initOffset: 0.5, // initial offset to solve blurred lines
  bgColor: "rgba(255,255,255,0.1)", // overall background color

  // watermark configuration
  watermaskType: "image", // watermark configuration type "text" (text) "image" (picture)
  watermaskContent: "vue-depth-chart", // text watermark directly assign text image watermark import object

  // line configuration
  gridLineColor: "#ddd", // grid line color
  rulerLineColor: "#999", // ruler line color color

  // axis configuration
  xAxisGridSpace: 90, // x-axis background grid spacing
  yAxisGridSpace: 30, // y-axis background grid spacing
  xAxisFontColor: "#666", // x-axis ruler scale font color
  yAxisFontColor: "#666", // y-axis ruler scale font color
  xAxisFontSize: "12px Aria", // x-axis ruler scale font size
  yAxisFontSize: "12px Aria", // y-axis ruler scale font size
  yAxisShadowColor: "#fff", // y-axis ruler tick shadow color

  // spacing configuration
  bottomSpace: 8, // the distance between the depth map and the y-axis
  buySellSpace: 10, // Space between buy and sell ends

  // prompt box
  tipWidth: 120,
  tipHeight: 60,
  tipLocationLineColor: "#999", // Tip box positioning line color
  tipLocationDotRadius: 5, // Tip box dot radius
  tipType: "mouse", // Mouse movement prompt information position axis (on the xy axis) mouse (displayed with absolute positioning following the mouse position)

  // buy prompt box
  tipBuyLocationDotBgColor: "#fff", // background color of tip box dots
  tipBuyBorderColor: "#67c23a", // The border color of the tip box
  tipBuyBgColor: "#f0f9eb", // background color of tip box
  tipBuyTextColor: "#67c23a", // The color of the tip text in the tip box

  // sell prompt
  tipSellLocationDotBgColor: "#fff", // background color of tip box dots
  tipSellBorderColor: "#f56c6c", // The border color of the prompt box
  tipSellBgColor: "#fef0f0", // background color of the prompt box
  tipSellTextColor: "#f56c6c", // The prompt text color of the prompt box

  // custom depth style configuration
  buyStrokeColor: "rgb(111,168,0)", // buy stroke color rgb(111,168,0)
  sellStrokeColor: "rgb(234,0,112)", // Sell stroke color rgb(234,0,112)
  buyLinearGradientArray: [
    "rgb(141,186,51)",
    "rgb(160,197,87)",
    "rgb(189,215,138)",
    "rgb(228,238,206)",
    "rgb(250,250,241)",
  ],
  sellLinearGradientArray: [
    "rgb(255,148,152)",
    "rgb(255,164,172)",
    "rgb(255,192,197)",
    "rgb(255,228,230)",
    "rgb(255,249,249)",
  ],
};

const defaultNightOptions = {
  // basic configuration
  width: 780, // depth map visible width
  height: 540, // depth map visible height
  initOffset: 0.5, // initial offset to solve blurred lines
  bgColor: "rgba(0,0,0,0.9)", // overall background color

  // watermark configuration
  watermaskType: "image", // watermark configuration type "text" (text) "image" (picture)
  watermaskContent: "vue-depth-chart", // text watermark directly assign text image watermark import object

  // line configuration
  gridLineColor: "#333", // grid line color
  rulerLineColor: "rgb(47,47,47)", // ruler line color color

  // axis configuration 0
  xAxisGridSpace: 90, // x-axis background grid spacing
  yAxisGridSpace: 30, // y-axis background grid spacing
  xAxisFontColor: "#999", // x-axis ruler scale font color
  yAxisFontColor: "#999", // y-axis ruler scale font color
  xAxisFontSize: "12px Aria", // x-axis ruler scale font size
  yAxisFontSize: "12px Aria", // y-axis ruler scale font size
  yAxisShadowColor: "#333", // y-axis ruler tick shadow color

  // spacing configuration
  bottomSpace: 8, // the distance between the depth map and the y-axis
  buySellSpace: 10, // Space between buy and sell ends

  // prompt box
  tipWidth: 120,
  tipHeight: 60,
  tipLocationLineColor: "#999", // Tip box positioning line color
  tipLocationDotRadius: 5, // Tip box dot radius
  tipType: "mouse", // Mouse movement prompt information position axis (on the xy axis) mouse (displayed with absolute positioning following the mouse position)

  // buy prompt box
  tipBuyLocationDotBgColor: "#fff", // background color of tip box dots
  tipBuyBorderColor: "#67c23a", // The border color of the tip box
  tipBuyBgColor: "#f0f9eb", // background color of tip box
  tipBuyTextColor: "#67c23a", // The color of the tip text in the tip box

  // sell prompt
  tipSellLocationDotBgColor: "#fff", // background color of tip box dots
  tipSellBorderColor: "#f56c6c", // The border color of the prompt box
  tipSellBgColor: "#fef0f0", // background color of the prompt box
  tipSellTextColor: "#f56c6c", // The prompt text color of the prompt box

  // custom depth style configuration
  buyStrokeColor: "rgb(111,168,0)", // buy stroke color rgb(111,168,0)
  sellStrokeColor: "rgb(234,0,112)", // Sell stroke color rgb(234,0,112)
  buyLinearGradientArray: [
    "rgb(80,120,3)",
    "rgb(72,106,4)",
    "rgb(51,74,5)",
    "rgb(35,49,7)",
    "rgb(20,25,8)",
  ],
  sellLinearGradientArray: [
    "rgb(217,45,127)",
    "rgb(172,37,101)",
    "rgb(115,27,68)",
    "rgb(64,19,39)",
    "rgb(26,13,18)",
  ],
};

export default {
  name: 'SvgDepthChart',
  props: {
    data: {
      type: [Object, Array],
      default() {
        return {};
      },
    },
    customizeOptions: {
      type: [Object],
      default() {
        return defaultDayOptions;
      },
    },
    theme: {
      type: String,
      default() {
        return "day";
      },
    },
  },
  setup(props) {
    let options = Object.assign(defaultDayOptions, props.customizeOptions);
    let yAxis = [];
    let xAxis = [];
    let buyDepth = [];
    let sellDepth = [];
    let pointList = [];
    const pointer = ref({
      x: 0,
      y: 0,
      value: {
        amount: '',
        price: ''
      }
    });

    onBeforeMount(() => {
      if (props.theme === "night") {
        options = Object.assign(defaultNightOptions, props.customizeOptions);
      }
      init();
    })

    const init = () => {
      drawGrid(props.data, options);
      drawDepth(props.data, options);
    }

    const deepClone = (obj) => {
      if (obj instanceof RegExp) return new RegExp(obj);
      if (obj instanceof Date) return new Date(obj);
      if (obj === null || typeof obj !== "object") return obj; //If it is not a reference type, return directly
      let newObj = new obj.constructor(); //if obj is an array, newObj=[]; if obj is an object, newObj={}
      for (let key in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, key)) {
          //Is it its own object
          newObj[key] = deepClone(obj[key]); //assignment
        }
      }
      return newObj;
    }

    const drawGrid = (data, options) => {
      const {
        width,
        height,
        initOffset,
        bottomSpace,
        buySellSpace,
        xAxisGridSpace,
        yAxisGridSpace,
      } = options;
      const yLen = height - yAxisGridSpace;
      const origin = [0, yLen];
      const allData = deepClone(data)
      const buyOriginData = allData["buy"];
      const buyData = buyOriginData.reverse();
      const sellData = data["sell"];
      const buyLen = buyData.length;
      const sellLen = sellData.length;
      const buyBasePrice = buyData[buyLen - 1].price;
      const sellBasePrice = sellData[sellLen - 1].price;
      const buyPriceDiff = buyData[0].price - buyData[buyLen - 1].price;
      const sellPriceDiff = sellData[0].price - sellData[sellLen - 1].price;
      const staticWidth = (width - buySellSpace) / 2; // single width
      const staticHeight = yLen - bottomSpace;
      const buyTotalAmount = buyData[0].amount; // Buy order full depth amount
      const sellTotalAmount = sellData[0].amount; // total depth of sell order
      const totalAmount = Math.max(buyTotalAmount, sellTotalAmount); // Determine the quantity corresponding to the maximum height

      // Create vertical grid line path
      for (
          let i = initOffset + xAxisGridSpace;
          i < width;
          i += xAxisGridSpace
      ) {
        xAxis.push({
          transform: `translate(${i.toString()}, 0)`,
          value: ''
        });
      }

      let index = xAxis.length - 1;
      for (let i = origin[0] + xAxisGridSpace; i < width; i += xAxisGridSpace) {
        let number = 0;
        if (i <= staticWidth) {
          number = parseFloat(
              (i / staticWidth) * buyPriceDiff + buyBasePrice
          ).toFixed(2);
        } else if (i > staticWidth + buySellSpace) {
          number = parseFloat(
              ((i - (staticWidth + buySellSpace)) / staticWidth) * sellPriceDiff +
              sellBasePrice
          ).toFixed(2);
        } else {
          number = null;
        }
        xAxis[index].value = number;
        index --;
      }
      xAxis.unshift({
        transform: `translate(0, 0)`,
        value: ''
      });

      // Create a horizontal grid line path
      for (
          let j = initOffset + yAxisGridSpace;
          j < height;
          j += yAxisGridSpace
      ) {
        yAxis.push({
          transform: `translate(0, ${(j - bottomSpace).toString()})`,
          value: ''
        });
      }

      index = yAxis.length - 2;
      for (
          let j = origin[1] - yAxisGridSpace;
          j > origin[1] - yLen;
          j -= yAxisGridSpace
      ) {
        yAxis[index].value = parseFloat(
            ((staticHeight - j) / staticHeight) * totalAmount
        ).toFixed(2);
        index --;
      }
    }

    const drawDepth = (data, options) => {
      const {
        width,
        height,
        bottomSpace,
        buySellSpace,
        yAxisGridSpace,
      } = options;
      const allwidth = width; // overall width
      const allheight = height - yAxisGridSpace;
      const staticWidth = (allwidth - buySellSpace) / 2; // fixed single width
      const staticHeight = allheight - bottomSpace;
      const allData = deepClone(data)
      const buyOriginData = allData["buy"];
      const buyData = buyOriginData.reverse();
      const sellData = data["sell"];
      const buyLen = buyData.length;
      const sellLen = sellData.length;

      const buySidePrice = buyData[0].price; // buy edge price
      const sellSidePrice = sellData[0].price; // buy edge price
      const buyTotalAmount = buyData[0].amount; // Buy order full depth amount
      const sellTotalAmount = sellData[0].amount; // total depth of sell order
      const totalAmount = Math.max(buyTotalAmount, sellTotalAmount); // Determine the amount corresponding to the maximum height
      const buyMaxPriceDiff = buyData[buyLen - 1].price - buyData[0].price; // maximum price difference for buy orders
      const sellMaxPriceDiff = sellData[0].price - sellData[sellLen - 1].price; // maximum price difference for sell order

      // Add a buy order drawing point and make sure that the coordinate points are all integers
      buyData.forEach((item) => {
        const x = Math.round(
            ((item.price - buySidePrice) / buyMaxPriceDiff) * staticWidth
        );
        const y = Math. round(
            ((totalAmount - item.amount) / totalAmount) * staticHeight
        );
        pointList.push({ x, y, side: "buy", value: item });
      });

      // Add sell order drawing points and make sure the coordinate points are all integers
      sellData.forEach((item) => {
        const x = Math.round(
            width -
            ((sellSidePrice - item.price) / sellMaxPriceDiff) * staticWidth
        );
        const y = Math. round(
            ((totalAmount - item.amount) / totalAmount) * staticHeight
        );
        pointList.push({ x, y, side: "sell", value: item });
      });

      pointList = pointList.sort((a, b) => {
        return a.x - b.x;
      });

      pointList.forEach((item, index) => {
        const x = item.x;
        const y = item.y;
        const cellHeight = staticHeight - item.y;
        const xAfter = pointList[index + 1] ? pointList[index + 1].x : x;
        const yAfter = pointList[index + 1] ? pointList[index + 1].y : y;
        const yBefore = pointList[index - 1] ? pointList[index - 1].y : y;
        const cellWidth = xAfter - x;
        const strokeDashLen = cellWidth + (yAfter > y ? yAfter - y : 0);
        const strokeGapLen = (cellWidth + cellHeight) * 2 - strokeDashLen - (yBefore > y ? yBefore - y : 0);
        if (item.side === "sell") {
          sellDepth.push({
            x,
            y,
            width: cellWidth,
            height: cellHeight,
            strokeDashLen,
            strokeGapLen
          });
        } else {
          buyDepth.push({
            x,
            y,
            width: cellWidth,
            height: cellHeight,
            strokeDashLen,
            strokeGapLen
          });
        }

      });
    }

    const onMouseMove = ({offsetX}) => {
      pointList.forEach((item) => {
        if (item.x < offsetX) {
          pointer.value = { ...item };
        }
      });
    }

    return {
      options,
      xAxis,
      yAxis,
      buyDepth,
      sellDepth,
      pointList,
      pointer,
      onMouseMove
    };
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
