# Config README

## 1. 板卡通道映射关系

通过板卡通道的编号和配置文件名字来确定，如下的程序将名称为 Pumping， Cooling等DDS通道分别映射到数组内方便程序调用。
!!! note
    rwg_slot: 使用的板卡编号。板卡编号与板卡在机箱中的位置一致，从左至右依次为[1，2，3，4，0，5，6，7]
`pos=0`表示板卡按照rwg_slot的顺序从左至右依次编号，第0个输出通道。
```python
Global411 = DDS.Global411 = DDS(pos=0).f[0](135).a[0](1)
Global411.on = Global411().a[1](0.64)
Global411.off = Global411().a[1](0).f[1](60)  # .a[2](0).a[3](0).a[4](0)
Global411.rsblow = Global411().a[1](0.4)
Global411.rsbhigh = Global411().a[1](0.4)
Global411.bsblow = Global411().a[1](0.4)
Global411.bsbhigh = Global411().a[1](0.4)
Global411.shelf0 = Global411().a[1](0.64)
Global411.shelf1 = Global411().f[1](-8.54).a[1](0.64)
Global411.shelf2 = Global411().f[1](9.11).a[1](0.64)
Global411.test = Global411().a[1](0.54)
```

## 2. DDS

### 2.1 单音模式 && 多音模式
定义DDS，DDS接口的一个频率输出有三个可调的参数f,a,p(默认为0)。多音模式即一个通道可以同时输出多个频率的叠加，比较常用的是2个频率，在程序中定义如下：

```python 
LAOD = DDS.LAOD = DDS(pos=1).f[0](170.0).a[0](1)
LAOD.on = LAOD().f[1](0.3).a[1](0.35).f[2](-0.3).a[2](0.35)
LAOD.off = LAOD().f[1](-1).a[1](0.35).f[2](0.3).a[2](0.35)
```

这里定义了一个名为LAOD的DDS，它对应的物理板卡是pos=1，代表是第一个板卡上的第一个通道。
它的基频是170.0MHz，幅度是1，on状态下同时输出两个频率，分别是170+0.3和170-0.3MHz，幅度都是0.35。off状态下两个频率分别是170-1和170+1MHz，幅度都是0.35。


## 3. pluse(TTL)

类似于DDS，定义好名字和板卡连接即可。TTL只有打开和关闭两种状态，较dds设置简单。

```python
class TTL:
    Pumping = 0
    EOM3432 = 1
    Cooling = 2
    CCD = 4
    PMT = 15
```
这里定义的TTL类，将TTL编号与名字对应起来，方便程序调用，当然也可以直接调用编号，具体调用方法下面sequence说明里面给出示例。

## 4. Sequence

### 4.1 时序基本单元定义

我们将一组DDS输出和TTL信号的组合称为最小时序，其格式定义如下：

```python
Wave.Count = Wave()
    .dds[0](**DDS.Dict(Cooling.on,  AOM976.on, AOM3432.on, LAOM411.on, RAOM411.on))
    .ttl[0]([TTL.Cooling,TTL.PMT]).ttl[1]([TTL.Cooling])
Wave.Cooling = Wave()
    .dds[0](**DDS.Dict(Cooling.on,  AOM976.on, AOM3432.on, LAOM411.on,RAOM411on))
Wave.CoolingC = Wave()
    .dds[0](**DDS.Dict(Cooling.on,  AOM976.on, AOM3432.on))
    .ttl[0]([15]).dds[1](**DDS.Dict(Cooling.on,  AOM976.on, AOM3432.on))
```

以Wave.Count为例，定义了一个名为Count的时序，其中dds[0]表示第一段时序单元中使用的DDS通道，后面的参数是一个字典，表示这个时序单元开启的DDS状态，ttl[0]表示第一个TTL时序单元中开启的TTL通道，后面的参数是一个列表，表示使用的TTL。同理可以定义出第二段DDS和TTL的时序单元。

### 4.2 探测时序基本单元定义

```python
Wave.Detection = Wave()
    .dds[0](**DDS.Dict(Pumping.detection,AOM976.on))
    .ttl[0]([2,TTL.PMT]).dds[1](**DDS.Dict(AOM976.on))
    .ttl[1]([2])
```
一般在Detection序列中会使用两段时序单元，第一段是探测信号，第二段是探测完毕后的状态，其余的时序单元基本只有一段。

### 4.3. 时序的运行模式

!!! note
    这里的运行模式是指在实验中如何运行时序，如何进行扫描等。
    参考`data.py`中的代码，可以看到具体的运行模式。
    ```python
    CCD_TH = 200
    CCD_TH = 200
    PMT_TH = 1.5
    INIT_TH = 2
    def C0(arr): return arr  # arr.sum(1)
    def S0(arr): return arr > PMT_TH  # arr.sum(1)>1
    def P0(arr): return arr <= PMT_TH  # arr.sum(1)<=1
    def C1(arr): return arr
    def S1(arr): return arr > CCD_TH
    def SC(arr): return arr[arr[:,0]>INIT_TH,1]>PMT_TH
    ```

#### 4.3.1 Sweep
反复运行同一段时序，直到手动结束

```python
# sweep mode：sweep(self, seq, stat=None, repeats=100, res=None)
sequencer.sweep(seq, "C0")
```

#### 4.3.2 Scan
对系统中一个维度的参数（如时间、频率、相位、幅度等）进行扫描，得到一组以扫描参数为自变量的结果

```python
data,datafile = sequencer.scan(Seq(Wave).Detection(1000,1).Wait(sym.t),sym.t,(2000,3000,50))
# scan(self, seq, func, rng, stat=None, repeats=100, prefix='', res=None, plt=True):
```
参数说明：
func：需要进行扫描的参数，如果是对时间(sym.t)进行扫描，是整个整数n，表示对是时序中第n个位置的时间进行扫描（从0开始计数）。如果是其他参数，如`Cooling.s0.f`则表示对这个频率进行扫描

??? cite
    这里的func是一个函数，不能直接传入数字，必须是一个函数。对于时间扫描时，传入`sym.a，sym.b`之类的也可。`sym`是一套符号表达式系统，用来模拟Mathematica，属于是历史包袱。

rng:range, 三个参数分别是 start, stop, step
stat：同Sweep：计数相同
