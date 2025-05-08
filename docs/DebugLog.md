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

### 406-xwjRTMQdebug记录-20250325

1. `sequencer.sweep(Seq(Wave).Detection(1000,1).Wait(10),"C0")`中的`Wait(10)`需要时间稍长一些，否则Detection的数据来不及传输到电脑上
2. `plot`使用mqtt传输数据，在设置内置数据库时默认匿名登录，不需要设置用户名和密码，如果添加用户名和密码会导致无法连接，需要在`config.cmd`中设置
3. `plot`无输出数据时可以检查`Exp.run(seq)`是否有返回值

### 414-hhyRTMQdebug记录-20250508

1. 修复了`config.ipynb`修改后不能正常加载的错误
2. 删除了关于`config.db`的相关操作，现在不通过`config.db`来加载配置而是直接在`Exp.ipynb`中加载`config.py`中的配置