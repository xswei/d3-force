# d3-force

这个模块实现了用以模拟粒子物理运动的 [velocity Verlet](https://en.wikipedia.org/wiki/Verlet_integration) 数值积分器。仿真的演化: 它假设任意单位时间步长 Δ*t* = 1，所有的粒子的单位质量常量 *m* = 1。作用在每个粒子上的合力 *F* 相当于在单位时间 Δ*t* 内的恒定加速度 *a*。并且可以简单的通过为每个粒子添加速度并计算粒子的位置来模拟仿真。

在信息可视化领域，物理仿真在研究 [networks](http://bl.ocks.org/mbostock/ad70335eeef6d167bc36fd3c04378048) 和 [hierarchies](http://bl.ocks.org/mbostock/95aa92e2f4e8345aaa55a4a94d41ce37) 时非常有用。

[<img alt="Force Dragging III" src="https://raw.githubusercontent.com/d3/d3-force/master/img/graph.png" width="420" height="219">](http://bl.ocks.org/mbostock/ad70335eeef6d167bc36fd3c04378048)[<img alt="Force-Directed Tree" src="https://raw.githubusercontent.com/d3/d3-force/master/img/tree.png" width="420" height="219">](http://bl.ocks.org/mbostock/95aa92e2f4e8345aaa55a4a94d41ce37)

你可以使用带有碰撞检测的圆形，比如 [bubble charts](http://www.nytimes.com/interactive/2012/09/06/us/politics/convention-word-counts.html) 或 [beeswarm plots](http://bl.ocks.org/mbostock/6526445e2b44303eebf21da3b6627320):

[<img alt="Collision Detection" src="https://raw.githubusercontent.com/d3/d3-force/master/img/collide.png" width="420" height="219">](http://bl.ocks.org/mbostock/31ce330646fa8bcb7289ff3b97aab3f5)[<img alt="Beeswarm" src="https://raw.githubusercontent.com/d3/d3-force/master/img/beeswarm.png" width="420" height="219">](http://bl.ocks.org/mbostock/6526445e2b44303eebf21da3b6627320)

甚至可以用来作为一个基本的物理引擎，比如:

[<img alt="Force-Directed Lattice" src="https://raw.githubusercontent.com/d3/d3-force/master/img/lattice.png" width="480" height="250">](http://bl.ocks.org/mbostock/1b64ec067fcfc51e7471d944f51f1611)

用本模块，为一组 [nodes](#simulation_nodes) 创建一个 [simulation(仿真)](#simulation)，并组合需要的 [forces(力模型)](#simulation_force)。然后 [listen(监听)](#simulation_on) `tick` 事件来不断更新图形系统，比如 Canvas 或 SVG.

## Installing

NPM 安装: `npm install d3-force`. 也可以下载 [latest release](https://github.com/d3/d3-force/releases/latest). 此外还可以直接从 [d3js.org](https://d3js.org) 以 [standalone library](https://d3js.org/d3-force.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分直接载入. 支持 AMD, CommonJS 以及最基本的标签引入形式，如果使用标签引入形式则会暴露全局 `d3_force` 变量:

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

[在浏览器中测试 d3-force.](https://tonicdev.com/npm/d3-force)

## API Reference

### Simulation

<a name="forceSimulation" href="#forceSimulation">#</a> d3.<b>forceSimulation</b>([<i>nodes</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js "Source")

使用指定的 [*nodes*](#simulation_nodes) 创建一个新的没有任何 [forces(力学模型)](#simulation_force) 的仿真。如果没有指定 *nodes* 则默认为空数组。仿真会自动 [starts](#simulation_restart)；使用 [*simulation*.on](#simulation_on) 来监听仿真运行过程中的 `tick` 事件。如果你想手动运行仿真，则需要调用 [*simulation*.stop](#simulation_stop) 然后根据需求调用 [*simulation*.tick](#simulation_tick)。

<a name="simulation_restart" href="#simulation_restart">#</a> <i>simulation</i>.<b>restart</b>() [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L80 "Source")

重新调用仿真的定时器并且返回仿真。与 [*simulation*.alphaTarget](#simulation_alphaTarget) 或 [*simulation*.alpha](#simulation_alpha) 结合使用，这个方法可以再次激活仿真，在有些交互比如拖拽节点或者在使用 [*simulation*.stop](#simulation_stop) 临时暂停仿真时候使用。

<a name="simulation_stop" href="#simulation_stop">#</a> <i>simulation</i>.<b>stop</b>() [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L84 "Source")

暂停仿真内部的定时器并返回当前仿真。如果仿真内部定时器已经处于停止状态则什么都不做。这个方法在手动运行仿真时很有用，参考 [*simulation*.tick](#simulation_tick)。

<a name="simulation_tick" href="#simulation_tick">#</a> <i>simulation</i>.<b>tick</b>() [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L38 "Source")

通过 ([*alphaTarget*](#simulation_alphaTarget) - *alpha*) × [*alphaDecay*](#simulation_alphaDecay) 递增当前的 [*alpha*](#simulation_alpha) 值。然后调用每个注册的 [force](#simulation_force) 并传递新的 *alpha*。然后通过 *velocity* × [*velocityDecay*](#simulation_velocityDecay) 来递减每个节点的速度并调整节点的位置。

这个方法不会分发 [events](#simulation_on)，事件仅仅在 [creation](#forceSimulation) 仿真或者调用 [*simulation*.restart](#simulation_restart) 的时候才会被分发。从仿真开始到结束的 `tick` 次数为 ⌈*log*([*alphaMin*](#simulation_alphaMin)) / *log*(1 - [*alphaDecay*](#simulation_alphaDecay))⌉ 也就是默认为 300 次。

这个方法可以与 [*simulation*.stop](#simulation_stop) 结合使用来创建 [static force layout(静态力学布局)](https://bl.ocks.org/mbostock/1667139)。对于大规模图而言静态布局应该 [in a web worker(在worker)](https://bl.ocks.org/mbostock/01ab2e85e8727d6529d20391c0fd9a16) 中计算以避免阻塞 UI 进程。

<a name="simulation_nodes" href="#simulation_nodes">#</a> <i>simulation</i>.<b>nodes</b>([<i>nodes</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L88 "Source")

如果指定了 *nodes* 则将仿真的节点设置为指定的对象数组，并根据需要创建它们的位置和速度，然后 
[re-initializes(重新初始化)](#force_initialize) 并且绑定 [forces(力模型)](#simulation_force) 返回当前仿真。如果没有指定 *nodes* 则返回当前仿真的节点数组。

每个 *node* 必须是一个对象类型，下面的几个属性将会被仿真系统添加:

* `index` - 节点在 *nodes* 数组中的索引
* `x` - 节点当前的 *x*-坐标
* `y` - 节点当前的 *y*-坐标
* `vx` - 节点当前的 *x*-方向速度
* `vy` - 节点当前的 *y*-方向速度

位置 ⟨*x*,*y*⟩ 以及速度 ⟨*vx*,*vy*⟩ 随后可能被仿真中的 [forces](#forces) 修改. 如果 *vx* 或 *vy* 为 NaN, 则速度会被初始化为 ⟨0,0⟩. 如果 *x* 或 *y* 为 NaN, 则位置会按照 [phyllotaxis arrangement(布局算法)](http://bl.ocks.org/mbostock/11478058) 被初始化, 这样初始化布局是为了能使得节点在原点周围均匀分布。

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

*alpha* 衰减系数定义了当前的 *alpha* 值向 [target *alpha*](#simulation_alphaTarget) 迭代快慢。默认的目标 *alpha* 为 0 因此从布局形式上可以认为衰减系数决定了布局冷却的快慢。衰减系数越大，布局冷却的越快，但是衰减系数大的话会引起迭代次数不够充分，导致效果不够好。衰减系数越小，迭代次数越多，最终的布局效果越好。如果想要布局永远停不下来则可以将衰减系数设置为 0；也可以设置 [target *alpha*](#simulation_alphaTarget) 大于 [minimum *alpha*](#simulation_alphaMin) 达到相同的效果。

<a name="simulation_alphaTarget" href="#simulation_alphaTarget">#</a> <i>simulation</i>.<b>alphaTarget</b>([<i>target</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L104 "Source")

如果指定了 *target* 则将当前的目标 [*alpha*](#simulation_alpha) 设置为指定的值，需要在 [0, 1] 之间。如果没有指定 *target* 则返回当前默认的目标 *alpha* 值, 默认为 0.

<a name="simulation_velocityDecay" href="#simulation_velocityDecay">#</a> <i>simulation</i>.<b>velocityDecay</b>([<i>decay</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L108 "Source")

如果指定了 *decay* 则设置仿真的速度衰减系数并返回仿真，范围为 [0, 1]。如果没有指定 *decay* 则返回当前的速度衰减系数，默认为 0.4，衰减系数类似于阻力。每次 `tick` 结束后每个节点的速度都会乘以 1 - *decay* 以降低节点的运动速度。速度衰减系数与 [alpha decay rate](#simulation_alphaDecay) 类似，较低的衰减系数可以使得迭代次数更多，其布局结果也会更理性，但是可能会引起数值不稳定从而导致震荡。

<a name="simulation_force" href="#simulation_force">#</a> <i>simulation</i>.<b>force</b>(<i>name</i>[, <i>force</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L112 "Source")

如果指定了 *force* 则表示为仿真添加指定 *name* 的 [force(力学模型)](#forces) 并返回仿真。如果没有指定 *force* 则返回当前仿真的对应 *name* 的力模型，如果没有对应的 *name* 则返回 `undefined`. (默认情况下仿真没有任何力学模型，需要手动添加). 例如创建一个用来对图进行布局的仿真，可以如下:

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

返回距离 ⟨*x*,*y*⟩ 位置最近的节点，并可以指定搜索半径 *radius*. 如果没有指定 *radius* 则默认为无穷大。如果在指定的搜索区域内没有找到节点，则返回 `undefined`.

<a name="simulation_on" href="#simulation_on">#</a> <i>simulation</i>.<b>on</b>(<i>typenames</i>, [<i>listener</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L139 "Source")

如果指定了 *listener* 则将其指定的 *typenames* 的回调。如果对应的 *typenames* 已经存在事件监听器，则将其替换。如果 *listener* 为 `null` 则表示移除对应 *typenames* 的事件监听器。如果没有指定 *listener* 则返回第一个符合条件的 *typenams* 对应的事件监听器，当指定的事件触发时，每个回调都会被调用，回调中 `this` 指向仿真本省。

*typenames* 可以由多个由空格隔开的 *typename*。每个 *typename* 都由 *type* 和可选的 *name* 组成，用 (`.`) 连接。比如  `tick.foo` 和 `tick.bar`。也就是可以为同一种事件类型注册多个事件监听器。其中 *type* 必须为以下几种:

* `tick` - 仿真内部定时器每次 `tick` 之后.
* `end` - 当 *alpha* < [*alphaMin*](#simulation_alphaMin) 时仿真内部定时器停止.

需要注意的是，`tick` 事件在手动调用 [*simulation*.tick](#simulation_tick) 时不会执行。`tick` 事件只会被内部定时器调用用以模拟布局过程。如果需要个性化调整布局，应该在 [forces](#simulation_force) 中注册而不是在每次 `tick` 时修改节点位置。

参考 [*dispatch*.on](https://github.com/d3/d3-dispatch#dispatch_on) 获取更多详情。

### Forces

A *force* is simply a function that modifies nodes’ positions or velocities; in this context, a *force* can apply a classical physical force such as electrical charge or gravity, or it can resolve a geometric constraint, such as keeping nodes within a bounding box or keeping linked nodes a fixed distance apart. For example, a simple positioning force that moves nodes towards the origin ⟨0,0⟩ might be implemented as:

```js
function force(alpha) {
  for (var i = 0, n = nodes.length, node, k = alpha * 0.1; i < n; ++i) {
    node = nodes[i];
    node.vx -= node.x * k;
    node.vy -= node.y * k;
  }
}
```

Forces typically read the node’s current position ⟨*x*,*y*⟩ and then add to (or subtract from) the node’s velocity ⟨*vx*,*vy*⟩. However, forces may also “peek ahead” to the anticipated next position of the node, ⟨*x* + *vx*,*y* + *vy*⟩; this is necessary for resolving geometric constraints through [iterative relaxation](https://en.wikipedia.org/wiki/Relaxation_\(iterative_method\)). Forces may also modify the position directly, which is sometimes useful to avoid adding energy to the simulation, such as when recentering the simulation in the viewport.

Simulations typically compose multiple forces as desired. This module provides several for your enjoyment:

* [Centering](#centering)
* [Collision](#collision)
* [Links](#links)
* [Many-Body](#many-body)
* [Positioning](#positioning)

Forces may optionally implement [*force*.initialize](#force_initialize) to receive the simulation’s array of nodes.

<a name="_force" href="#_force">#</a> <i>force</i>(<i>alpha</i>) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L44 "Source")

Applies this force, optionally observing the specified *alpha*. Typically, the force is applied to the array of nodes previously passed to [*force*.initialize](#force_initialize), however, some forces may apply to a subset of nodes, or behave differently. For example, [d3.forceLink](#links) applies to the source and target of each link.

<a name="force_initialize" href="#force_initialize">#</a> <i>force</i>.<b>initialize</b>(<i>nodes</i>) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L71 "Source")

Assigns the array of *nodes* to this force. This method is called when a force is bound to a simulation via [*simulation*.force](#simulation_force) and when the simulation’s nodes change via [*simulation*.nodes](#simulation_nodes). A force may perform necessary work during initialization, such as evaluating per-node parameters, to avoid repeatedly performing work during each application of the force.

#### Centering

The centering force translates nodes uniformly so that the mean position of all nodes (the center of mass if all nodes have equal weight) is at the given position ⟨[*x*](#center_x),[*y*](#center_y)⟩. This force modifies the positions of nodes on each application; it does not modify velocities, as doing so would typically cause the nodes to overshoot and oscillate around the desired center. This force helps keeps nodes in the center of the viewport, and unlike the [positioning force](#positioning), it does not distort their relative positions.

<a name="forceCenter" href="#forceCenter">#</a> d3.<b>forceCenter</b>([<i>x</i>, <i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L1 "Source")

Creates a new centering force with the specified [*x*-](#center_x) and [*y*-](#center_y) coordinates. If *x* and *y* are not specified, they default to ⟨0,0⟩.

<a name="center_x" href="#center_x">#</a> <i>center</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L27 "Source")

If *x* is specified, sets the *x*-coordinate of the centering position to the specified number and returns this force. If *x* is not specified, returns the current *x*-coordinate, which defaults to zero.

<a name="center_y" href="#center_y">#</a> <i>center</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L31 "Source")

If *y* is specified, sets the *y*-coordinate of the centering position to the specified number and returns this force. If *y* is not specified, returns the current *y*-coordinate, which defaults to zero.

#### Collision

The collision force treats nodes as circles with a given [radius](#collide_radius), rather than points, and prevents nodes from overlapping. More formally, two nodes *a* and *b* are separated so that the distance between *a* and *b* is at least *radius*(*a*) + *radius*(*b*). To reduce jitter, this is by default a “soft” constraint with a configurable [strength](#collide_strength) and [iteration count](#collide_iterations).

<a name="forceCollide" href="#forceCollide">#</a> d3.<b>forceCollide</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js "Source")

Creates a new circle collision force with the specified [*radius*](#collide_radius). If *radius* is not specified, it defaults to the constant one for all nodes.

<a name="collide_radius" href="#collide_radius">#</a> <i>collide</i>.<b>radius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L86 "Source")

If *radius* is specified, sets the radius accessor to the specified number or function, re-evaluates the radius accessor for each node, and returns this force. If *radius* is not specified, returns the current radius accessor, which defaults to:

```js
function radius() {
  return 1;
}
```

The radius accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the radius of each node is only recomputed when the force is initialized or when this method is called with a new *radius*, and not on every application of the force.

<a name="collide_strength" href="#collide_strength">#</a> <i>collide</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L82 "Source")

If *strength* is specified, sets the force strength to the specified number in the range [0,1] and returns this force. If *strength* is not specified, returns the current strength which defaults to 0.7.

Overlapping nodes are resolved through iterative relaxation. For each node, the other nodes that are anticipated to overlap at the next tick (using the anticipated positions ⟨*x* + *vx*,*y* + *vy*⟩) are determined; the node’s velocity is then modified to push the node out of each overlapping node. The change in velocity is dampened by the force’s strength such that the resolution of simultaneous overlaps can be blended together to find a stable solution.

<a name="collide_iterations" href="#collide_iterations">#</a> <i>collide</i>.<b>iterations</b>([<i>iterations</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L78 "Source")

If *iterations* is specified, sets the number of iterations per application to the specified number and returns this force. If *iterations* is not specified, returns the current iteration count which defaults to 1. Increasing the number of iterations greatly increases the rigidity of the constraint and avoids partial overlap of nodes, but also increases the runtime cost to evaluate the force.

#### Links

The link force pushes linked nodes together or apart according to the desired [link distance](#link_distance). The strength of the force is proportional to the difference between the linked nodes’ distance and the target distance, similar to a spring force.

<a name="forceLink" href="#forceLink">#</a> d3.<b>forceLink</b>([<i>links</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js "Source")

Creates a new link force with the specified *links* and default parameters. If *links* is not specified, it defaults to the empty array.

<a name="link_links" href="#link_links">#</a> <i>link</i>.<b>links</b>([<i>links</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L92 "Source")

If *links* is specified, sets the array of links associated with this force, recomputes the [distance](#link_distance) and [strength](#link_strength) parameters for each link, and returns this force. If *links* is not specified, returns the current array of links, which defaults to the empty array.

Each link is an object with the following properties:

* `source` - the link’s source node; see [*simulation*.nodes](#simulation_nodes)
* `target` - the link’s target node; see [*simulation*.nodes](#simulation_nodes)
* `index` - the zero-based index into *links*, assigned by this method

For convenience, a link’s source and target properties may be initialized using numeric or string identifiers rather than object references; see [*link*.id](#link_id). When the link force is [initialized](#force_initialize) (or re-initialized, as when the nodes or links change), any *link*.source or *link*.target property which is *not* an object is replaced by an object reference to the corresponding *node* with the given identifier.

If the specified array of *links* is modified, such as when links are added to or removed from the simulation, this method must be called again with the new (or changed) array to notify the force of the change; the force does not make a defensive copy of the specified array.

<a name="link_id" href="#link_id">#</a> <i>link</i>.<b>id</b>([<i>id</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L96 "Source")

If *id* is specified, sets the node id accessor to the specified function and returns this force. If *id* is not specified, returns the current node id accessor, which defaults to the numeric *node*.index:

```js
function id(d) {
  return d.index;
}
```

The default id accessor allows each link’s source and target to be specified as a zero-based index into the [nodes](#simulation_nodes) array. For example:

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

Now consider a different id accessor that returns a string:

```js
function id(d) {
  return d.id;
}
```

With this accessor, you can use named sources and targets:

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

This is particularly useful when representing graphs in JSON, as JSON does not allow references. See [this example](http://bl.ocks.org/mbostock/f584aa36df54c451c94a9d0798caed35).

The id accessor is invoked for each node whenever the force is initialized, as when the [nodes](#simulation_nodes) or [links](#link_links) change, being passed the node and its zero-based index.

<a name="link_distance" href="#link_distance">#</a> <i>link</i>.<b>distance</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L108 "Source")

If *distance* is specified, sets the distance accessor to the specified number or function, re-evaluates the distance accessor for each link, and returns this force. If *distance* is not specified, returns the current distance accessor, which defaults to:

```js
function distance() {
  return 30;
}
```

The distance accessor is invoked for each [link](#link_links), being passed the *link* and its zero-based *index*. The resulting number is then stored internally, such that the distance of each link is only recomputed when the force is initialized or when this method is called with a new *distance*, and not on every application of the force.

<a name="link_strength" href="#link_strength">#</a> <i>link</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L104 "Source")

If *strength* is specified, sets the strength accessor to the specified number or function, re-evaluates the strength accessor for each link, and returns this force. If *strength* is not specified, returns the current strength accessor, which defaults to:

```js
function strength(link) {
  return 1 / Math.min(count(link.source), count(link.target));
}
```

Where *count*(*node*) is a function that returns the number of links with the given node as a source or target. This default was chosen because it automatically reduces the strength of links connected to heavily-connected nodes, improving stability.

The strength accessor is invoked for each [link](#link_links), being passed the *link* and its zero-based *index*. The resulting number is then stored internally, such that the strength of each link is only recomputed when the force is initialized or when this method is called with a new *strength*, and not on every application of the force.

<a name="link_iterations" href="#link_iterations">#</a> <i>link</i>.<b>iterations</b>([<i>iterations</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L100 "Source")

If *iterations* is specified, sets the number of iterations per application to the specified number and returns this force. If *iterations* is not specified, returns the current iteration count which defaults to 1. Increasing the number of iterations greatly increases the rigidity of the constraint and is useful for [complex structures such as lattices](http://bl.ocks.org/mbostock/1b64ec067fcfc51e7471d944f51f1611), but also increases the runtime cost to evaluate the force.

#### Many-Body

The many-body (or *n*-body) force applies mutually amongst all [nodes](#simulation_nodes). It can be used to simulate gravity (attraction) if the [strength](#manyBody_strength) is positive, or electrostatic charge (repulsion) if the strength is negative. This implementation uses quadtrees and the [Barnes–Hut approximation](https://en.wikipedia.org/wiki/Barnes–Hut_simulation) to greatly improve performance; the accuracy can be customized using the [theta](#manyBody_theta) parameter.

Unlike links, which only affect two linked nodes, the charge force is global: every node affects every other node, even if they are on disconnected subgraphs.

<a name="forceManyBody" href="#forceManyBody">#</a> d3.<b>forceManyBody</b>() [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js "Source")

Creates a new many-body force with the default parameters.

<a name="manyBody_strength" href="#manyBody_strength">#</a> <i>manyBody</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L97 "Source")

If *strength* is specified, sets the strength accessor to the specified number or function, re-evaluates the strength accessor for each node, and returns this force. A positive value causes nodes to attract each other, similar to gravity, while a negative value causes nodes to repel each other, similar to electrostatic charge. If *strength* is not specified, returns the current strength accessor, which defaults to:

```js
function strength() {
  return -30;
}
```

The strength accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the strength of each node is only recomputed when the force is initialized or when this method is called with a new *strength*, and not on every application of the force.

<a name="manyBody_theta" href="#manyBody_theta">#</a> <i>manyBody</i>.<b>theta</b>([<i>theta</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L109 "Source")

If *theta* is specified, sets the Barnes–Hut approximation criterion to the specified number and returns this force. If *theta* is not specified, returns the current value, which defaults to 0.9.

To accelerate computation, this force implements the [Barnes–Hut approximation](http://en.wikipedia.org/wiki/Barnes–Hut_simulation) which takes O(*n* log *n*) per application where *n* is the number of [nodes](#simulation_nodes). For each application, a [quadtree](https://github.com/d3/d3-quadtree) stores the current node positions; then for each node, the combined force of all other nodes on the given node is computed. For a cluster of nodes that is far away, the charge force can be approximated by treating the cluster as a single, larger node. The *theta* parameter determines the accuracy of the approximation: if the ratio *w* / *l* of the width *w* of the quadtree cell to the distance *l* from the node to the cell’s center of mass is less than *theta*, all nodes in the given cell are treated as a single node rather than individually.

<a name="manyBody_distanceMin" href="#manyBody_distanceMin">#</a> <i>manyBody</i>.<b>distanceMin</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L101 "Source")

If *distance* is specified, sets the minimum distance between nodes over which this force is considered. If *distance* is not specified, returns the current minimum distance, which defaults to 1. A minimum distance establishes an upper bound on the strength of the force between two nearby nodes, avoiding instability. In particular, it avoids an infinitely-strong force if two nodes are exactly coincident; in this case, the direction of the force is random.

<a name="manyBody_distanceMax" href="#manyBody_distanceMax">#</a> <i>manyBody</i>.<b>distanceMax</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L105 "Source")

If *distance* is specified, sets the maximum distance between nodes over which this force is considered. If *distance* is not specified, returns the current maximum distance, which defaults to infinity. Specifying a finite maximum distance improves performance and produces a more localized layout.

#### Positioning

The [*x*](#forceX)- and [*y*](#forceY)-positioning forces push nodes towards a desired position along the given dimension with a configurable strength. The [*radial*](#forceRadial) force is similar, except it pushes nodes towards the closest point on a given circle. The strength of the force is proportional to the one-dimensional distance between the node’s position and the target position. While these forces can be used to position individual nodes, they are intended primarily for global forces that apply to all (or most) nodes.

<a name="forceX" href="#forceX">#</a> d3.<b>forceX</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js "Source")

Creates a new positioning force along the *x*-axis towards the given position [*x*](#x_x). If *x* is not specified, it defaults to 0.

<a name="x_strength" href="#x_strength">#</a> <i>x</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js#L32 "Source")

If *strength* is specified, sets the strength accessor to the specified number or function, re-evaluates the strength accessor for each node, and returns this force. The *strength* determines how much to increment the node’s *x*-velocity: ([*x*](#x_x) - *node*.x) × *strength*. For example, a value of 0.1 indicates that the node should move a tenth of the way from its current *x*-position to the target *x*-position with each application. Higher values moves nodes more quickly to the target position, often at the expense of other forces or constraints. A value outside the range [0,1] is not recommended.

If *strength* is not specified, returns the current strength accessor, which defaults to:

```js
function strength() {
  return 0.1;
}
```

The strength accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the strength of each node is only recomputed when the force is initialized or when this method is called with a new *strength*, and not on every application of the force.

<a name="x_x" href="#x_x">#</a> <i>x</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js#L36 "Source")

If *x* is specified, sets the *x*-coordinate accessor to the specified number or function, re-evaluates the *x*-accessor for each node, and returns this force. If *x* is not specified, returns the current *x*-accessor, which defaults to:

```js
function x() {
  return 0;
}
```

The *x*-accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the target *x*-coordinate of each node is only recomputed when the force is initialized or when this method is called with a new *x*, and not on every application of the force.

<a name="forceY" href="#forceY">#</a> d3.<b>forceY</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js "Source")

Creates a new positioning force along the *y*-axis towards the given position [*y*](#y_y). If *y* is not specified, it defaults to 0.

<a name="y_strength" href="#y_strength">#</a> <i>y</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js#L32 "Source")

If *strength* is specified, sets the strength accessor to the specified number or function, re-evaluates the strength accessor for each node, and returns this force. The *strength* determines how much to increment the node’s *y*-velocity: ([*y*](#y_y) - *node*.y) × *strength*. For example, a value of 0.1 indicates that the node should move a tenth of the way from its current *y*-position to the target *y*-position with each application. Higher values moves nodes more quickly to the target position, often at the expense of other forces or constraints. A value outside the range [0,1] is not recommended.

If *strength* is not specified, returns the current strength accessor, which defaults to:

```js
function strength() {
  return 0.1;
}
```

The strength accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the strength of each node is only recomputed when the force is initialized or when this method is called with a new *strength*, and not on every application of the force.

<a name="y_y" href="#y_y">#</a> <i>y</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js#L36 "Source")

If *y* is specified, sets the *y*-coordinate accessor to the specified number or function, re-evaluates the *y*-accessor for each node, and returns this force. If *y* is not specified, returns the current *y*-accessor, which defaults to:

```js
function y() {
  return 0;
}
```

The *y*-accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the target *y*-coordinate of each node is only recomputed when the force is initialized or when this method is called with a new *y*, and not on every application of the force.

<a name="forceRadial" href="#forceRadial">#</a> d3.<b>forceRadial</b>(<i>radius</i>[, <i>x</i>][, <i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

[<img alt="Radial Force" src="https://raw.githubusercontent.com/d3/d3-force/master/img/radial.png" width="420" height="219">](https://bl.ocks.org/mbostock/cd98bf52e9067e26945edd95e8cf6ef9)

Creates a new positioning force towards a circle of the specified [*radius*](#radial_radius) centered at ⟨[*x*](#radial_x),[*y*](#radial_y)⟩. If *x* and *y* are not specified, they default to ⟨0,0⟩.

<a name="radial_strength" href="#radial_strength">#</a> <i>radial</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

If *strength* is specified, sets the strength accessor to the specified number or function, re-evaluates the strength accessor for each node, and returns this force. The *strength* determines how much to increment the node’s *x*- and *y*-velocity. For example, a value of 0.1 indicates that the node should move a tenth of the way from its current position to the closest point on the circle with each application. Higher values moves nodes more quickly to the target position, often at the expense of other forces or constraints. A value outside the range [0,1] is not recommended.

If *strength* is not specified, returns the current strength accessor, which defaults to:

```js
function strength() {
  return 0.1;
}
```

The strength accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the strength of each node is only recomputed when the force is initialized or when this method is called with a new *strength*, and not on every application of the force.

<a name="radial_radius" href="#radial_radius">#</a> <i>radial</i>.<b>radius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

If *radius* is specified, sets the circle *radius* to the specified number or function, re-evaluates the *radius* accessor for each node, and returns this force. If *radius* is not specified, returns the current *radius* accessor.

The *radius* accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the target radius of each node is only recomputed when the force is initialized or when this method is called with a new *radius*, and not on every application of the force.

<a name="radial_x" href="#radial_x">#</a> <i>radial</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

If *x* is specified, sets the *x*-coordinate of the circle center to the specified number and returns this force. If *x* is not specified, returns the current *x*-coordinate of the center, which defaults to zero.

<a name="radial_y" href="#radial_y">#</a> <i>radial</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

If *y* is specified, sets the *y*-coordinate of the circle center to the specified number and returns this force. If *y* is not specified, returns the current *y*-coordinate of the center, which defaults to zero.
