<template>
  <div
    class="badge-container"
    :class="{
      selected: isDrag,
    }"
    @mousedown="handleDown"
    @mousemove="handleMove"
  >
    <div v-if="count !== 0" class="canvas-box">
      <canvas
        class="badge-cvs"
        :class="{
          selected: isDrag,
        }"
        type="2d"
        :id="name"
        @mouseup="handleMoveEnd"
      ></canvas>
    </div>
  </div>
</template>

<script>
import {
  setCvsSize,
  drawBadge,
  drawInRange,
  getRAfterMove,
  badgeRebound,
  badgeExplode,
} from "./draw.js";

// 最大拖拽距离
const MAX_DRAG_DISTANCE = 60;
// 封顶数字
const MAX_COUNT = 99;
// 固定圆初始值：x, y, r 同一值
const DEF_CIRCLE = 10;
// 固定圆初始值（红点模式）：x, y, r 同一值
const DEF_CIRCLE_DOT = 8;
// 初始大小
const INITIALSIZE = 20

export default {
  name: "BadgeC",
  props: {
    dot: {
      type: Boolean,
      value: false,
    },
    // 消息数量
    count: {
      type: Number,
      value: 0,
    },
    name: String,
  },
  data() {
    return {
      isStart: false,
      // 拖拽状态
      isDrag: false,
      // 拖拽距离是否已过大
      isOutOfRange: false,
      // 固定圆{x, y}
      fixCircle: null,
      // 两圆点圆心距
      distance: 0,
      // 两圆夹角（touch - fix）
      angle: 0,
      // canvas 实例
      cvsInstance: null,
    };
  },
  methods: {
    // 获取canvas实例
    getCvsAndCtx: function (selector) {
      return new Promise((resolve, reject) => {
        const query = document.querySelector(`#${selector}`);
        if (query) {
          const cvs = query;
          const ctx = cvs.getContext("2d");
          cvs.width = INITIALSIZE
          cvs.height = INITIALSIZE
          const initialSize = { h: query.height, w: query.width };
          // 用于计算初始点的位置 防止机型不同导致取数异常
          const initialPos = { left: cvs.offsetLeft, top: cvs.offsetTop };
          const cvsInstance = { ctx, cvs, initialSize, initialPos };
          setCvsSize(cvsInstance, query.height, query.width);
          resolve(cvsInstance);
        } else {
          const error = new Error(`Can not get canvas ${selector}`);
          reject(error);
        }
      });
    },
    handleDown() {
      this.isStart = true
      const w = document.body.clientWidth;
      const h = document.body.clientHeight;
      setCvsSize(this.cvsInstance, h, w);
      this.isDrag = true
    },
    // 鼠标拖动，状态/数据保存
    handleMove: function (event) {
      if (!this.isStart) {
        return
      }
      const defCircle = this.dot ? DEF_CIRCLE_DOT : DEF_CIRCLE;
      const touchCircle = {
        // x: event.touches[0].pageX,
        // y: event.touches[0].pageY,
        x: event.pageX,
        y: event.pageY,
      };
      let fixCircle = this.fixCircle;
      if (!this.fixCircle) {
        fixCircle = {
          x: this.cvsInstance.initialPos.left + defCircle,
          y: this.cvsInstance.initialPos.top + defCircle,
        };
        this.fixCircle = fixCircle
      }
      const distance = Math.sqrt(
        (touchCircle.x - fixCircle.x) ** 2 + (touchCircle.y - fixCircle.y) ** 2
      );
      // 斜率推得角度
      const rate =
        (touchCircle.x - fixCircle.x) / (touchCircle.y - fixCircle.y);
      const angle = Math.atan(rate);

      // 已处在拖拽状态
      if (this.isDrag) {
        this.handleDraw(distance, defCircle, touchCircle, fixCircle);
      }

      // 首次拖拽 canvas尺寸放大 / 判断状态
      // if (!this.isDrag && distance > defCircle) {
      //   const w = document.body.clientWidth;
      //   const h = document.body.clientHeight;
      //   setCvsSize(this.cvsInstance, h, w);
      //   this.isDrag = true
      // }
      if (!this.isOutOfRange && distance > MAX_DRAG_DISTANCE) {
        this.isOutOfRange = true
      }
      this.distance = distance
      this.angle = angle
    },
    // 拖动时绘图
    handleDraw: function (distance, defCircle, touchCircle, fixCircle) {
      const r = getRAfterMove(distance, MAX_DRAG_DISTANCE, defCircle);
      const value = this.count > MAX_COUNT ? `${MAX_COUNT}+` : this.count;
      // 连接带
      if (!this.isOutOfRange && distance < MAX_DRAG_DISTANCE) {
        drawInRange(this.cvsInstance, fixCircle, touchCircle, r);
      } else {
        this.cvsInstance.ctx.clearRect(
          0,
          0,
          this.cvsInstance.cvs.width,
          this.cvsInstance.cvs.height
        );
      }
      // 移动圆
      drawBadge(
        this.cvsInstance.ctx,
        this.dot,
        { x: touchCircle.x, y: touchCircle.y, r: r.touchR },
        value
      );
    },
    // 拖拽停止
    handleMoveEnd: function (event) {
      // 真机点击也会触发该事件
      if (!this.isDrag) return;
      const distance = this.distance;
      const size = this.cvsInstance.initialSize;
      const ctx = this.cvsInstance.ctx;
      const value = this.count > MAX_COUNT ? `${MAX_COUNT}+` : this.count;
      const defCircle = this.dot ? DEF_CIRCLE_DOT : DEF_CIRCLE;
      const touchCircle = {
        x: event.pageX,
        y: event.pageY,
      };
      ctx.clearRect(
        0,
        0,
        this.cvsInstance.cvs.width,
        this.cvsInstance.cvs.height
      );

      if (distance >= MAX_DRAG_DISTANCE && this.isOutOfRange) {
        // 已读 => 爆炸
        badgeExplode(this.cvsInstance, touchCircle);
        // 恢复初始化值 缩放画布
        setTimeout(() => {
          this.initAfterMove(size, distance, defCircle, value);
          this.isDrag = false
        }, 200);
      } else if (distance < MAX_DRAG_DISTANCE && !this.isOutOfRange) {
        // 未读 => 回弹
        badgeRebound(
          this.cvsInstance,
          this.angle,
          this.dot,
          touchCircle,
          this.fixCircle,
          defCircle,
          distance,
          value
        );
        // 恢复初始化值 缩放画布
        setTimeout(() => {
          this.initAfterMove(size, distance, defCircle, value);
          this.isDrag = false
        }, 30);
      } else {
        this.initAfterMove(size, distance, defCircle, value);
        this.isDrag = false
      }
      this.isStart = false
    },
    resetData(distance, defCircle, value) {
      this.isDrag = false;
      this.isOutOfRange = false;
      this.fixCircle = null;
      this.distance = 0;
      this.angle = 0;
      if (distance < MAX_DRAG_DISTANCE) {
        // 防止 css 修改未生效 导致红点显示但宽高比例有误
        setTimeout(() => {
          if (!this.isDrag) {
            drawBadge(this.cvsInstance.ctx, this.dot, defCircle, value);
          }
        }, 20);
      } else {
        this.$emit("disappear", {});
      }
    },
    initAfterMove: function (size, distance, defCircle, value) {
      setCvsSize(this.cvsInstance, size.h, size.w);
      this.resetData(distance, defCircle, value);
    },
  },
  async mounted() {
    if (this.count) {
      const r = await this.getCvsAndCtx(this.name);
      if (r) {
        this.cvsInstance = r
        const defCircle = this.dot ? DEF_CIRCLE_DOT : DEF_CIRCLE;
        const value = this.count > MAX_COUNT ? `${MAX_COUNT}+` : this.count;
        drawBadge(r.ctx, this.dot, defCircle, value);
      }
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.badge-container {
  width: 20px;
  height: 20px;
}

.badge-container.selected {
  position: fixed;
  top: 0;
  right: 0;
  width: 100vw;
  height: 100vh;
}

.canvas-box {
  height: 100%;
}

.badge-cvs {
  height: 100%;
  width: 100%;
}

.badge-cvs.selected {
  width: 100vw;
}
</style>
