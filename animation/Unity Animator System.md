##### Aniation Clip  
可以来源于外部导入（捕捉，3D软件制作），也可以来源于内部创建（创建组件的时间线动画）。

---
##### Animator  
将模型和Controller和Avatar结合起来的Component。  
Update Mode：  
1. 	Normal：动画更新和Update同步，动画时间受timescale影响。  
2. 	Animate Physics：动画更新和FixedUpdate同步，适用于动画和physics交互，例如可推动的rigidbody对象。  
3. 	Unscaled Time：和Normal不同的是会忽略timescale影响，例如需要忽略暂停的GUI动画。

Culling Mode：
1. Always Animate：始终处理动画。
2. Cull Update Transforms：renderers不可见时会禁用Retarget, IK 和Transforms写入。
3. Cull Completely：renderers不可见时全部禁止。


---

##### 外部导入的动画  
动画对象和动画数据可以在一个文件，也可以分开放置，导入的动画是不能编辑的，如果需要编辑可以创建一个动画剪辑将关键帧复制过来进行修改。

---

##### Humanoid Avatars
类人纸娃娃系统提供了不同人形生物之间动画的映射，以便重用动画数据。

---

##### 动画编辑器快捷键  
， 转到上一帧。  
。 转到下一帧。  
Alt+， 转至上一个关键帧。  
Alt键+ 转到下一个关键帧。  
A 适合显示所有关键帧
F 适合显示选择的关键帧


---

##### Animator Controller  
使用多个参数变量来控制动画状态机。
支持Bool，Int，Float3种类型和Trigger触发。  
Trigger和Bool的区别在于触发器在触发状态切换后会自动重置。

---

##### State
状态可以关联一段动画剪辑或者Blend Tree，并可以通过一组条件控制如何转换到其他状态。  
可以通过Entry节点来指定默认状态，也可以设置多个条件来选择初始状态。  

Any State节点可以用来实现只要达成条件就可以从任意状态跳转。  
> 由于Any State也包含当前状态，可能触发自身循环切换，可以通过取消勾选Settings-Can Transition To Self来解决。 

状态转换条件同时达成时按Transitions列表里的顺序决定。  


Foot IK：激活脚的IK，只适用于humanoid人形动画。  
Write Defaults：在播放某一个动画Clip时，对于当前Clip没有涉及到的属性（而其他Clip修改过此属性），如果勾选了Write Defaults则使用默认值（即初始状态）。反之使用上一个动画状态结束时修改过的值。  
Mirror：镜像动作，只适用人形动画。
Mute可以关闭选中的条件转换，Solo可以只处理选中的转换，同时选中Mute优先。  

---
##### State Transition

如果为转换启用了Has Exit Time，并且有一个或多个条件，则仅在状态的退出时间之后才检查这些条件。这可以确保转换只发生在动画的特定部分。
Exit Time是一个归一值，表示在百分之多少时间后启用转换条件，对于循环动画，小于1的值每个循环都生效，大于1的值只在指定的循环数后生效一次。

Fixed Duration：转换持续时间是归一化值还是秒数。  
Transition Offset：在转换为目标状态时的播放偏移量。  

Interruption Source：一般来说State在Transition的时候是无法打断的，也可以通过设置允许进行打断。 

1. None：默认不允许打断。    
2. Current：当前状态的转换列表里的转换如果符合条件允许打断。  
3. Next：目标状态的转换列表里的转换如果符合条件允许打断。  
4. Current State Then Next State：当前状态优先于目标状态。  
5. Next State Then Current State：目标状态优先于当前状态。  

> 打断的转换会保留当前已转换的动画状态，直接向新的目标状态融合。  

Ordered Interruption：在允许打断的情况下，如果勾选则只包含排在当前转换前边的，不勾选则包括全部（同时满足条件始终会按列表优先级进行触发）  




---

##### StateMachineBehaviour  
状态上还可以附加Behaviour脚本，包括以下预定义行为： 
```
OnStateMachineEnter：过渡到状态机时在第一个Update帧调用。  
OnStateMachineExit：退出状态机时的最后一个Update帧调用。   
参数：（Animator animator，int stateMachinePathHash）
过渡到状态机子状态时不调用,必须通过Entry节点才会调用，
```


```
OnStateEnter：进入状态的Update帧调用。  
OnStateExit：退出状态的Update帧调用。
OnStateIK：在MonoBehaviour.OnAnimatorIK之后立即调用。  
OnStateMove：在MonoBehaviour.OnAnimatorMove之后立即调用。 
OnStateUpdate：除去进入和离开的其他Update帧调用。  
参数：（Animator animator, AnimatorStateInfo stateInfo, int layerIndex）  
```

>   默认情况会实例化每个行为脚本，除非指定[SharedBetweenAnimators]标签来共享行为脚本，这时脚本的成员变量可能会影响所有实例。


---

##### Sub-state Machine  
如果状态过于复杂，可以合并一些状态为子状态机来简化结构。  
包含Up名称的状态节点表示跳转到父状态机。  

---

##### Animation Layers  
可以混合身体不同部位的不同动画，通过AwatarMask来指定要混合的骨骼，Override方式会丢弃其他层的动画信息，而Additive则会将动画叠加。  
Sync功能允许同步复制其他Layer的状态连接并使用不同的动画剪辑。
Timing功能如果勾选，由权重决定调整同步层中每个动画所需的时间，如果未勾选，则同步层上的动画将拉伸到原始层上动画的长度。  

---

##### Target Matching  
可以用Animator.MatchTarget将指定的骨骼移动到指定的位置，比如跳石头或抓栏杆。

