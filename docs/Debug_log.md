# 调试记录

## 调试log

### 414RTMQdebug记录-20250314

1. 槽位从左到右为1 2 3 4 0 5 6 7 8，对应Exp.py的`rwg_slots = [4,5,6,7]`，在414平台上最右边板卡没有被用到，其中0代表master板卡
2. 修改dds通道对应`config.ipynb`需要同时在`config.ipynb`和`Exp.py`中的`gpio_dir`里面施加魔法，暂未习得，勿加改动， 只有改PMT输入通道才需要改`gpio_dir`，其他修改DDS通道或者TTL通道不需要改，直接使用`config.py`里面定义好的通道
3. ttl定义里面`Wave().ttl[0]([4])`中的ttl[0]代表在第一段时序里面，而非通道含义

### 210RTMQdebug记录-20250319

1. `config.ipynb`中修改结束后，需要使用`Exp.Init()`才生效
2. `Exp.Init()`之后机箱输出均归0，DDS/Wave这些数据结构里的数值都还在 运行下一个时序的时候就会生效
3. 机箱当前输出的信号是由sequencer.scan/sweep（底层是Exp.Run）写入的最后一次序列的最后一个Wave里面的内容决定的