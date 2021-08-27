[TOC]


# 7.6

了解grpc中关于以太网通信的部分

了解redis中关于网络并行优化的部分

# 7.7

## 进行仿真编译

bazel build //common/tools/simulation:simulation_main

## mviz and replay

### 开始进行replay调试

#### 从服务器端复制mviz代码

./mviz.py yueyang07 --record_name 20210706_144006_s20-029 --download_start_time 20210706154133 --download_time_duration 30

7.7似乎mviz文件并没有.py后缀

--no_lidar --no_image
不下载雷达和图片

#### 实际使用的代码

./bazel-bin/tools/suite/mviz yueyang07 --record_name 20210706_144006_s20-029 --download_start_time 20210706154133 --download_time_duration 30 --no_lidar --no_image

#### 错误输出

Load record info success!
map_version===hualikan_v3.0.7.r
Run: 'python3 scripts/map_downloader.py --debug --map_version hualikan_v3.0.7.r'
ERROR: download map, Detail: [Errno 13] Permission denied: '/autocar'.
Failed to download map data from S3
[Warning] Faild to download map, use replay default map dir.
[ERROR] Faild to download map, please check.

#### 解决方式

1. 手动下载地图数据
2. 可以直接使用sudo

### 手动下载地图数据

python3 scripts/map_downloader.py --debug --map_version hualikan_v3.0.7.r

#### 错误输出

ERROR: download map, Detail: [Errno 13] Permission denied: '/autocar'.
Failed to download map data from S3

#### 解决方式（错误，不能使用sudo）
添加sudo权限

sudo python3 scripts/map_downloader.py --debug --map_version hualikan_v3.0.7.r

#### 正确输出

Location: hualikan_v3.0.7.r, File Path: /autocar/data/map/hualikan_v3.0.7.r

### 开始使用replay

下载过地图到默认路径后就可以不使用下面这一行了，否则需要手动指定路径

--map_dir=/autocar/data/map/hualikan_v3.0.7.r

sudo ./bazel-bin/tools/suite/mviz yueyang07 --record_name 20210706_144006_s20-029 --download_start_time 20210706154133  --download_time_duration 30 --no_lidar --no_image  --use_replay

#### 错误输出

Current Timestamp: 20210706153416.299This application failed to start because it could not find or load the Qt platform plugin "xcb"
in "".

Reinstalling the application may fix this problem.
*** SIGABRT received at time=1625642131 ***
PC: @     0x7f8820c45fb7  (unknown)  raise
    @          0x1d0583d        144  absl::lts_2019_08_08::WriteFailureInfo()
    @          0x1d05617         64  absl::lts_2019_08_08::AbslFailureSignalHandler()
    @     0x7f888ed16980  1007226672  (unknown)
    @          0x448bf50  (unknown)  (unknown)
    @     0x7f88000000e8  (unknown)  (unknown)
Aborted (core dumped)

#### 解决方式

1. 失败修改~/.bashrc
2. 使用cli解决

### 修改~/.bashrc（无效）

在~/.bashrc添加

export QT_PLUGIN_PATH=/opt/qt5/plugins

### 使用cli解决

./bazel-bin/common/tools/simulation/simulation_main --record_dir .mad_cache/20210706/144006-s20-029  --headless=true --append_rerun_node=Dreamview

#### 错误输出

