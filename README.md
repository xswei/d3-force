# d3-force

这个模块实现了用以模拟粒子物理运动的 [velocity Verlet](https://en.wikipedia.org/wiki/Verlet_integration) 数值积分器。仿真思路如下: 它假设任意单位时间步长 Δ*t* = 1，所有的粒子的单位质量常量 *m* = 1。作用在每个粒子上的合力 *F* 相当于在单位时间 Δ*t* 内的恒定加速度 *a*。并且可以简单的通过为每个粒子添加速度并计算粒子的位置来模拟仿真。

在信息可视化领域，物理仿真在研究 [networks](http://bl.ocks.org/mbostock/ad70335eeef6d167bc36fd3c04378048) 和 [hierarchies](http://bl.ocks.org/mbostock/95aa92e2f4e8345aaa55a4a94d41ce37) 时非常有用。

[<img alt="Force Dragging III" src="https://raw.githubusercontent.com/d3/d3-force/master/img/graph.png" width="420" height="219">](http://bl.ocks.org/mbostock/ad70335eeef6d167bc36fd3c04378048)[<img alt="Force-Directed Tree" src="https://raw.githubusercontent.com/d3/d3-force/master/img/tree.png" width="420" height="219">](http://bl.ocks.org/mbostock/95aa92e2f4e8345aaa55a4a94d41ce37)

你可以使用带有碰撞检测的圆盘，比如 [bubble charts](http://www.nytimes.com/interactive/2012/09/06/us/politics/convention-word-counts.html) 或 [beeswarm plots](http://bl.ocks.org/mbostock/6526445e2b44303eebf21da3b6627320):

[<img alt="Collision Detection" src="https://raw.githubusercontent.com/d3/d3-force/master/img/collide.png" width="420" height="219">](http://bl.ocks.org/mbostock/31ce330646fa8bcb7289ff3b97aab3f5)[<img alt="Beeswarm" src="https://raw.githubusercontent.com/d3/d3-force/master/img/beeswarm.png" width="420" height="219">](http://bl.ocks.org/mbostock/6526445e2b44303eebf21da3b6627320)

甚至可以用来作为一个基本的物理引擎，比如:

[<img alt="Force-Directed Lattice" src="https://raw.githubusercontent.com/d3/d3-force/master/img/lattice.png" width="480" height="250">](http://bl.ocks.org/mbostock/1b64ec067fcfc51e7471d944f51f1611)

用本模块为一组 [nodes](#simulation_nodes) 创建一个 [simulation(仿真)](#simulation)，并组合需要的 [forces(力模型)](#simulation_force)。然后 [监听](#simulation_on) `tick` 事件来不断更新图形系统，比如 `Canvas` 或 `SVG`.

## Installing

NPM 安装: `npm install d3-force`。 也可以下载 [最新发行版](https://github.com/d3/d3-force/releases/latest). 此外还可以直接从 [d3js.org](https://d3js.org) 以 [标准库](https://d3js.org/d3-force.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分直接载入. 支持 `AMD`, `CommonJS` 以及最基本的标签引入形式，如果使用标签引入则会暴露全局 `d3` 变量:

```html
<script src="https://d3js.org/d3-collection.v1.min.js"></script>
<script src="https://d3js.org/d3-dispatch.v1.min.js"></script>
<script src="https://d3js.org/d3-quadtree.v1.min.js"></script>
<script src="https://d3js.org/d3-timer.v1.min.js"></script>
<script src="https://d3js.org/d3-force.v1.min.js"></script>
<script>

var simulation = d3.forceSimulation(nodes);

</script>
```

[在浏览器中测试 `d3-force`](https://tonicdev.com/npm/d3-force)

## API Reference

### Simulation

<a name="forceSimulation" href="#forceSimulation">#</a> d3.<b>forceSimulation</b>([<i>nodes</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js "Source")

使用指定的 [*nodes*](#simulation_nodes) 创建一个新的没有任何 [forces(力模型)](#simulation_force) 的仿真。如果没有指定 *nodes* 则默认为空数组。仿真会自动 [starts(启动)](#simulation_restart)；使用 [*simulation*.on](#simulation_on) 来监听仿真运行过程中的 `tick` 事件。如果你想手动运行仿真，则需要调用 [*simulation*.stop](#simulation_stop) 然后根据需求调用 [*simulation*.tick](#simulation_tick)。

<a name="simulation_restart" href="#simulation_restart">#</a> <i>simulation</i>.<b>restart</b>() [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L80 "Source")

重新启动仿真的内部定时器并且返回仿真。与 [*simulation*.alphaTarget](#simulation_alphaTarget) 或 [*simulation*.alpha](#simulation_alpha) 结合使用，这个方法可以在交互期间再次激活仿真，比如拖拽节点或者在使用 [*simulation*.stop](#simulation_stop) 临时暂停仿真后使用。

<a name="simulation_stop" href="#simulation_stop">#</a> <i>simulation</i>.<b>stop</b>() [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L84 "Source")

暂停仿真内部的定时器并返回当前仿真。如果仿真内部定时器已经处于停止状态则什么都不做。这个方法在手动运行仿真时很有用，参考 [*simulation*.tick](#simulation_tick)。

<a name="simulation_tick" href="#simulation_tick">#</a> <i>simulation</i>.<b>tick</b>() [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L38 "Source")

按指定的迭代次数手动执行仿真，并返回仿真。如果没有指定 *iterations* 则默认为 1，也就是迭代一次(单步)。

对于每一次迭代，仿真都会通过 ([*alphaTarget*](#simulation_alphaTarget) - *alpha*) × [*alphaDecay*](#simulation_alphaDecay) 计算当前的 [*alpha*](#simulation_alpha) 值。然后调用注册的 [力模型](#simulation_force)，并传入当前新的 *alpha* 值；然后通过 *velocity* × [*velocityDecay*] 来递减每个节点的速度；最后
最后根据速度计算出每个节点的位置。

这个方法不会派发 [events](#simulation_on)；事件仅仅在创建仿真自动启动或者使用 [*simulation*.restart](#simulation_restart) 时通过内部的定时器触发。自然迭代次数为 ⌈*log*([*alphaMin*](#simulation_alphaMin)) / *log*(1 - [*alphaDecay*](#simulation_alphaDecay))⌉ 也就是默认为 300 次。

这个方法可以与 [*simulation*.stop](#simulation_stop) 结合使用计算一个 [static force layout(静态力导向布局)](https://bl.ocks.org/mbostock/1667139)。对于规模比较大的图，静态布局应该使用 [worker](https://bl.ocks.org/mbostock/01ab2e85e8727d6529d20391c0fd9a16) 计算避免阻塞用户界面。

<a name="simulation_nodes" href="#simulation_nodes">#</a> <i>simulation</i>.<b>nodes</b>([<i>nodes</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L88 "Source")

如果指定了 *nodes* 则将仿真的节点设置为指定的对象数组，并根据需要创建它们的位置和速度，然后 [重新初始化](#force_initialize) 绑定的 [力模型](#simulation_force)，并返回当前仿真。如果没有指定 *nodes* 则返回当前仿真的节点数组。

每个 *node* 必须是一个对象类型，下面的几个属性将会被仿真系统添加:

* `index` - 节点在 *nodes* 数组中的索引
* `x` - 节点当前的 *x*-坐标
* `y` - 节点当前的 *y*-坐标
* `vx` - 节点当前的 *x*-方向速度
* `vy` - 节点当前的 *y*-方向速度

位置 ⟨*x*,*y*⟩ 以及速度 ⟨*vx*,*vy*⟩ 随后可能被仿真中的 [力模型](#forces) 修改. 如果 *vx* 或 *vy* 为 NaN, 则速度会被初始化为 ⟨0,0⟩. 如果 *x* 或 *y* 为 NaN, 则位置会按照 [phyllotaxis arrangement](http://bl.ocks.org/mbostock/11478058) 被初始化, 这样初始化布局是为了能使得节点在原点周围均匀分布。

如果想要某个节点固定在一个位置，可以指定以下两个额外的属性:

* `fx` - 节点的固定 *x*-位置
* `fy` - 节点的固定 *y*-位置

每次 [tick](#simulation_tick) 结束后，节点的 *node*.x 会被重新设置为 *node*.fx 并且 *node*.vx 会被设置为 0；同理 *node*.y 会被重新替换为 *node.fy* 并且 *node*.vy 被设置为 0；如果想要某个节点解除固定，则将 *node*.fx 和 *node*.fy 设置为 null 或者删除这两个属性。

如果指定的节点数组发生了变化，比如添加或删除了某些节点，则这个方法必须使用新的节点数组重新被调用一次以通知仿真发生了变化。仿真不会对输入数组做副本。

<a name="simulation_alpha" href="#simulation_alpha">#</a> <i>simulation</i>.<b>alpha</b>([<i>alpha</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L92 "Source")

如果指定了 *alpha* 则将仿真的当前 *alpha* 值设置为指定的值，必须在 [0,1] 之间。如果没有指定 *alpha* 则返回当前的 *alpha* 值，默认为 1。

<a name="simulation_alphaMin" href="#simulation_alphaMin">#</a> <i>simulation</i>.<b>alphaMin</b>([<i>min</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L96 "Source")

如果指定了 *min* 则将 *alpha* 的最小值设置为指定的值，需要在 [0, 1] 之间。如果没有指定 *min* 则返回当前的最小 *alpha* 值，默认为 0.001. 在仿真内部，会不断的减小 [*alpha*](#simulation_alpha) 值直到 [*alpha*](#simulation_alpha) 值小于 最小 *alpha*。默认的 [alpha decay rate(alpha 衰减系数)](#simulation_alphaDecay) 为 ~0.0228，因此会进行 300 次迭代。

<a name="simulation_alphaDecay" href="#simulation_alphaDecay">#</a> <i>simulation</i>.<b>alphaDecay</b>([<i>decay</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L100 "Source")

如果指定了 *decay* 则将当前的 [*alpha*](#simulation_alpha) 衰减系数设置为指定的值，要在[0, 1] 之间。如果没有指定 *decay* 则返回当前的 *alpha* 衰减系数，默认为 0.0228… = 1 - *pow*(0.001, 1 / 300)，其中 0.001 是默认的 [最小 *alpha*](#simulation_alphaMin).

*alpha* 衰减系数定义了当前的 *alpha* 值向 [target *alpha*](#simulation_alphaTarget) 迭代的快慢。默认的目标 *alpha* 为 0 因此从布局形式上可以认为衰减系数决定了布局冷却的快慢。衰减系数越大，布局冷却的越快，但是衰减系数大的话会引起迭代次数不够充分，导致效果不够好。衰减系数越小，迭代次数越多，最终的布局效果越好。如果想要布局永远停不下来则可以将衰减系数设置为 0；也可以设置 [target *alpha*](#simulation_alphaTarget) 大于 [minimum *alpha*](#simulation_alphaMin) 达到相同的效果。

<a name="simulation_alphaTarget" href="#simulation_alphaTarget">#</a> <i>simulation</i>.<b>alphaTarget</b>([<i>target</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L104 "Source")

如果指定了 *target* 则将当前的目标 [*alpha*](#simulation_alpha) 设置为指定的值，需要在 [0, 1] 之间。如果没有指定 *target* 则返回当前默认的目标 *alpha* 值, 默认为 0.

<a name="simulation_velocityDecay" href="#simulation_velocityDecay">#</a> <i>simulation</i>.<b>velocityDecay</b>([<i>decay</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L108 "Source")

如果指定了 *decay* 则设置仿真的速度衰减系数并返回仿真，范围为 [0, 1]。如果没有指定 *decay* 则返回当前的速度衰减系数，默认为 0.4，衰减系数类似于阻力。每次 `tick` 结束后每个节点的速度都会乘以 1 - *decay* 以降低节点的运动速度。速度衰减系数与 [alpha decay rate](#simulation_alphaDecay) 类似，较低的衰减系数可以使得迭代次数更多，其布局结果也会更理性，但是可能会引起数值不稳定从而导致震荡。

<a name="simulation_force" href="#simulation_force">#</a> <i>simulation</i>.<b>force</b>(<i>name</i>[, <i>force</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L112 "Source")

如果指定了 *force* 则表示为仿真添加指定 *name* 的 [力模型](#forces) 并返回仿真。如果没有指定 *force* 则返回当前仿真的对应 *name* 的力模型，如果没有对应的 *name* 则返回 `undefined`。 (默认情况下仿真没有任何力模型，需要手动添加)。例如创建一个用来对图进行布局的仿真，可以如下:

```js
var simulation = d3.forceSimulation(nodes)
    .force("charge", d3.forceManyBody())
    .force("link", d3.forceLink(links))
    .force("center", d3.forceCenter());
```

如果要移除对应的 *name* 的仿真，可以为其指定 `null`，比如:

```js
simulation.force("charge", null);
```

<a name="simulation_find" href="#simulation_find">#</a> <i>simulation</i>.<b>find</b>(<i>x</i>, <i>y</i>[, <i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L116 "Source")

返回距离 ⟨*x*,*y*⟩ 位置最近的节点，并可以指定搜索半径 *radius*。 如果没有指定 *radius* 则默认为无穷大。如果在指定的搜索区域内没有找到节点，则返回 `undefined`。

<a name="simulation_on" href="#simulation_on">#</a> <i>simulation</i>.<b>on</b>(<i>typenames</i>, [<i>listener</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L139 "Source")

如果指定了 *listener* 则将其指定的 *typenames* 的回调。如果对应的 *typenames* 已经存在事件监听器，则将其替换。如果 *listener* 为 `null` 则表示移除对应 *typenames* 的事件监听器。如果没有指定 *listener* 则返回第一个符合条件的 *typenames* 对应的事件监听器，当指定的事件触发时，每个回调都会被调用，回调中 `this` 指向仿真本身。

*typenames* 可以由多个由空格隔开的 *typename*。每个 *typename* 都由 *type* 和可选的 *name* 组成，用 (`.`) 连接。比如  `tick.foo` 和 `tick.bar`。也就是可以为同一种事件类型注册多个事件监听器。其中 *type* 必须为以下几种:

* `tick` - 仿真内部定时器每次 `tick` 之后。
* `end` - 当 *alpha* < [*alphaMin*](#simulation_alphaMin) 时仿真内部定时器停止。

需要注意的是，`tick` 事件在手动调用 [*simulation*.tick](#simulation_tick) 时不会执行。`tick` 事件只会被内部定时器调用用以模拟布局过程。如果需要调整布局，应该在 [forces](#simulation_force) 中注册力模型而不是在每次 `tick` 时修改节点位置。

参考 [*dispatch*.on](https://github.com/d3/d3-dispatch#dispatch_on) 获取更多详情。

### Forces

*force（力模型）* 是一个用以修改节点位置和速度的函数；在力模型上下文中，*force* 可以施加电荷或重力之类的经典物理力学，也可以用来解决几何约束，例如将节点保持在边界框内或者保持节点之间的相对距离。例如将节点移动到原点 ⟨0,0⟩ 的简单力学模型可以实现如下:

```js
function force(alpha) {
  for (var i = 0, n = nodes.length, node, k = alpha * 0.1; i < n; ++i) {
    node = nodes[i];
    node.vx -= node.x * k;
    node.vy -= node.y * k;
  }
}

// 只在初始化时被调用，可选的
force.initialize = function (nodes) {
  // do something
}

const simulation = d3.forceSimulation()
  .force('myForce', force)
```

力学模型通常读取节点的当前位置 ⟨*x*,*y*⟩ 然后修改节点的速度 ⟨*vx*,*vy*⟩。但是力学图也能预测到节点的下一个位置  ⟨*x* + *vx*,*y* + *vy*⟩；这对于通过 [iterative relaxation(迭代松弛)](https://en.wikipedia.org/wiki/Relaxation_\(iterative_method\)) 来解决几何约束是必需的。力学模型也可以直接修改节点的位置，有时可以通过直接修改节点位置来避免向仿真中添加能量，比如在视口中重新进行仿真。

仿真通常需要多个力学模型的组合，下面是内置的几种力学模型:

* [Centering(向心力)](#centering)
* [Collision(碰撞检测)](#collision)
* [Links(弹簧力)](#links)
* [Many-Body(电荷力)](#many-body)
* [Positioning](#positioning)

力模型可以选择通过 [*force*.initialize](#force_initialize) 来接收仿真的节点数组。

<a name="_force" href="#_force">#</a> <i>force</i>(<i>alpha</i>) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L44 "Source")

应用此力模型，可以选择观测指定的 *alpha* 值。通常情况下，该力在节点数组被传递给 [*force*.initialize](#force_initialize) 之前被应用，但是某些力可能适用于节点子集或者有不同的行为，比如 [d3.forceLink](#links) 被应用于每个边的 `source` 和 `target`.

<a name="force_initialize" href="#force_initialize">#</a> <i>force</i>.<b>initialize</b>(<i>nodes</i>) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L71 "Source")

将 *nodes* 数组分配给此力模型。这个方法在将力模型通过 [*simulation*.force](#simulation_force) 添加到仿真中并且通过 [*simulation*.nodes](#simulation_nodes) 指定节点数组时被调用。可以在初始化阶段执行必要的工作，比如评估每个节点的参数特征，要避免在每次使用力模型时执行重复的工作。

#### Centering

`center` (向心力) 可以将所有的节点的中心统一整体的向指定的位置 ⟨[*x*](#center_x),[*y*](#center_y)⟩ 移动。这种力强制修改每个节点的位置，但是不会修改速度，因为修改速度会造成节点在期望的位置附近抖动。这种力可以辅助保持所有的节点在视口中心，与 [positioning force](#positioning) 不同的是它不会修改节点之间的相对位置。

<a name="forceCenter" href="#forceCenter">#</a> d3.<b>forceCenter</b>([<i>x</i>, <i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L1 "Source")

使用指定的 [*x*-](#center_x) 和 [*y*-](#center_y) 坐标创建一个新的向心力模型。如果 *x* 和 *y* 没有指定则默认为 ⟨0,0⟩.

<a name="center_x" href="#center_x">#</a> <i>center</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L27 "Source")

如果指定了 *x* 则将向心力的中心点 *x* 坐标设置为指定的数值并返回力学模型。如果没有指定则返回当前的中心点 *x* 坐标，默认为 0.

<a name="center_y" href="#center_y">#</a> <i>center</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L31 "Source")

如果指定了 *y* 则将向心力的中心点 *y* 坐标设置为指定的数值并返回力学模型。如果没有指定则返回当前的中心点 *y* 坐标，默认为 0.

#### Collision

`collision（碰撞力）` 将节点视为具有一定 [radius](#collide_radius) 的圆，而不是点，并且阻止节点之间的重叠。从形式上来说，假设节点 *a* 和节点 *b* 是两个独立的节点，则 *a* 和 *b* 之间最小距离为 *radius*(*a*) + *radius*(*b*)。为减少抖动，默认情况下，碰撞检测是一个可配置 [strength(强度)](#collide_strength) 和 [iteration count(迭代次数)](#collide_iterations) 的软约束。

<a name="forceCollide" href="#forceCollide">#</a> d3.<b>forceCollide</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js "Source")

根据指定的 [*radius*](#collide_radius) 创建一个新的圆形区域的碰撞检测。如果没有指定 *radius* 则默认所有的节点半径都为 1.

<a name="collide_radius" href="#collide_radius">#</a> <i>collide</i>.<b>radius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L86 "Source")

如果指定的 *radius* 则表示设置节点半径访问器，*radius* 可以是一个数值或者方法，如果是方法则会为每个节点调用，并返回碰撞检测力模型。如果没有指定 *radius* 则返回当前的 *radius* 访问器，默认为:

```js
function radius() {
  return 1;
}
```

半径访问器为仿真中的每个节点调用并传递当前的节点 *node* 以及基于 0 的 *index*。其返回值在内部被保存，这样的话每个节点的半径仅在初始化以及使用新的半径访问器时才会被调用，而不是每次应用时候重新计算。

<a name="collide_strength" href="#collide_strength">#</a> <i>collide</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L82 "Source")

如果指定了 *strength* 则将碰撞强度设置为指定的数值，强度范围为 [0, 1]。并返回当前碰撞力模型。如果没有指定 *strength* 则返回当前的碰撞强度，默认为 `0.7`.

重叠的节点通过迭代松弛来解决。对于每个节点，将估算在下一次 `tick` 时候重叠(使用预期位置 ⟨*x* + *vx*,*y* + *vy*⟩ )的其他节点。然后节点的速度会被修改以避免重叠。速度的变化会受 *strength* 的影响，这样仿真的重叠问题可以得到一个比较稳定的解。

<a name="collide_iterations" href="#collide_iterations">#</a> <i>collide</i>.<b>iterations</b>([<i>iterations</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L78 "Source")

如果指定了 *iterations* 则将每次应用碰撞检测力模型时候的迭代次数设置为指定的数值。如果没有指定 *iterations* 则返回当前的迭代次数，默认为 1。迭代次数越大，最终的布局越优，但是会增加程序运行上的消耗。

#### Links

`link froce(弹簧模型)` 可以根据 [link distance](#link_distance) 将有关联的两个节点拉近或者推远。力的强度与被链接两个节点的距离成比例，类似弹簧力。

<a name="forceLink" href="#forceLink">#</a> d3.<b>forceLink</b>([<i>links</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js "Source")

根据指定的 *links* 以及默认参数创建一个弹簧力模型。如果没有指定 *links* 则默认为空数组。

<a name="link_links" href="#link_links">#</a> <i>link</i>.<b>links</b>([<i>links</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L92 "Source")

如果指定了 *links* 则将其设置为该弹簧力模型的关联边数组，并重新计算每个边的 [distance](#link_distance) 和 [strength](#link_strength) 参数，返回当前力模型。如果没有指定 *links* 则返回当前力模型的边数组，默认为空。

每个边都是都是一个包含以下属性的对象:

* `source` - 边的源节点; 参考 [*simulation*.nodes](#simulation_nodes)
* `target` - 边的目标节点; 参考 [*simulation*.nodes](#simulation_nodes)
* `index` - 基于 0 的在 *links* 中的索引, 会自动分配

为方便起见，边的源和目标节点可以在初始时使用索引或者字符串标识符，而不一定非要为对象引用。参考 [*link*.id](#link_id)。当弹簧力被 [initialized](#force_initialize)(或者重新初始化，节点或者边改变)，任何不是引用的 *link*.source or *link*.target 都会根据指定的标识符被转为 *node* 引用形式。

如果指定的 *links* 数组被修改，比如添加或者移除边，这个方法必须使用新的数组重新调用一次，这个方法不会创建指定数组的副本。

<a name="link_id" href="#link_id">#</a> <i>link</i>.<b>id</b>([<i>id</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L96 "Source")

如果指定了 *id* 则将节点的 `id` 访问器设置为指定的函数并返回弹簧模型。如果 *id* 没有指定则返回当前的节点 `id` 访问器，默认为数值类型的节点索引 *node*.index:

```js
function id(d) {
  return d.index;
}
```

默认的 `id` 访问器允许每个边的源节点和目标节点为基于 0 的 [nodes](#simulation_nodes) 数组的节点索引。例如:

```js
var nodes = [
  {"id": "Alice"},
  {"id": "Bob"},
  {"id": "Carol"}
];

var links = [
  {"source": 0, "target": 1}, // Alice → Bob
  {"source": 1, "target": 2} // Bob → Carol
];
```

现在可以使用字符串 `id` 来表示节点：

```js
function id(d) {
  return d.id;
}
```

使用 `id` 访问器可以将源和目标属性设置为字符串:

```js
var nodes = [
  {"id": "Alice"},
  {"id": "Bob"},
  {"id": "Carol"}
];

var links = [
  {"source": "Alice", "target": "Bob"},
  {"source": "Bob", "target": "Carol"}
];
```

这个方法在使用 `JSON` 表示图数据时特别有用，因为 JSON 数据不允许引用。参考 [this example](http://bl.ocks.org/mbostock/f584aa36df54c451c94a9d0798caed35).

`id` 访问器会在力模型初始化时为每个节点调用，与修改 [nodes](#simulation_nodes) 或 [links](#link_links) 一样，并传递当前节点以及基于 0 的索引。

<a name="link_distance" href="#link_distance">#</a> <i>link</i>.<b>distance</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L108 "Source")

如果指定了 *distance* 则将距离访问器设置为指定的数值或者方法，并重新计算每个边的距离访问器，返回当前力模型。如果没有指定 *distance* 则返回当前的距离访问器，默认为：

```js
function distance() {
  return 30;
}
```

距离访问器会为每个 [link](#link_links) 调用，并传递 *link* 以及基于 0 的索引。返回数值会被存储在内部，这样只有在初始化以及重新设置新的 *distance* 时才会重新计算，而不会在每次应用力模型的时候重复计算。

<a name="link_strength" href="#link_strength">#</a> <i>link</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L104 "Source")

如果指定了 *strength* 则将强度访问器设置为指定的数值或者方法，并重新计算每个边的强度访问器，返回当前力模型。如果没有指定 *strength* 则返回当前的强度访问器，默认为：

```js
function strength(link) {
  return 1 / Math.min(count(link.source), count(link.target));
}
```

其中 *count*(*node*) 是一个根据指定节点计算其出度和入度的函数。使用这种方法设置会自动降低连接到度大的节点的边的强度，有利于提高稳定性。

强度访问器会为每个 [link](#link_links) 调用，并传递 *link* 以及基于 0 的索引。返回数值会被存储在内部，这样只有在初始化以及重新设置新的 *strength* 时才会重新计算，而不会在每次应用力模型的时候重复计算。

<a name="link_iterations" href="#link_iterations">#</a> <i>link</i>.<b>iterations</b>([<i>iterations</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L100 "Source")

如果指定了 *iterations* 则将迭代次数设置为指定的值并返回当前力模型。如果没有指定 *iterations* 则返回当前的迭代次数，默认为 `1`。增加迭代次数会增加约束的刚性，这对于 [complex structures such as lattices](http://bl.ocks.org/mbostock/1b64ec067fcfc51e7471d944f51f1611) 会有用，但是同时也会增加程序运行消耗。

#### Many-Body

`many-body`(N-body问题，译为电荷力比较容易理解) 在所有的节点之间相互作用。如果 [strength](#manyBody_strength) 为正可以被用来模拟重力(吸引力)，如果强度为负可以用来模拟排斥力。这个力模型的实现采用四叉树以及 [Barnes–Hut approximation](https://en.wikipedia.org/wiki/Barnes–Hut_simulation) 大大提高了性能。精确度可以使用 [theta](#manyBody_theta) 参数自定义。

与弹簧力不同的是，弹簧力仅仅影响两端的节点，而电荷力是全局的: 每个节点都受到其他所有节点的影响，甚至他们处于不连通的子图。

<a name="forceManyBody" href="#forceManyBody">#</a> d3.<b>forceManyBody</b>() [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js "Source")

创建一个使用默认参数的电荷力模型。

<a name="manyBody_strength" href="#manyBody_strength">#</a> <i>manyBody</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L97 "Source")

如果指定了 *strength* 则将强度访问器设置为指定的数值或者方法，重新评估每个节点的强度访问器并返回此电荷力。若强度为正值则表示节点之间相互吸引，负值表示节点之间相互排斥。如果没有指定 *strength* 则返回当前的强度访问器，默认为:

```js
function strength() {
  return -30;
}
```

强度访问器会为仿真中的每个节点调用，并传递当前的节点 *node* 以及基于 0 的索引。返回值结果会在内部被存储，也就是只在初始化和设置新的 *strength* 的时候才会重新计算，而不是在每次应用力模型的时候重新计算。

<a name="manyBody_theta" href="#manyBody_theta">#</a> <i>manyBody</i>.<b>theta</b>([<i>theta</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L109 "Source")

如果指定了 *theta* 则 Barnes–Hut 算法的临界阈值设置为指定的值并返回当前力模型。如果没有指定 *theta* 则返回当前的值，默认为 `0.9`.

为了加快计算，这个力模型基于 [Barnes–Hut approximation](http://en.wikipedia.org/wiki/Barnes–Hut_simulation) 进行实现，其时间复杂度为 O(*n* log *n*)，其中 *n* 为 [nodes](#simulation_nodes) 个数。在每次应用中，[quadtree](https://github.com/d3/d3-quadtree) 存储当前节点的位置；然后对于每个节点，计算其受其他所有节点的合力。对于距离很远的节点群，可以通过聚类将其视为一个更大的节点来模拟电荷力。*theta* 参数决定了这种近似精度：如果比率 *w* / *l* (*w* 表示四叉树正方形的宽度，*l* 表示节点到正方形中心的距离) 小于 *theta* 则将该正方形内的所有节点视为一个单独的节点。

<a name="manyBody_distanceMin" href="#manyBody_distanceMin">#</a> <i>manyBody</i>.<b>distanceMin</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L101 "Source")

如果指定了 *distance* 则设置电荷力模型的最小节点间距参考距离。如果没有指定 *distance* 则返回当前默认的最小参考距离，默认为 1。最小间距在相邻的节点之间建立了一个强度上限以避免不稳定的情况。极端的情况是两个节点完全重叠，则它们之间的斥力可能无限大，此时力的方向是随机的，因此设置最小间距是必要的。

<a name="manyBody_distanceMax" href="#manyBody_distanceMax">#</a> <i>manyBody</i>.<b>distanceMax</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L105 "Source")

如果指定了 *distance* 则将节点之间的最大距离设置为指定的值并返回当前力模型。如果没有指定 *distance* 则返回当前默认的最大距离，默认为无穷大。指定最大间距可以提高性能，有利于生成局部布局。

#### Positioning

[*x*](#forceX)- 和 [*y*](#forceY)-定位力模型可以将节点沿着指定的维度进行排列。与 [*radial*](#forceRadial) 力类似，只不过环形力的参考位置是一个闭合的环。力的强度与节点位置到目标位置的距离成正比。虽然这些里可以定位某个单个节点，但是主要用于适用于所有(或大多数)节点的全局力布局。

<a name="forceX" href="#forceX">#</a> d3.<b>forceX</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js "Source")

根据指定的 [*x*](#x_x) 创建一个沿着 *x*-轴 方向的新的定位力模型。如果 *x* 没有指定则默认为 0.

<a name="x_strength" href="#x_strength">#</a> <i>x</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js#L32 "Source")

如果指定了 *strength* 则将强度访问器设置为指定的数值或者方法，强度访问器会为每个节点进行调用，并返回当前力模型。*strength* 决定了节点 *x*-速度的增量: ([*x*](#x_x) - *node*.x) × *strength*. 例如值为 0.1 时表示节点在每次应用时应该从当前 *x*-位置向目标 *x*-位置移动十分之一。值越大则移动的越快，但是会牺牲其他力模型或者约束。不建议使用 [0, 1] 之外的值。

如果没有指定 *strength* 则返回当前的强度访问器，默认为:

```js
function strength() {
  return 0.1;
}
```

强度访问器会为仿真中的每个节点进行调用，并传递当前的节点 *node* 以及基于 0 的索引。其值会被存储在内部，也就是只在初始化或者修改 *strength* 时会重新计算，而不会在每次应用时都重新计算。

<a name="x_x" href="#x_x">#</a> <i>x</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js#L36 "Source")

如果指定了 *x* 则设置 *x*-坐标访问器为指定的数值或者方法，并重新计算每个节点的 *x*-访问器，并返回当前力模型。如果没有指定 *x* 则返回当前的 *x*-访问器，默认为:

```js
function x() {
  return 0;
}
```

*x*-访问器会为仿真中的每个节点进行调用，并传递当前 *node* 以及基于 0 的索引。计算结果会被内部存储，只有在初始化或者重新设置 *x* 访问器时参会重新计算，避免在每次应用时都重新计算。

<a name="forceY" href="#forceY">#</a> d3.<b>forceY</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js "Source")

根据指定的 [*y*](#y_y) 创建一个沿着 *y*-轴 方向的新的定位力模型。如果 *y* 没有指定则默认为 0.

<a name="y_strength" href="#y_strength">#</a> <i>y</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js#L32 "Source")

如果指定了 *strength* 则将强度访问器设置为指定的数值或者方法，强度访问器会为每个节点进行调用，并返回当前力模型。*strength* 决定了节点 *y*-速度的增量: ([*y*](#y_y) - *node*.y) x *strength*. 例如值为 0.1 时表示节点在每次应用时应该从当前 *y*-位置向目标 *y*-位置移动十分之一。值越大则移动的越快，但是会牺牲其他力模型或者约束。不建议使用 [0, 1] 之外的值。

如果没有指定 *strength* 则返回当前的强度访问器，默认为:

```js
function strength() {
  return 0.1;
}
```

强度访问器会为仿真中的每个节点进行调用，并传递当前的节点 *node* 以及基于 0 的索引。其值会被存储在内部，也就是只在初始化或者修改 *strength* 时会重新计算，而不会在每次应用时都重新计算。

<a name="y_y" href="#y_y">#</a> <i>y</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js#L36 "Source")

如果指定了 *y* 则设置 *y*-坐标访问器为指定的数值或者方法，并重新计算每个节点的 *y*-访问器，并返回当前力模型。如果没有指定 *y* 则返回当前的 *y*-访问器，默认为:

```js
function y() {
  return 0;
}
```

*y*-访问器会为仿真中的每个节点进行调用，并传递当前 *node* 以及基于 0 的索引。计算结果会被内部存储，只有在初始化或者重新设置 *y* 访问器时参会重新计算，避免在每次应用时都重新计算。

<a name="forceRadial" href="#forceRadial">#</a> d3.<b>forceRadial</b>(<i>radius</i>[, <i>x</i>][, <i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

[<img alt="Radial Force" src="https://raw.githubusercontent.com/d3/d3-force/master/img/radial.png" width="420" height="219">](https://bl.ocks.org/mbostock/cd98bf52e9067e26945edd95e8cf6ef9)

创建一个沿着指定 [*radius*](#radial_radius)、圆心坐标在 ⟨[*x*](#radial_x),[*y*](#radial_y)⟩ 的圆环的环形布局。如果没有指定 *x* 和 *y* 则默认为 ⟨0,0⟩.

<a name="radial_strength" href="#radial_strength">#</a> <i>radial</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

如果指定了 *strength* 则将强度访问函数设置为指定的数值或者方法，并重新评估每个节点的强度访问器，返回当前力模型。*strength* 决定了节点的  *x*- 和 *y*-速度的增量。例如 0.1 表示每次应用力模型时从当前为值向最近的圆环上位置移动十分之一。值越大移动的越快，但是会牺牲其他力模型或者约束，因此建议使用 [0, 1] 之间的值。

如果没有指定 *strength* 则返回当前的强度访问器，默认为:

```js
function strength() {
  return 0.1;
}
```

强度访问器会为仿真中的每个节点进行调用，并传递当前节点 *node* 以及基于 0 的索引。计算结果会被内部存储，只有在初始化或者重新设置新的强度访问器时才会重新评估，避免在每次应用力模型时候重复计算。

<a name="radial_radius" href="#radial_radius">#</a> <i>radial</i>.<b>radius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

如果指定了 *radius* 则将圆环的 *radius* 设置为指定的数值或者函数，并重新评估 *radius* 访问器，返回当前力模型。如果没有指定 *radius* 则返回当前的 *radius* 访问器。

*radius* 访问器会为仿真中的每个节点进行调用，并传递当前的节点 *node* 以及基于 0 的索引。计算结果会被内部存储，只有在初始化或者重新设置新的 *radius* 时参会重新评估，以避免在每次应用时都重新计算。

<a name="radial_x" href="#radial_x">#</a> <i>radial</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

如果指定了 *x* 则将圆环的 *x*-坐标设置为指定的数值并返回当前力模型。如果没有指定则返回当前 *x*-坐标，默认为 0。

<a name="radial_y" href="#radial_y">#</a> <i>radial</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

如果指定了 *y* 则将圆环的 *y*-坐标设置为指定的数值并返回当前力模型。如果没有指定则返回当前 *y*-坐标，默认为 0。
