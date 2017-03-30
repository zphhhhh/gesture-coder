## 如何运行？

- 可直接使用 chrome 浏览器打开，进入开发者模式的移动设备模式即可测试。[测试环境：windows 10, chrome 57]
- 全局安装 nodejs 模块 http-server，启动 http-server 可在移动端测试。[测试环境：iPhone 7, iOS 10.3]

## 实现思路

- 9 个圆圈使用 `div` 配合 `border-radius` 完成
- 手势移动时的线段也是 `div` (`height: 4px; border-radius: 2px;`)
- 使用 `transform-origin` / `transform: rotate` 控制线段的旋转与角度
- 手势移动时会遍历所有非 `.active` 的圆圈，若进入了圆圈范围，则连接两个圆圈并新建一条线段
- 每个圆圈代表一个数字，自左至右为 1-9，初始密码为 `1235789`（大写的 `Z` 形状）

## 使用 canvas ？

最开始想到了 `canvas` 实现，与纯 `div` 相比可以不计算线段旋转角度，可以节省不少的逻辑处理。  
但由于使用 `canvas` 较少，手势移动时需要频繁绘制和擦除最近一条线段，不清楚如何只擦除这最近一条线段，时间仓促，只好改用 `div` 实现。  
还想到了一种 `canvas` 配合 `div` 使用，即圆圈和连接两个圆圈的线段都使用 `div` 实现，只有手势移动的线段使用 `canvas` 绘制和擦除，这样整个 `canvas` 画板只有一条线段，也很好处理。

## 遇到的问题

我先在 windows chrome 上测试，测试已经没有问题了，于是启动 http-server 用手机打开，怀着激动地心情，果然一堆 bug...

- 滑动不生效，只有第一个圆圈会被点亮。
    - 找到原因：使用了 `for of` 语句，Safari 不支持？多次尝试确认 Safari 的 `for of` 不支持迭代 `DOMList` 这种类数组元素
    - 解决方法：`for of` 改为一般的 `for i` 语句（或者将 `DOMList` 转为数组）
- 页面无法滑动
    - 找到原因：因为最初想到滑动时可能触发原生滚动事件，所以使用了 `e.preventDefault()` 阻止默认事件传播，结果点击其他地方不能滚动了...
    - 解决方法：只在触发手势解锁的时候阻止事件传播
- 滑动页面后线段的位置出错...
    - 找到原因：表示线段的 `div` 使用了 `absolute` 定位，计算 `left` 时使用圆圈的 `Element.getClientRects()[0].top`，而这里的 `top` 表示该 `div` 与可视区顶端距离，而非 `Element.offsetTop` 表示的 `div 与页面顶部距离
    - 解决方法：使用 `Element.offsetTop` / `offsetLeft` 代替 `Element.getClientRects()[0].top` / `left`
- 无痕模式下滑动手势没反应，起初是在手机上测试时忘了原密码，为了偷懒就开了无痕模式，结果...
    - 找到原因：Safari 的无痕模式禁止使用 localStorage
    - 解决方法：try catch 检测是否可以使用 localStorage，弹框提示

## 联系我

张棚贺，邮箱：apphe7@163.com