rd_dir .mad_cache/20210706/144006-s20-029  --headless=true --append_rerun_node=Dreamview
E0707 15:26:55.314716  7187 onboard_util.cc:188] Dump flags to file: /tmp/simulationmain.flags
E0707 15:26:55.315845  7187 file_util.cc:103] Failed change file permission: /tmp/simulationmain.flags: Operation not permitted [1]
F0707 15:26:55.315863  7187 onboard_util.cc:189] Check failed: DumpGflagsToFile(export_file) Failed to dump gflags to file: /tmp/simulationmain.flags
*** Check failure stack trace: ***
*** SIGABRT received at time=1625642815 ***
PC: @     0x7f0e83fc7fb7  (unknown)  raise
    @          0x1d0583d        144  absl::lts_2019_08_08::WriteFailureInfo()
    @          0x1d05617         64  absl::lts_2019_08_08::AbslFailureSignalHandler()
    @     0x7f0ef2098980  1413933616  (unknown)
    @          0x231eccf        176  google::LogMessage::SendToLog()
    @          0x231ef43         48  google::LogMessage::Flush()
    @          0x2320e3f         32  google::LogMessageFatal::~LogMessageFatal()
    @          0x1cac5b3        800  walle::InitOnboard()
    @           0xa4ddab         96  main
    @     0x7f0e83faabf7  (unknown)  __libc_start_main
    @ 0xcbd6258d4c544155  (unknown)  (unknown)
已放弃 (核心已转储)

#### 添加sudo

sudo ./bazel-bin/common/tools/simulation/simulation_main --record_dir .mad_cache/20210706/144006-s20-029  --headless=true --append_rerun_node=Dreamview

#### 错误输出

E0707 15:37:00.420341  7835 calibration_param_manager.cc:84] Failed to load config from Invalid path: "config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt"
E0707 15:37:00.420390  7835 calibration_param_manager.h:76] Failed to load proto from config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt
E0707 15:37:00.537622  7835 onboard_system.cc:88] Node LidarTransform registered comms
E0707 15:37:00.537714  7835 simulation_module_factory.cc:85] Create Rerun for node: Dreamview
truncating vsnprintf buffer: [,]
truncating vsnprintf buffer: [,]
terminate called after throwing an instance of 'CivetException'
  what():  null context when constructing CivetServer. Possible problem binding to port.
*** SIGABRT received at time=1625643420 ***
PC: @     0x7f01d0fbafb7  (unknown)  raise
    @          0x1d0583d        144  absl::lts_2019_08_08::WriteFailureInfo()
    @          0x1d05617         64  absl::lts_2019_08_08::AbslFailureSignalHandler()
    @     0x7f023f08b980  (unknown)  (unknown)
    @ ... and at least 2 more frames
已放弃

#### 解决方式

处理端口重复绑定问题，这一般是单进程多进程混调的时候出现的问题

### 处理端口重复绑定问题（未解决问题）

sudo netstat -ntpl 

查看占用端口

### 正常方法

--headless=true是没有输出

在确认了问题的大致输出后，可以下载指定范围内的完整数据

./bazel-bin/tools/suite/mviz yueyang07 --record_name 20191226_143835_mkz-00 --full_image --download_start_time 20191226144050 --download_time_duration 60

需要显示图像的话添加 --use_replay 否则可以删掉。这一段跑仿真

./bazel-bin/tools/suite/mviz your_mis_id -ones 123456 --use_replay -- --append_rerun_node=Planning --log_dir=xx

#### 正常运行

./bazel-bin/tools/suite/mviz yueyang07 --car_name ZCX-25 --record_start_time 20210706165059 --record_end_time 20210706165119 --use_replay

#### 错误输出

Download dir:  .mad_cache 
{'carName': 'ZCX-25', 'startTime': 1625561459000, 'fileTypes': '', 'modules': 'CameraThumbnail,Canbus,Control,Dreamview,GnssDriver,LidarLeft,LidarPacketCollector,LidarRight,LidarMain,Localization,Matching,Perception,Planning,Prediction,Routing,TrafficLight,OnboardMonitor,ContiRadar,ReferenceLine,SemanticSegmenter', 'endTime': 1625561479000}
record_name: 20210706_160838_zcx-25
0M [00:00, ?M/s]
0M [00:00, ?M/s]
Load record info success!
map_version===zhonglihai_v3.1.6.r
Run: 'python3 scripts/map_downloader.py --debug --map_version zhonglihai_v3.1.6.r'
ERROR: download map, Detail: [Errno 13] Permission denied: '/autocar/data/map/zhonglihai_v3.1.6.r'.
Failed to download map data from S3
[Warning] Faild to download map, use replay default map dir.
[ERROR] Faild to download map, please check.

