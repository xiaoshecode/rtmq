# Device库使用文档

## 1. Device库核心文件树

   ```powershell
	+---config //设备配置文件夹
	|   +---config.json
	|   \---confgig-example.json
	+---install //安装脚本，按照顺序运行
	|   +---0config.cmd
	|   +---1createCondaEnv.cmd
	|   \---device.yml //创建conda环境
	+---libs //设备库文件夹
	|   +---dcam
	|   +---dll
	|   +---fit
	|   +---fitModels
	|   +---RTMQ
	|   \---若干设备控制的python代码
	+---DeviceWeb //设备控制的前端网页
	+---log //日志文件夹
	+---repo //实验室使用到的设备整理仓库
	+---manual //一些简单的使用说明
	+---AllKill.cmd //关闭所有设备
	+---AllStart.cmd //开启所有设备
	+---CMOSViewer.cmd //CMOS查看器
	+---CMOSViewer.py //CMOSViewer控制脚本
	+---Device.py //设备控制脚本
	+---.gitignore
   ```

## 2.代码逻辑介绍
!!! quote ""
	如无特别注释，默认当前文件夹为`Device`文件夹，如无特殊需要请勿改动。

本仓库主要解决实验台子上设备分散，所有实验需要控制的设备均可以使用这套`Device`框架进行连接，管理和控制。并且将整个框架分解为前端和后端两个没有耦合的部分。

- 前端：使用Vue.js编写了前端网页，通过订阅MQTT的`topic`进行数据的获取和显示。前端网页可以在`DeviceWeb`文件夹中找到。
- 后端：使用了MQTT(轻量化物联网协议)可以使得一台设备同时连接多台电脑进行通信和控制。例如激光器和波长计可供多个实验台子同时进行使用。设备通过对应的dll文件进行控制，并在MQTT上`publish`相应的`topic`，其他设备可以通过订阅相应的`topic`进行控制。MQTT的使用可以参考[MQTT](https://mqtt.org/)。

## 3. 使用方法

1. 按照[DeviceInstall.md](./DeviceInstall.md)中的步骤安装好环境，可能有缺少的包需要`pip`手动安装下
2. 修改`./libs/MQTT.py`中的MQTT配置，修改为你电脑的IP地址和端口号
3. 修改`./DeviceWeb/js/config.js`中的MQTT配置，修改为你电脑的IP地址和端口号
4. 将运行中的设备文件放在`./run`目录下，注意文件名和`config.json`中的topic一致
5. 修改配置文件`config/config.json`，添加需要的设备和对应的文件路径。`config.json` 规定了程序和`topic`之间的对应关系。它的文件结构为：
	```json
		{
			"Device": "./Device.py",
			"CMOS": "./run/CMOS.py",
			"CCD": "./run/CCD.py", 
		}
	```
6. 运行`run.cmd`
	```powershell
		title Device Console
		conda activate device && python Device.py
	```
7. 启动设备
      - 运行`AllStart.cmd`，启动所有设备
      - 参考`./manual/CMOS.ipynb`，使用jupyter notebook中启动单个设备
8. 操作设备
      - 参考`./manual/CMOS.ipynb`，使用jupyter notebook操作设备
      - 使用前端网页进行操作
9. 关闭设备，参考步骤7

