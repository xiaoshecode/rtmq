# Device config配置

由于存在特殊设备本身操作较为复杂，本处针对特殊设备（CMOS等）配置进行说明，帮助大家更好地使用设备，之后也会持续更新特殊设备的使用说明。

## CMOS

CMOS设备的配置文件为`./config/config.json`，其内容如下：

```json
{
    "Device": "./Device.py",
    "CMOS": "./run/CMOS.py",
}
```
其中`CMOS`对应的文件为`./config/CMOS.json`，该文件中定义了CMOS设备的相关参数和操作方法。
其中关于CMOS的参数定义如下：

```json
{
    "exposureTime": 500000,
    "intensity": [
        100,
        400
    ],
    "region": [
        748,
        1600,
        0,
        4096
    ],
    "root": "./data",
    "scale": [
        100,
        400
    ],
    "fit_params": {
        "sigma": 10,
        "threshold_rel": 0.2,
        "rlimit": [
            5,
            100
        ]
    }
}   
```

### CMOS参数说明
1. `exposureTime`：曝光时间
2. `intensity`：CMOS的强度，范围为0-1000，默认值为100-400
3. `region`：CMOS的区域，格式为[x1,x2,y1,y2]，表示CMOS的区域范围，默认值为[748,1600,0,4096]
4. `root`：数据存储的根目录，默认值为`./data`
5. `scale`：CMOS的缩放比例，范围为0-1000，默认值为100-400
6. `fit_params`：拟合参数，用来进行离子识别，修改后可以在`CMOSViewer`中实时更新，便于调参：
    - `sigma`：高斯拟合的标准差，默认值为10
    - `threshold_rel`：对比度，默认值为0.2
    - `rlimit`：离子半径，格式为[min,max]，默认值为[5,100]

### CMOSViewer使用和离子识别程序使用

1. `CMOSViewer`：用于实时查看CMOS的图像和离子识别结果，使用方法为：
    - 打开设备后会自动打开或者使用`CMOSViewer.cmd`命令打开
    - 在`CMOSViewer`中可以实时查看CMOS的图像和离子识别结果
2. `CMOSViewer`右侧的条可以拖动来调整对比度和亮度，可以更好地查看图
3. 使用的离子识别程序为`./libs/cvutilsdev.py`里面的`findIons`函数，源码如下：
```python
def findIons(
    image: np.ndarray,
    sigma: float = 1,
    threshold_rel: float = 0.3,
    rlimit: list = [0.5, 10],
    **kwargs,
) -> Union[float, Tuple[np.ndarray, np.ndarray]]:
    """find ions of input image

    Parameters
    ----------
    image : np.ndarray, 2dims or 3dims(will be averaged to 2dims along the first axis)
    verbose : bool, optional
        if true return (pks, radius), elese return (N) ion numbers only, by default True
    sigma : float, optional
        sigma used in gaussian_laplace, by default 2
    threshold_rel : float, optional
        the relative threshold to find ions, by default 0.3

    Returns
    -------
    Union[float, Tuple[np.ndarray, np.ndarray]]
        ion number or (pks, radius) of ions
    """
    # print('min r', min_r)
    if not (isinstance(image, np.ndarray) and image.ndim in [2, 3]):
        logger.error("Detector Wrong format of image")
        return False

    image = image.mean(0) if image.ndim == 3 else image
    img = gaussian_laplace(image, sigma=sigma, output=np.float32)
    img = np.where(1 - norm(img) > threshold_rel, 1, 0)
    img_label, ion_num = label(img, 0, True)

    pks, rds = np.empty((0, 2)), np.empty(0)
    if ion_num <= MAX_ION_NUM:
        props = regionprops(img_label, intensity_image=image)
        rds = np.array([prop.equivalent_diameter / 2 for prop in props])
        if len(rlimit) == 2 and rlimit[1] > rlimit[0] and rlimit[0] > 0:
            idx = np.logical_and(rds > rlimit[0], rds < rlimit[1])
        rds = rds[idx]
        if rds.size:
            pks = np.array([p.centroid_weighted for i, p in enumerate(props) if idx[i]])
            idx = np.lexsort((np.round(pks[:, 0]), np.round(pks[:, 1])))
            pks = pks[idx]
            rds = rds[idx]
    return pks, rds
```

## CCD
TODO