#### 解决方法

sudo chown yueyang:yueyang /autocar/

#### 错误输出

Current Timestamp: 20210706164655.076E0707 17:43:14.184896 14689 file_util.cc:44] Failed open file "/home/yueyang/.mad/hmi_config.pb.txt" for read.: 没有那个文件或目录 [2]
E0707 17:43:14.184922 14689 hmi_config_helper.cc:156] Failed to load hmiconfig from /home/yueyang/.mad/hmi_config.pb.txt
E0707 17:43:14.184927 14689 hmi_config_helper.cc:157] Load default config instead.
E0707 17:43:14.430930 14689 calibration_param_manager.cc:84] Failed to load config from Invalid path: "config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt"
E0707 17:43:14.430979 14689 calibration_param_manager.h:76] Failed to load proto from config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt
E0707 17:43:14.436908 14689 record_data_progressbar.cc:141] Record 20210706_160838_zcx-25 time range 20210706160853.000, 20210706180149.000
E0707 17:43:14.437018 14689 float_progressbar_widget.cc:34] Record 20210706_160838_zcx-25 time range 20210706160853.000, 20210706180149.000
E0707 17:43:15.030836 14689 road_graph_loader.cc:43] Loading Roadgraph ....
E0707 17:43:15.277067 14689 road_graph_loader.cc:55] Loading Roadgraph completed.
E0707 17:43:15.277271 14689 routing_component.cc:296] Doesn't have read privilege to file: /autocar/data/map/zhonglihai_v3.1.6.r/temp_routing
Current Timestamp: 20210706165059.014E0707 17:43:16.521209 14689 context.cc:45] GL version = 3.3
libpng warning: iCCP: known incorrect sRGB profile
libpng warning: iCCP: known incorrect sRGB profile
F0707 17:43:16.537868 14722 device_utils.h:21] Check failed: cudaSuccess == cudaStreamCreateWithFlags(&stream_, 0x01) (0 vs. 2) no error.
*** Check failure stack trace: ***
*** SIGABRT received at time=1625650997 ***
PC: @     0x7fa752494fb7  (unknown)  raise
    @          0x1d0583d        144  absl::lts_2019_08_08::WriteFailureInfo()
    @          0x1d05617         64  absl::lts_2019_08_08::AbslFailureSignalHandler()
    @     0x7fa7c0565980       3920  (unknown)
    @          0x231eccf        176  google::LogMessage::SendToLog()
    @          0x231ef43         48  google::LogMessage::Flush()
    @          0x2320e3f         32  google::LogMessageFatal::~LogMessageFatal()
    @           0xfd7a51         64  cuda::DeviceStream::DeviceStream()
    @          0x1cfe9bb        112  walle::JpegCodec::JpegCodec()
    @          0x1ce60b8         48  std::_Function_handler<>::_M_invoke()Current Timestamp: 20210706164655.076E0707 17:43:14.184896 14689 file_util.cc:44] Failed open file "/home/yueyang/.mad/hmi_config.pb.txt" for read.: 没有那个文件或目录 [2]
