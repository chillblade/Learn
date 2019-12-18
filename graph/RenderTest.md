渲染管线主要包含三个测试，分别是Alpha,Stencil和Depth。

============================================================
1.Alpha

处理透明有两种方式：
------------------------------------------------------------
Alpha Test：
要么符合条件完全不透明，要么不符合条件，完全透明


在Unity中使用以下语句进行测试
clip (o.Alpha - _Cutoff);
clip的参数如果小于0则调用discard放弃该fragment

也可以使用Pass标签AlphaTest来使用
Greater  点的Alpha值大于 X时渲染,          
GEuqal   点的Alpha值大于等于 X时渲染,
Equal    点的Alpha值等于 X时渲染,
NotEqual 点的Alpha值不等于X时渲染,
Less     点的Alpha值小于 X时渲染,
LEqual   点的Alpha值小于等于 X时渲染,
Always   全渲染,
Never    全不渲染

AlphaTest Greater _Cutoff

------------------------------------------------------------
Alpha Blending：
使用指定的算法来混合ColorBuffer中的颜色值。
首先需要关闭ZWrite，如果不关闭就会导致半透明物体背后的物体无法通过ZTest。
还需要设置为Transparent队列，来保证半透明物体比Opaque物体后处理，避免因为没有ZWrite而被更远的Opaque物体覆盖。
另外Unity还会对所有半透明物体进行排序，保证是从远到近渲染，不然没有ZWrite无法保证半透明物体之间的正确覆盖，但是如果遇到互相交错的情况也没法得到正确的结果。

混合方式
Blend Off  关闭混合
BlendOp Operation 混合操作(默认操作为Add)
Blend SrcFactor DstFactor  指定分别和来源颜色和目标颜色相乘的混合因子
Blend SrcFacter DstFactor, SrcFactorA  DstFactorA  同上，不过单独指定了用于Alpha分量的混合因子


混合操作
Add 将来源颜色和目标颜色相加
Sub 用来源颜色减去目标颜色
RevSub 用目标颜色减去来源颜色
Min 取来源颜色和目标颜色中的最小值（逐分量比较）
Max 取来源颜色和目标颜色中的最大值（逐分量比较）
[ 来源指的是shader返回的颜色，目标指缓冲区颜色。]


混合因子
One	 1
Zero 0
SrcColor	源颜色值
SrcAlpha	源颜色的Alpha值
DstColor	目标颜色值
DstAlpha	目标颜色的Alpha值
OneMinusSrcColor	1-源颜色值
OneMinusSrcAlpha	1-源颜色的Alpha值
OneMinusDstColor	1-目标颜色值
OneMinusDstAlpha	1-目标颜色的Alpha值


------------------------------------------------------------
关于性能，Alpha Test虽然逻辑简单，但是并不会比Alpha Blending提升多少，在IOS和某些Android设备上，Alpha Test的性能消耗反而更大，所以除非特殊情况，还是建议使用Blending。


------------------------------------------------------------
关于渲染队列
Opaque队列的对象是从近到远处理
Transparent队列的对象是从远到近处理





