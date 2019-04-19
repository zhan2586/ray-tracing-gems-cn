最重要的新函数，TraceRay(),生成一根射线。从逻辑上讲，这类似于纹理获取:它会暂停着色器以获取一个变量(而且可能是大量)的 GPU 时钟，当结果可用，为进行进一步处理，它会恢复执行。
射线生成，closest-hit 和 miss 着色器可以调用 TraceRay ()。这些着色器可以在每个线程中启动零条、一条或多条光线。
光线发射的基本代码如下:

```C++

1 RaytracingAccelerationStructure scene; // Scene BVH from C++
2 RayDesc ray = { rayOrigin, minHitDist, rayDirection, maxHitDist };
3 UserDefinedPayloadStruct payload = { ... <initialize here>... };
4
5 TraceRay( scene, RAY_FLAG_NONE, instancesToQuery, // What geometry?
6       hitGroup, numHitGroups, missShader, // Which shaders?
7       ray, // What ray to trace?
8       payload ); // What data to use?

```

用户定义的payload结构包含每根光线的数据持续到光线的生命周期。在遍历以及TraceRay()返回结果时，用它来维护光线的状态。DirectX定义了RayDesc用于存储光线的起点，方向，最小和最大的碰撞距离（有序打包成两个float4）。
超过规定的间距，光线相交就会被忽略。acceleration结构的定义源自主机API(查看3.8.1部分)。

第一个TraceRay()参数选择包含几何图形的BVH。简单光线跟踪器通常使用一个BVH，但是独立地查询多个结构可以允许不同的几何类(例如，透明/不透明、动态/静态)有不同的行为。第二个参数包含改变光线行为的标志，例如，指定对光线有效的附加优化。第三个参数是一个整数实例掩码，它允许基于每个实例位掩码跳过几何图形;这应该是0xFF来测试所有几何图形。

第四个和第五个参数帮助选择使用哪一个碰撞组。一个碰撞组包括相交，最近碰撞以及任何碰撞着色器（其中一些可能是空）。这些参数决定使用哪个集合，以及测试什么几何类型和BVH实例。对于基本光线跟踪器，每种光线类型通常有一个碰撞组:例如，主光线可能使用碰撞组0，阴影光线使用碰撞组1，全局光照光线使用碰撞组2。在这种情况下，第四个参数选择光线类型，第五个参数指定不同类型的数量。第六个参数指定使用哪个miss着色器。这只是索引到加载的miss着色器列表中。第七个参数是要跟踪的射线，第八个参数应该是该射线的用户定义的持久的payload结构