E0707 17:43:14.184922 14689 hmi_config_helper.cc:156] Failed to load hmiconfig from /home/yueyang/.mad/hmi_config.pb.txt
E0707 17:43:14.184927 14689 hmi_config_helper.cc:157] Load default config instead.
E0707 17:43:14.430930 14689 calibration_param_manager.cc:84] Failed to load config from Invalid path: "config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt"
E0707 17:43:14.430979 14689 calibration_param_manager.h:76] Failed to load proto from config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt
E0707 17:43:14.436908 14689 record_data_progressbar.cc:141] Record 20210706_160838_zcx-25 time range 20210706160853.000, 20210706180149.000
E0707 17:43:14.437018 14689 float_progressbar_widget.cc:34] Record 20210706_160838_zcx-25 time range 20210706160853.000, 20210706180149.000
E0707 17:43:15.030836 14689 road_graph_loader.cc:43] Loading Roadgraph ....
E0707 17:43:15.277067 14689 road_graph_loader.cc:55] Loading Roadgraph completed.
E0707 17:43:15.277271 14689 routing_component.cc:296] Doesn't have read privilege to file: /autocar/data/map/zhonglihai_v3.1.6.r/temp_routing
Current Timestamp: 20210706165059.014E0707 17:43:16.521209 14689 context.cc:45] GL version = 3.3
libpng warning: iCCP: known incorrect sRGB profile
libpng warning: iCCP: known incorrect sRGB profile
F0707 17:43:16.537868 14722 device_utils.h:21] Check failed: cudaSuccess == cudaStreamCreateWithFlags(&stream_, 0x01) (0 vs. 2) no error.
*** Check failure stack trace: ***
*** SIGABRT received at time=1625650997 ***
PC: @     0x7fa752494fb7  (unknown)  raise
    @          0x1d0583d        144  absl::lts_2019_08_08::WriteFailureInfo()
    @          0x1d05617         64  absl::lts_2019_08_08::AbslFailureSignalHandler()
    @     0x7fa7c0565980       3920  (unknown)
    @          0x231eccf        176  google::LogMessage::SendToLog()
    @          0x231ef43         48  google::LogMessage::Flush()
    @          0x2320e3f         32  google::LogMessageFatal::~LogMessageFatal()
    @           0xfd7a51         64  cuda::DeviceStream::DeviceStream()
    @          0x1cfe9bb        112  walle::JpegCodec::JpegCodec()
    @          0x1ce60b8         48  std::_Function_handler<>::_M_invoke()
    @          0x1d4fd7e         64  walle::EventCodecFactory::Create()
    @          0x1d42bd5         32  walle::OnboardContextBase::CreateEventCodec()
    @          0x1c84d93        160  walle::simulation::RecordDataRepublishModule::CreateCodec()
    @          0x1c857ee        144  walle::simulation::RecordDataRepublishModule::CreateCameraCodecIfNeeded()
    @          0x1c8549a        224  walle::simulation::RecordDataRepublishModule::PublishMessageRoutine()
    @          0x1c85ab5        240  walle::simulation::RecordDataRepublishModule::IterationRoutine()
    @          0x1cebd4e         48  walle::Module::OneIteration()
    @          0x1c7bd7c        192  walle::simulation::ConditionalScheduler::NextIteration()
    @          0x1c7ebe2         96  std::thread::_State_impl<>::_M_run()
    @     0x7fa7d9a366df  (unknown)  (unknown)
F0707 17:43:17.435711 14689 hmi_context.cc:123] Check failed: ::base::Status::OK() == (context->gl_context()->CheckError()) (OK vs. UNKNOWN: OpenGL Error - GL_INVALID_OPERATION) The HMIComponent 'VehicleMeshComponent' initialized failed.
*** Check failure stack trace: ***
[failure_signal_handler.cc : 316] RAW: Signal 6 raised at PC=0x7fa752494fb7 while already in AbslFailureSignalHandler()
    @          0x1d4fd7e         64  walle::EventCodecFactory::Create()
    @          0x1d42bd5         32  walle::OnboardContextBase::CreateEventCodec()
    @          0x1c84d93        160  walle::simulation::RecordDataRepublishModule::CreateCodec()
    @          0x1c857ee        144  walle::simulation::RecordDataRepublishModule::CreateCameraCodecIfNeeded()
    @          0x1c8549a        224  walle::simulation::RecordDataRepublishModule::PublishMessageRoutine()
    @          0x1c85ab5        240  walle::simulation::RecordDataRepublishModule::IterationRoutine()
    @          0x1cebd4e         48  walle::Module::OneIteration()
    @          0x1c7bd7c        192  walle::simulation::ConditionalScheduler::NextIteration()
    @          0x1c7ebe2         96  std::thread::_State_impl<>::_M_run()
    @     0x7fa7d9a366df  (unknown)  (unknown)
