# 数据格式与类，以及缓存机制

OpenScene 是对大规模 [nuPlan 数据集](https://motional-nuplan.s3.ap-northeast-1.amazonaws.com/index.html) 的压缩再发布版本，只保留了与 NAVSIM 相关的 2Hz 传感器数据与标注，这使得数据规模减少了 10 倍以上。  
在 NAVSIM 中，这些数据被组织为 `navsim.common.dataclasses.Scene` 对象。  
一个 `Scene` 是由多个 `Frame` 组成的列表，每个 `Frame` 包含训练规划 `Agent` 所需的输入与标注。

**缓存（Caching）。**  
在评估规划器时，需要对原始标注数据进行大量预处理，包括在每个 `Frame` 上查询全局地图并转换到局部坐标系。  
你可以通过如下命令生成缓存：

```bash
cd $NAVSIM_DEVKIT_ROOT/scripts/
./run_metric_caching.sh
```

该脚本会在 `$NAVSIM_EXP_ROOT/metric_cache` 下创建指标缓存，其中 `$NAVSIM_EXP_ROOT` 是你在安装过程中通过环境变量设置的实验根目录。


