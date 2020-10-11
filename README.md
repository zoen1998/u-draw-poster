# uni-draw-poster 海报绘制插件

创建绘制海报矩形方法。用于小程序和uniapp快速构建海报，内置了图片绘制，圆角矩形绘制，换行字体绘制等方法。
注意，由于`canvas-2d`仅小程序支持。如需在多端中使用请参考 [旧API的兼容](#h2--api-)

#### 安装
~~~
npm i uni-draw-poster -S
~~~

#### 1. 创建生成海报绘制工具

~~~html
<canvas id="canvas" style="width:100rpx;height:100rpx" type="2d" />
~~~

~~~js
import DrawPoster from 'uni-draw-poster'
// 传入选择器, 初始化绘制工具
const drawPoster = await DrawPoster.build("#canvas")
~~~

#### 2. 设置画布尺寸

~~~js
// 获取rpx单位
const rpx = drawPoster.rpx
// 设置长高为100rpx的矩形宽高
drawPoster.node.width = 100*rpx
drawPoster.node.height = 100*rpx
~~~

#### 3. 绘制任意内容

`drawPoster.draw(async callback(ctx))`

~~~js
drawPoster.draw(async (ctx) => {
    // 绘制背景颜色
    ctx.fillStyle = "#F4F4F4";
    ctx.fillRect(0, 0, drawPoster.node.width, drawPoster.node.height);
    // 绘制字体
    ctx.textBaseline = "top";
    ctx.textAlign = "start";
    ctx.fillStyle = "white";
    ctx.font = `bold ${22}px sans-serif`;
    ctx.fillText('周先生', drawPoster.node.width/2, 38.5);
})
~~~

#### 4. 进行绘制

需要注意的是，`drawPoster.draw`并不会马上绘制，只是将该任务添加到了栈堆，需要使用`drawPoster.awaitCreate`函数进行绘制。

~~~js
// 由于每个任务都有可能会有异步的绘制任务, 所以得需要使用await等待绘制
const result = await drawPoster.awaitCreate();
// 绘制成功将返回每个任务的绘制状况组成的数组
console.log("draw绘制状况:", result); // draw绘制状况: [true]
~~~

[^为什么这么做]: 当全部同步绘制时，将会出现绘制时间保持不一致的情况。这样就会导致一个问题，绘制图层覆盖导致显示未达到预期效果，所以这么设计是有必要的。

#### 5. 生成图片本地地址

在生产开发中，海报往往需要有保存功能。如需要用到保存功能时，可以使用`drawPoster.createImgUrl` 进行创建图片地址，在由微信`api`进行保存。
~~~js
// 等待创建地址
const posterImgUrl = await drawPoster.createImagePath();
console.log("绘制生成本地地址:", posterImgUrl);
~~~
你也可以不使用`drawPoster.awaitCreate`方法，当调用`drawPoster.createImgUrl`时会自动检测任务列表，如果有则执行绘制任务后在创建地址。

如果使用的是`type=2d`的方式，`createImgUrl` 会根据 `node.width` 与 `node.height` 进行创建图片。如果需要使用[旧API](#h2--api-)，或自定义参数，`awaitCreate` 方法可以接受一个配置对象。

~~~js
drawPoster.createImagePath({
	x?: number;
	y?: number;
	width?: number;
	height?: number;
	destWidth?: number;
	destHeight?: number;
})
~~~

#### 使用解析赋值更加便捷的使用

~~~js
const { node, draw, awaitCreate, createImgUrl } = await DrawPoster.build("#canvas");
// 设置尺寸
node.width = 100;
node.height = 100;
// 进行绘制
draw(async ctx=>{
    // ...
})
draw(async ctx=>{
    // ...
})
const result = await awaitCreate();
console.log("draw绘制状况:", result); // draw绘制状况:[true, true]
// 等待创建地址
const posterImgUrl = await createImagePath();
console.log("绘制生成本地地址:", posterImgUrl); // 制生成本地地址:tmp.....
~~~

## ctx 扩展方法

`drawPoster`在创建时，会自动的向`ctx(画笔)`添加扩展方法，以便更加方便的构建海报矩形。

#### 绘制图片

`drawPoster`绘制图片与原生小程序绘制不相同，`ctx.loadDrawImage`内部已经内置了`uini.downloadFile`，只需要传入网络地址即可

~~~js
drawPoster.draw(async (ctx)=>{
    const headImgUrl = "https://pshangcheng.wsandos.com/pic/15946033604872"
 /** ctx的等待绘制图片方法
   * @param  {string} url 网络图片地址(必须)
   * @param  {number} x 绘制x轴位置(必须)
   * @param  {number} y 绘制y轴位置(必须)
   * @param  {number} w 绘制图片宽度(必须)
   * @param  {number} h 绘制图片高度(必须)
   * @returns {Promise} 图片绘制成功时返回true, 需ctx.restore()绘制出实体
   */
    await ctx.drawLoadImage(
      headImgUrl,
      88,
      174.94,
      198.98, 
      36
    );
})
~~~
注意: 需要添加域名才能绘制成功！

#### 绘制换行字体

~~~js
drawPoster.draw(async (ctx)=>{
 /** ctx的绘制换行字体方法
   * @param  {string} text 本地图片地址(必须)
   * @param  {number} maxWidth 绘制换行字体的最大宽度(必须)
   * @param  {number} fontHeight 字体高度(必须)
   * @param  {number} layer 绘制层数(必须)
   * @param  {number} x 绘制x轴位置(必须)
   * @param  {number} y 绘制y轴位置(必须)
   * @returns {null} 无返回值, 需ctx.restore()绘制出实体
   */
   ctx.fillWarpText({
      text: `您好，我是李先生，我负责xxx合作，如果您在xxx商务合作需求，请直接咨询我。`,
      maxWidth: 300.97,
      fontHeight: 20,
      layer: 3,
      x: 37,
      y: 241.21
    });
})
~~~

#### 绘制圆角矩形

~~~js
drawPoster.draw(async (ctx)=>{
 /** ctx.roundRect(x, y, w, h, r)
   * @param {number} x x坐标轴(必须)
   * @param {number} y y坐标轴(必须)
   * @param {number} w 宽度(必须)
   * @param {number} h 高度(必须)
   * @param {number} r 圆角半径 默认为0
   */
   // 设置矩形颜色
   ctx.fillStyle = "#fff";
   // 进行绘制
   ctx.fillRoundRect(15, 179, 345, 365.5, 10);
})
~~~
## 旧 API 的兼容

~~~html
<canvas style="width: 300px; height: 200px" canvas-id="firstCanvas"></canvas>
~~~

~~~js
// 创建绘制矩形工具
const drawPoster = await DrawPoster.build("firstCanvas");
// 添加绘制任务
drawPoster.draw(async (ctx) => {
  const path = "https://....";
  await ctx.drawLoadImage(path, 0, 0, 50, 50);
});
// 进行绘制
await drawPoster.createImagePath({
    width: 300,
    height: 300,
    destWidth: 300,
    destHeight: 300
});
~~~
