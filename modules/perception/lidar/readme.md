<a name="lidar_module" />

## lidar

## app
app目录中主要实现两个功能，分别为 lidar_obstacle_detection 和 lidar_obstacle_tracking，并被构建为lib。
### lidar_obstacle_detection 
此lib共有继承自 BaseLidarObstacleDetection 类，声明并实现了检测程序的初始化函数以及数据处理接口。

在源文件中，Init 方法以及 Process 方法都有两套函数签名，分别用于不同的调用情况。
第一套签名对应利用配置参数特别配置的初始化和数据处理过程，第二套签名对应 Pipeline 的配置和调用过程。

#### Init
初始化函数有两种实现方法，第一种是输入 LidarObstacleDetectionInitOptions 的对象引用进行初始化操作，另一种是输入 pipeline_config 的对象引用进行初始化操作。

其初始化的内容包括：
1. 激光雷达障碍物检测器的初始化；
2. 场景管理器的初始化（如果使能）；
3. 地图管理器的初始化（如果使能）。

TODO: 场景管理器的作用？
TODO: 地图管理器的作用？

#### Process
数据处理函数有三种实现方法，可以分为两类。一类是只输入点云数据，调用 pipeline::InnerProcess 函数处理它，另一类是输入点云数据的同时输入选项信息，调用 ProcessCommon 函数处理它。

其数据处理的内容包括：
1. 点云的前处理（如果需要）；
2. 更新地图信息；
3. 检测目标；
4. 基于检测目标创建障碍物目标（如果使能）；
5. 使用目标滤波器过滤障碍物目标（如果使能）。

使用 InnerProcess 与使用 ProcessCommon 在功能性上没有区别，只不过在处理过程的管理上有区别。Pipeline 情况是前提是将激光雷达障碍物检测的各个阶段都配置为 pipeline 中的一个个 stage 那么就直接调用 InnerProcess 进行处理。如果没有，则需要使用 ProcessCommon 进行处理。

### lidar_obstacle_tracking
此 lib 公有继承自 BaseLidarObstacleTracking 类，声明并实现了跟踪的初始化函数以及数据处理接口。

在源文件中，Init 方法以及 Process 方法都有两套函数签名，分别用于不同的调用情况。同上。

#### Init
初始化函数有两种实现方法，第一种是输入 LidarObstacleTrackingInitOptions 的对象引用进行初始化操作，另一种是输入 PipelineConfig 的对象引用进行初始化操作。

其初始化的内容包括：
1. 创建并初始化多目标跟踪器；
2. 创建并初始化分类器。

#### Process
数据处理函数有两种实现方法，一类是只输入 data_frame，调用 pipeline::InnerProcess 函数处理它，另一类是输入 frame 的同时输入选项信息，调用 ProcessCommon 函数处理它。

其数据处理的内容包括：
1. 障碍物目标的跟踪；
2. 障碍物目标的分类。

## lib目录
lib 目录存储了基于激光雷达感知的需要使用的核心运算库，每个库一个文件夹，作为一个单位编译。

#### classifier
classifier库中定义了五个库，分别为 util ， type_fusion_interface， ccrf_type_fusion， fuse_classifier， fused_classifier_test。

util库为需要使用的帮助函数。

type_fusion_interface 声明了 BaseOneShotTypeFusion 和 BaseSequenceTypeFusion 两个虚类，分别为基于单帧数据进行类别融合的类以及基于序列数据进行类别融合的类。类别融合的具体实现需要在 fused_classifier.cc 和 ccrf_type_fusion.cc 中定义。

ccrf_type_fusion 是基于条件随机场的类型融合库，继承自 BaseOneShotTypeFusion 和 BaseSequenceTypeFusion 分别实现了 CCRFOneShotTypeFusion 和 CCRFSequenceTypeFusion 两个类。
根据目前获得的信息[issue 13726](https://github.com/ApolloAuto/apollo/issues/13726)，当分类器可以给目标多种分类及其概率的时候，会使用类 CCRFOneShotTypeFusion 对类别进行处理，当前在 apollo 代码中，此类仅配合 cnnseg 使用。


fused_classifer 继承自 BaseClassifier，利用前面定义的类型融合类，实现具体的感知目标 object 类别的融合。其中在其 Classify 方法中，存在下面的代码：

```c++
    if (object->lidar_supplement.is_background) {//如果目标是背景目标，则认为它属于 UNKNOWN_UNMOVABLE 类，概率为1
        object->type_probs.assign(static_cast<int>(ObjectType::MAX_OBJECT_TYPE),
                                  0);
        object->type = ObjectType::UNKNOWN_UNMOVABLE;
        object->type_probs[static_cast<int>(ObjectType::UNKNOWN_UNMOVABLE)] =
            1.0;
        continue;
      }
```
该代码的作用是判断检测阶段给目标赋予的类别是否为**背景目标**，如果是，则该目标的融合后类型为 UNKNOW_UNMOVABLE （未知不可移动目标） 概率为1。


#### ground_detector


#### map_manager


#### object_builder


#### object_filter_bank


#### pointcloud_preprocessor


#### roi_filter
感兴趣区域过滤

#### scene_manager
场景管理？？？

#### segmentation
分割

#### tracker
追踪

## tools
tools目录主要有2个工具，一个是OfflineLidarObstaclePerception，另一个是msg_exporter_main，下面分别介绍这2个工具的作用。  

