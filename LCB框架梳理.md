# LCB流程：（以Linker为核心）
一.Linker的初始化
1.首先在Activity，Fragment，Dialog创建时调用CreateLinker创建根Linker
2.CreateLinker会调用ViewBuilder的build方法创建Linker
3.build内部会：
调用createView创建Linker对应的View视图。
创建Linker对应的Controller实例。
创建Linker对应的Component实例。
通过构造函数new (view, controller, component)来实例化Linker。
二.Linker的依赖注入和附加
1.在Linker的构造函数内部调用component.inject(controller)，为controller进行注入，并为controller注入presenter
2.顺序调用Linker的attach方法，将Linker的View添加到根Linker里面：
Linker内部调用onAttach方法。
Controller调用attach方法。
Presenter调用didLoad方法。

根Linker：
位于应用的根层次结构中，通常由LCBActivity或LCBFragment直接创建。
负责管理整个应用或整个页面的主要视图和组件。
子Linker：
位于根Linker的下层次结构中，由根Linker或其他子Linker创建和管理。
负责管理局部视图和组件，是更细粒度的视图和逻辑管理单位。
三.Linker分离和销毁
在LCBActivity/LCBFragment生命周期的onDestroy或者onDestroyView方法中调用linker?.detach()方法：
Linker内部调用detach方法，移除所有子Linker并调用其detach方法，调用controller的detach方法。
Controller执行onDetach,Presenter执行dispatchUnload。
Controller发布Controller生命周期事件DETACH到lifecycleSubject。

LCB的生命周期：

LCB类图：
一.主要的类：
LCBActivity、LCBFragment、ViewBuilder、Linker、Controller和Presenter
二.各类基础功能：
LCBActivity:所有Activity都要继承这个类
createLinker()方法：在onCreate时被执行，用于创建ViewLinker。
LCBFragment:所有Fragment都要继承这个类
createLinker()方法：在onCreateView时被执行，用于创建ViewLinker。
ViewBuilder:负责创建View和ViewLinker。
inflateView(): 创建View。
build(): 创建Linker。
Linker
Linker:负责将Component注入到Controller中，管理视图和控制器的生命周期。
变量：
controller(C): Linker的controller。
component(D): Linker的注入器。
_children: 子节点的集合，用于管理子Linker。
方法：
onAttach(): 在Linker附加时调用。
onDetach(): 在Linker分离时调用。
Controller:负责处理具体的逻辑和事件。
变量：
presenter: 使用@Inject注解的延迟加载变量。
lifecycleSubject: 用于发布控制器生命周期事件的BehaviorSubject。
方法：
onAttach(): 在控制器附加时调用。
onDetach(): 在控制器分离时调用。
Presenter:实现了LifecycleScopeProvider接口，用于管理UI逻辑。
方法：
didLoad(): 在onAttach时调用,对标onAttach时机。
willUnload(): 在onLoad时调用，对标onLoad时机。