F0707 17:43:17.435711 14689 hmi_context.cc:123] Check failed: ::base::Status::OK() == (context->gl_context()->CheckError()) (OK vs. UNKNOWN: OpenGL Error - GL_INVALID_OPERATION) The HMIComponent 'VehicleMeshComponent' initialized failed.
*** Check failure stack trace: ***
[failure_signal_handler.cc : 316] RAW: Signal 6 raised at PC=0x7fa752494fb7 while already in AbslFailureSignalHandler()

## 7.8

### 尝试仿真

./bazel-bin/tools/suite/mviz yueyang07 --car_name ZCX-25 --record_start_time 20210706165059 --record_end_time 20210706165119

#### 错误输出

E0708 13:20:38.156193 30876 record_util.cc:77] Update map info for onboard config from gflags, previous : zhonglihai_v3.1.6.r, current: zhonglihai_v3.1.6.r
E0708 13:20:41.955396 30876 node_serial_file_reader.cc:183] No file for pattern: .mad_cache/20210706/160838-zcx-25/rec/LidarTransform.*..rec
E0708 13:20:41.963706 30876 node_serial_file_reader.cc:280] No index data
E0708 13:20:42.211184 30876 calibration_param_manager.cc:84] Failed to load config from Invalid path: "config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt"
E0708 13:20:42.211249 30876 calibration_param_manager.h:76] Failed to load proto from config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt
E0708 13:20:42.334326 30876 node_serial_file_reader.cc:183] No file for pattern: .mad_cache/20210706/160838-zcx-25/rec/Regression.*..rec
E0708 13:20:42.334492 30876 frame_view_app.cc:520] Set start time: 2021-07-06 16:50:59.000
E0708 13:20:42.339118 30876 node_serial_file_reader.cc:280] No index data
Current Timestamp: 20210706165059.000E0708 13:20:42.422443 30876 file_util.cc:44] Failed open file "/home/yueyang/.mad/hmi_config.pb.txt" for read.: 没有那个文件或目录 [2]
E0708 13:20:42.422554 30876 hmi_config_helper.cc:156] Failed to load hmiconfig from /home/yueyang/.mad/hmi_config.pb.txt
E0708 13:20:42.422564 30876 hmi_config_helper.cc:157] Load default config instead.
E0708 13:20:42.664620 30876 calibration_param_manager.cc:84] Failed to load config from Invalid path: "config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt"
E0708 13:20:42.664800 30876 calibration_param_manager.h:76] Failed to load proto from config/${hierarchy_config}/calibration/GnssNovatel_extrinsic.pb.txt
E0708 13:20:42.671166 30876 record_data_progressbar.cc:141] Record 20210706_160838_zcx-25 time range 20210706160853.000, 20210706180149.000
E0708 13:20:42.671295 30876 float_progressbar_widget.cc:34] Record 20210706_160838_zcx-25 time range 20210706160853.000, 20210706180149.000
E0708 13:20:43.251786 30876 road_graph_loader.cc:43] Loading Roadgraph ....
E0708 13:20:43.505499 30876 road_graph_loader.cc:55] Loading Roadgraph completed.
E0708 13:20:43.505829 30876 routing_component.cc:296] Doesn't have read privilege to file: /autocar/data/map/zhonglihai_v3.1.6.r/temp_routing
Current Timestamp: 20210706165059.014E0708 13:20:44.719974 30876 context.cc:45] GL version = 3.3
Current Timestamp: 20210706165059.082libpng warning: iCCP: known incorrect sRGB profile
Current Timestamp: 20210706165059.099libpng warning: iCCP: known incorrect sRGB profile
Current Timestamp: 20210706164655.076E0708 13:20:45.152261 30884 frame_view_app.cc:510] Failed to get regression before 2021-07-06 16:50:59.101
Current Timestamp: 20210706165059.497F0708 13:20:45.638532 30876 hmi_context.cc:123] Check failed: ::base::Status::OK() == (context->gl_context()->CheckError()) (OK vs. UNKNOWN: OpenGL Error - GL_INVALID_OPERATION) The HMIComponent 'VehicleMeshComponent' initialized failed.
*** Check failure stack trace: ***
*** SIGABRT received at time=1625721645 ***
Current Timestamp: 20210706165059.501PC: @     0x7f99e37d3fb7  (unknown)  raise
Current Timestamp: 20210706165059.503    @           0xa999ad        144  absl::lts_2019_08_08::WriteFailureInfo()
Current Timestamp: 20210706165059.505    @           0xa99787         64  absl::lts_2019_08_08::AbslFailureSignalHandler()
    @     0x7f99e469a980  576660032  (unknown)
    @           0xad18bf        176  google::LogMessage::SendToLog()
