# Device

!!! reference
   [明明学长的Device库](https://cloud.tsinghua.edu.cn/d/708864f7fd554fc8bd6a/)
   用来管理实验中用到的设备。

## 硬件连接

1. 检查当前电脑是否有相关的环境

   ```bash
   conda env list
   ```

   如果有`device` or `work` 等目录应该就是符合的

2. 如没有相关环境, 运行`./install/1createCondaEnv.cmd`  

​		会在当前的 `conda`环境中创建名为`device`的环境，并安装依赖。

# Configuration

## MQTT Configuration

根本平台的配置修改 `MQTT.py`， line 9-16

```python
if True:
    broker = '192.168.50.27'
    port = 1883
    username = 'streamsheets'
    password = 'vgI8g1z2Ya'
    JUPYTER = 'http://localhost:8888'
    WS = f'"ws://{broker}:8083/mqtt",'+'{'+f'username:"{username}",password:"{password}"'+'}'
    WEB = "http://localhost"
```

## Device Configuration

### 配置文件

我们通过配置文件来管理某个平台时候运行的程序，它放在`./config`。可以参考 `./config/config-example.json`创建`./config/config.json`  。

`config.json` 规定了程序和`topic`之间的对应关系。

的文件结构为：

```json
{
    "Device": "./Device.py"，
    "topic": "path_to_program",
}
```

其中 `Device` 字段为必须字段，其他字段可以按照新增的设备自行添加，如

```json
{
    "Device": "./Device.py",
    "CMOS": "./run/CMOS.py"    
}
```

上面，我们添加了topic:`CMOS` 对应的文件是 `./run/CMOS.py`

### 文件位置

1.  对于在该电脑上运行的程序(除了 Device.py 外)请放在`./run`目录下（该目录需要自行创建）

2. 相关的库文件请放在 `./libs`(注意库文件的通用性，如是设备specific,请放在`./run`)

3. `./repo`中放了已经开发好的程序，可以按需复制到`./run`中，配置使用

   

## Web  Configuration

web相关的项目请参考：https://gitlab.hyqubit.com/trappedionsoftware/DeviceWeb

# Run

1. `run.cmd` 启动 `Device.py` 可以对相应的设备进行远程管理

   

# Add New Device

1. 参考[Device Control](https://e3wkuiehjm.feishu.cn/docx/OG4fdgMOMovWoYxsHfocOW8dn3g?from=from_copylink)开发规范，开发新的设备，如 `xxx.py`
2. 将`xxx.py` 加入到 `config.json` 中
3. 开发相应的网页（if necessary)



# Device Management and Remote Call

## Device Management

对设备的管理主要为 start, kill, restart，这些可以通过如下接口实现。一下以设备 Toptica 为例

```python
import MQTT

client = MQTT.create()
manager = MQTT.caller("Device", client)

device_name = "Toptica"

# start the device
manager.start(device_name)

# kill the device
manager.kill(device_name)

# restart the device: kill the device first if existed then start the device
manager.kill(device_name, True)
```



## Remote Call

在设备已经在 device 中启动之后，我们可以对设备进行远程调用，方式如下：

```python
import MQTT

# init the caller
client = MQTT.create()
device_name = "Toptica"
device = MQTT.caller(device_name, client)

# call related functions
# get the value
device.Vset()

# set the value
device.Vset(100)
```

1. 远程调用的接口和 `Toptica.py`中定义的相关函数，obj是一致的，支持层级调用，如`device.a.b()`

2. 对于`attribute`的调用也需要以`()`结尾，如想获取属性`a`需要使用在 Remote  call 应使用`Device.a()`而不是`python`中的`Dervice.a`
3. 目前对于数据类型的支持是有限的，常见的`list, str, bool, None, np.nadarray` 都能支持，而`figure`之列特殊数据目前是不支持的。对于任意数据类型的支持目前已经开发完毕，后续会进行更新。