---

##### Inverse Kinematics  
反向运动学可以通过 SetIKPositionWeight, SetIKRotationWeight, SetIKPosition, SetIKRotation, SetLookAtPosition, bodyPosition, bodyRotation等函数来反向影响骨骼。  
使用Layer的IK Pass来激活功能。


---
##### Root Motion

Body Transform是角色的重心，用于Mecanim系统的重定向引擎（映射动画到不同模型）中来提供稳定的模型移动。   
Orientation是角色模型在T姿势下上身和下身朝向的平均值。  
Body Transform和Orientation存储在Animation Clip中，这两个是世界空间的曲线，其他的动画曲线都是以相对body transform的形式存储的。  

Root Transform是body transform在Y平面上的投影，并且是运行时计算的。每一帧Root Transform的变化实时计算。然后Transform的变化会被应用到GameObject上从而让物体移动。  

通过对Animation Clip的设置来控制Body Transform投影到Root Transform的结果。
可以修改以下3个：  

###### 1. Root Transform Rotation：  
用于设置Root Transform的朝向（旋转）。  
Bake into Pose：选中后，角色的朝向会基于body transform（Pose）。Root Orientation会是一个常量，意味着Animation Clip不会旋转这个物体。  
> 只有AnimationClip的开始和结束位置旋转相似的时候，才应该使用这个选项。可以通过右边的绿灯判断。通常用于向前直行的走或跑的动画。  

Base Upon：可以设置动画的朝向基于的地方。  
- Body Orientation：动画会朝向身体正前方。这个设置适用于大多数身体朝前的动画，比如走跑跳。但是如果动画是向左或向右平移的话，会有问题。- 这时候可以使用下面的Offset来调节角色的朝向。
- Original：有的动画师会给动画手动加上旋转，确保动画的朝向正确，这时候可以使用这个选项，一般就不用再手动调整Offset了。
Offset：基于Base Upon的设置，调整偏移量。


###### 2. Root Transform Position (Y) :  
用于设置Root Transform位置的Y轴位置。

Bake into Pose：选中后，动画的Y轴的运动会保留在Body Transform（Pose）上。Root Transform的Y轴会是一个常数（不会受动画影响变化），也就是意味着动画不会改变物体位置的Y值。右边有一个绿灯指示动画起始位置和结束位置的高度是否一致，可以看出动画是否适合使用此选项。  

大多数的动画应该选中此选项，除了那些会改变物体高度的动画比如跳起、跳下这些动画。  

> 注意：Animator.gravityWeight是由Bake Into Pose position Y控制的。选中时gravityWeight = 1，不选中时gravityWeight = 0。gravityWeight用来在state转换时进行混合。  

Base Upon：和Root Transform Rotation设置类似，除了Original 或 Mass Center (Body)选项外，还有一个Feet选项。Feet选项非常适合改变物体高度的动画（不勾选Bake Into Pose）。使用Feet时，Root Transform Position Y会匹配骨骼中脚部的Y位置（更低的那个）。Feet选项可以避免混合或转换时浮空的现象。

Offset：可以设置高度的偏移量。

###### 3. Root Transform Position (XZ)：   
用于设置Root Transform位置的XZ轴位置。  

Bake Into Pose：通常用于原地不动的动画（动画在XZ轴上的位置为0）。可以用来去除动画循环累计的误差，造成位置的移动。也可以通过设置Based Upon Original来强制使用动画师设置的位置，否则会使用角色的重心作为Root。


---

##### Blend Trees  
将多个动作使用混合阈值平滑的混合起来，比如根据速度切换跑步走路，或是转向左右倾斜。   
###### 混合类型：  
1D：通过一个参数来混合动画。  
2D Simple Directional: 适合不同方向的混合动画，例如“向前走、向后走、向左走、向右走”，还可以选择在位置(0，0)处的单个动作例如“空闲”。不适合同一个方向上的多个动作，例如“向前走”和“向前跑”。  
2D Freeform Directional: 也适合不同方向的混合动画，但是可以在同一个方向上有多个动作，例如“向前走”和“向前跑”。运动集应该始终包含位置(0，0)的单个运动，例如“普通状态”。  
2D Freeform Cartesian: 适合不代表方向的混合动画。X参数和Y参数可以表示不同的概念，例如角速度和线速度。例如“向前不转弯”、“向前向右转”等运动。  
Direct:这种混合树使用参数直接控制多个动画片段的权重，适用于面部形状或随机闲置混合。


Compute Thresholds：可以使用Root Transform的值自动计算阈值（1D使用需要取消“Automate Thresholds”）。  


---

##### Blend Shapes
导出时勾选相关选项，就会在SkinnedMeshRenderer里看到相关数据，可以在Animation里进行动画变换编辑，也可以使用代码GetBlendShapeWeight和SetBlendShapeWeight来控制融合。


---

##### Animator Override Controllers  
创建基于另外Controller的Asset，可以共用状态机并替换不同的动画剪辑。

---

##### Performance and optimization
1. 播放简单动画时legacy animation system更加高效。  
2. 缩放动画曲线比位移和旋转更昂贵，尽量避免使用（不包括值没有变化的曲线）。  
3. 导入Humanoid animation时可以使用Avatar Mask来移除IK Goals或手指动画。  
4. 使用Generic animation时使用root motion会更昂贵，如果确定不使用则不要指定root bone。  
5. 使用哈希码代替String来检索动画。
6. 设置Animator的Culling Mode为Based on Renderers，并且禁用skinned mesh renderer的Update When Offscreen。