Current Timestamp: 20210706165059.509    @           0xad1b33         48  google::LogMessage::Flush()
    @           0xad358f         32  google::LogMessageFatal::~LogMessageFatal()
Current Timestamp: 20210706165059.510    @           0x77d512        160  walle::hmi::HMIContext::OnRenderInitialize()
Current Timestamp: 20210706165059.511    @           0x7993cc         48  walle::hmi::RenderDelegateManager::OnRenderInitialize()
Current Timestamp: 20210706165059.512    @           0x79a95f        160  walle::hmi::RenderWidget::Initialize()
Current Timestamp: 20210706165059.512    @     0x7f9a41d8626d  (unknown)  (unknown)
Current Timestamp: 20210706165059.512    @         0x447b3c30  (unknown)  (unknown)
    @     0x7f9a42575b40  (unknown)  (unknown)
Current Timestamp: 20210706165059.513    @ 0xfff601b0e9057400  (unknown)  (unknown)
Aborted (core dumped)

## 7.9

### 网络并行加速

#### 阅读redis代码

Nagle算法

### 未来安排

#### 第3周

使用grpc实现阻塞单线程版本的grpc_channel，做一个unit test，单机运行

##### ddl

周三之前，完成设计文档

#### 第4周

实现异步非阻塞版本，进行压力测试

#### 第5周

batch send等优化

#### 第6周

分析性能瓶颈，并进行优化

### 需要编译的文件

bazel build //tools/mviz:mviz_main

## 7.12

### 在学校远程实习

#### 阅读grpc代码

## 7.13

### 阅读grpc代码，配置vscode环境

#### clang-format设置

https://km.sankuai.com/page/530157931

#### grpc的quick start

https://grpc.io/docs/languages/cpp/quickstart/

## 7.14

### 配置docker

https://km.sankuai.com/page/249950675

bash docker/scripts/autocar/dev_into.sh

如果不行

bash docker/scripts/autocar/dev_start.sh

bash docker/scripts/autocar/dev_into.sh

### 与mentor交流

#### grpc_demo

http://dev.sankuai.com/code/repo-detail/walle/autocar/commit/004310fa85544327067e12fc1971e687949f2072?branch=refs%2Fheads%2Fdistributed_autocar_based_1.37

modules/distribution/ 文件夹

### 设计文档

https://km.sankuai.com/page/971752939

## 7.15



### c++知识

#### std::make_unique

std::make_unique<A>(xxx)；其中xxx为A的构造函数

#### std::thread

std::thread(lambda)，可以直接使用lambda表达式生成thread

#### lambda表达式

```cpp
[capture list] (params list) mutable exception-> return type { function body }
```

各项具体含义如下

1. capture list：捕获外部变量列表
2. params list：形参列表
3. mutable指示符：用来说用是否可以修改捕获的变量
4. exception：异常设定
5. return type：返回类型
6. function body：函数体

