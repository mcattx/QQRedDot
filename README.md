## 仿QQ红点消除 - vue 版

又名一键退朝

### 效果图

![预览](https://cdn.staticaly.com/gh/mcattx/imgbed@master/ne3jk-omzgm.4vz6ustwwag0.gif)

### 使用方式

1. 复制 [src/components](https://github.com/flashhu/drag-badge/tree/master/components) 文件夹到 对应文件夹内，如 components 文件夹
2. 在页面中使用，如示例：

   ```html
   <badge
    :id="index"
    :name="'cvs' + index"
    :count="item.count"
    @disappear="handleDisappear(index)">
  </badge>
   ```

### API

| 属性  | 说明                                      | 类型    | 默认值 |
| ----- | ----------------------------------------- | ------- | ------ |
| name  | 用于设 Canvas 的 id，注意同一页面不可重复 | String  | —      |
| dot   | 是否设置为红点模式（不显示数字）          | Boolean | false  |
| count | 展示的数字，默认大于99后，显示 99+        | Number  | 0      |

### 事件

| 事件名称      | 说明                       | 返回值 |
| ------------- | -------------------------- | ------ |
| disappear | 当小红点清除时触发（已读） | —      |


#### 过程分析

1. 点击的位置超出初始圆的范围，进入拖拽状态，画布缩放到全屏大小
2. 在最大拖拽范围内，画初始圆，拖动圆，贝塞尔曲线连接带
3. 超出最大拖拽范围，画拖动圆
4. 结束拖拽，画布缩放到初始大小
   1. 在最大拖拽范围外，已读，气泡爆炸效果，自定义触发事件
   2. 在最大拖拽范围内
      1. 如始终未超出范围，回弹动画，恢复初始状态
      2. 如曾超出范围，恢复初始状态


### 实现逻辑参考链接

[史上最详细仿QQ未读消息拖拽粘性效果的实现](https://www.jianshu.com/p/ed2721286778)

### 灵感来源以及致谢

- [drag-badge](https://github.com/flashhu/drag-badge)

