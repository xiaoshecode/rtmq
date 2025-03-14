# RTMQ ReadMe——IIIS

!!! reference
   [安装视频及简单使用教程](https://cloud.tsinghua.edu.cn/d/708864f7fd554fc8bd6a/)
## 硬件连接

插上电源，连接机箱，将USB插到电脑上

## 软件安装

step1：基本环境

1. anaconda2023.3（python3.10）

   - 将conda添加到系统环境中，安装过程可以参考[Anaconda安装在Windows](https://www.cnblogs.com/ajianbeyourself/p/17654155.html)

   - 终端中输入`conda info`，得到安装信息，证明安装成功

2. vscode安装

step2：conda环境

- `conda install jupyter`

step3：拷贝代码包

1. 代码包获取
   - TIQC清华云盘——[基于2023.3公司的版本](https://cloud.tsinghua.edu.cn/f/edc55be8e81647059545/)，现主要用于412
   - 406代码版本——xiaoshe

step4：代码包调试使用

1. 文件树说明

   ```powershell
   +---2024 //运行代码文件夹
   |   +---202409
   |   \---202410
   +---cdn //cdn环境目录
   |   +---ajax
   |   \---npm
   +---drv //芯片驱动，不需要修改
   |   \---FTD3XXDriver_WHQLCertified_v1.3.0.4
   +---js //JavaScript文件，用于前端网页显示
   +---log //log日志，jupyter自动生成
   +---md //使用说明文档
   +---py //程序python库
   +---vue //前端网页显示
   ```

2. 添加`jupyter_config.json`到用户文件夹下

   - 例如`C://user//xiaoshe//.jupyter`
   - 检查jupyter端口

辅助软件

1. powertoys:[Install PowerToys | Microsoft Learn](https://learn.microsoft.com/en-us/windows/powertoys/install) ，用于Windows置顶窗口,若干有用的功能自行探索
1. DeskPins，用于Windows置顶窗口

## 调试debug