sort(myvec.begin(), myvec.end(), cmp); // 旧式做法

sort(lbvec.begin(), lbvec.end(), [](int a, int b) -> bool { return a < b; });   *// Lambda表达式*

https://blog.csdn.net/lcalqf/article/details/79401210

### mutable

用于在const成员中使用可变对象

https://www.cnblogs.com/yongdaimi/p/9565996.html

## 7.16

### gtest

todo

### grpc error

error message : failed to connect to all addresses

不能直接使用localhost，要用127.0.0.1

## 7.19

### std::promise

### std::future

### git reset 

单个文件：取消add

某个commit：取消本地commit

--mixed：取消前面的commit和add，大概率需要-f强制push

https://www.jianshu.com/p/c2ec5f06cf1a

### thread使用方法

join()，阻塞当前线程，等待当前线程执行完毕

detach()，不阻塞当前线程

### explicit关键字

在写一个类的时候，如果一个构造器是接收单个参数的，那么最好要加上explicit。
如果不加的话，该构造函数还会拥有类型转换的情形，造成混淆。

解决办法：该构造函数的声明前加上explicit。

### std::ref std::cref

传递引用和常值引用

## 7.20

### 完成gtest

## 7.21

## 7.22

shadow memory

batch

## 7.23

autocar/common/event_record/[forwardonly_event_reader.cc](http://forwardonly_event_reader.cc/)

## 7.26

### 函数传参

std::function<void(int,bool)>

std::bind

```cpp
struct Foo {
    void print_sum(int n1, int n2) {
        std::cout << n1+n2 << '\n';
    }
    int data = 10;
};
int main() {
    Foo foo;
    auto f = std::bind(&Foo::print_sum, &foo, 95, std::placeholders::_1);
    f(5); // 100
}
```

https://www.jianshu.com/p/f191e88dcc80

### 存储位置

.mad_cache/20210725/082232-s20-027

## 7.27

### ISO C++11 does not allow conversion from string literal to 'char *const'

用char const* tmp = "dsfsfs"

./bazel-bin/common/framework/grpc_channel_benchmark first

yueyang@yueyang-G5-5500:~/Desktop/autocar$ ping git.sankuai.com
PING git.sankuai.com (10.4.14.24) 56(84) bytes of data.
^C
--- git.sankuai.com ping statistics ---
7 packets transmitted, 0 received, 100% packet loss, time 6132ms

yueyang@yueyang-G5-5500:~/Desktop/autocar$ sudo openconnect -u yueyang07 https://sslvpn.sankuai.com
[sudo] yueyang 的密码： 
POST https://sslvpn.sankuai.com/
Connected to 103.37.140.40:443
SSL negotiation with sslvpn.sankuai.com
Connected to HTTPS on sslvpn.sankuai.com
XML POST enabled
1.请选择正确的Group登录，请勿选择dx-vpn。

2.正式员工推荐使用大象“一键VPN”登录。
Please enter your username and password.
GROUP: [common-vpn|duanxin|dx-vpn|international]:yueyang07
认证选择“yueyang07”不可用
GROUP: [common-vpn|duanxin|dx-vpn|international]:duanxin
POST https://sslvpn.sankuai.com/
XML POST enabled
1.请选择正确的Group登录，请勿选择dx-vpn。

2.正式员工推荐使用大象“一键VPN”登录。
Please enter your username and password.
Password:
POST https://sslvpn.sankuai.com/
Response:
POST https://sslvpn.sankuai.com/
收到CONNECT响应：HTTP/1.1 200 OK
CSTP 已连接。DPD 30, 保持连接 20
Connected as 11.3.148.97, using SSL
Established DTLS connection (using GnuTLS). Ciphersuite (DTLS0.9)-(DHE-RSA-4294967237)-(AES-256-CBC)-(SHA1).
DTLS got write error: Error in the push function.. Falling back to SSL
