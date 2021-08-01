

# 基于MobileNetV3的人脸表情捕捉-加速

## 加速库 onnx/tensorRT 说明
1. onnx/tensorRT都是nVidia的开源加速库，到目前（2021.7.12）为止，它们只能加速inference，不能加速training。
2. onnx支持CPU或GPU。到目前（2021.7.12）为止，tensorRT仅支持GPU。
3. tensorRT加速原理：操作合并+降低精度。[详情](https://www.cnblogs.com/wujianming-110117/p/12983582.html)

## 环境（硬件+软件）
1. CPU
    - 硬件 Intel(R) Core(TM) i7-9700 CPU @ 3.00GHz   3.00 GHz
    - 系统 Windows 10 专业版 64位
    - 依赖包
        * python 3.7.5
        * torch     	1.9.0
        * torchvision	0.10.0
        * onnxruntime	1.7.0
    - 对应代码目录：LLFR_Online_Infer_cpu

2. GPU
    - 硬件 NVIDIA Tesla T4, 显存 8G
    - 系统 CentOS Linux release 7.2 (tlinux 2.2 64bit)
    - 驱动 
        * CUDA Version: 11.0
        * cuDNN 7.6.5
    - 依赖包
        * Python 3.7.10
        * torch        1.9.0
        * torchvision  0.10.0
        * onnxruntime  1.8.0
        * tensorrt     7.1.3.4
    - 对应代码目录：LLFR_Online_Infer_GPU


## 安装
### 编译环境
- 批处理 `sh install.sh`

### FBX
1. 打开[链接](https://www.autodesk.com/developer-network/platform-technologies/fbx-sdk-2020-2)，
    下载对应平台的版本（win,linux,mac），解压。

2. 将解压文件的目录加入路径，在`fbxloader.py`里注明。
    - 对于GPU，`fbxloader.py`在`/LLFR_Online_Infer_GPU/code/FBX_utils/fbxloader.py`里。
    - 对于CPU，同理

### tensorRT
1. 下载 **不同系统+cuda+cuDNN** 对应的版本
    - 以 **CentOS7+cuda11+cuDNN8** 为例，
    链接是[这里](https://developer.download.nvidia.cn/compute/machine-learning/tensorrt/secure/7.1/tars/TensorRT-7.1.3.4.CentOS-7.6.x86_64-gnu.cuda-11.0.cudnn8.0.tar.gz?mEkuTUaTr_RSV3NMYYw3hZ20W9vtzUedvAnASZteHqFc3Xoc_3YejNJEtaQFICnwTj0rYMsZ3hW-iVoQWybRzWENFz36BFpGS-uEwOaDeYwMNELRVtDUojnEPyz-FDGIFW-HXxTVSPuD-sgmdKHdJp2SgO2HCo_QJQ2um_bqyjNJ_cMowkJG4bHdGHdBd9PYvYRfNcr89aBmCwJjfdRv5wTNadJ5OdUfGVKzy_LI5ezA)
    - 说明：tensorRT仅支持GPU（截止2021.7.12）
    
2. 命令
    - `tar xzvf TensorRT-7.0.0.11.CentOS-7.6.x86_64-gnu.cuda-10.2.cudnn7.6.tar.gz`
    - `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/root/ft_local/tensorRT/TensorRT-7.0.0.11/lib`
    - `cd python`
    - `pip install tensorrt-*-cp3x-none-linux_x86_64.whl`
    - `cd TensorRT-${version}/graphsurgeon`
    - `sudo pip3 install graphsurgeon-0.4.5-py2.py3-none-any.whl`

3. 环境变量
    - `vim ~/.bashrc`
    - `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/root/env/tensorRT/TensorRT-7.1.3.4/lib`
    - `source ~/.bashrc`


## 运行
1. 测试onnx,tensorRT对faceBox和llfr的加速情况
    - 测速时
        - `python demo.py   --check-face=no   --id 4   --h 480 --w 270  --input-path=test_clip.mp4`
    - 验证人脸识别有没有受到影响，保存图片和视频
        - `python demo.py   --check-face=yes   --id 4   --h 480 --w 270  --input-path=test_clip.mp4`
    - 验证加速之后，是否影响重建性能
        - `--checkMSE=yes`
    - 修改输出维度
        - `--num-class=57`

    - 更多命令见        
        - `sh experimentGPU.sh`
		
	- 注意事项
		- 如果提示`timeStamp没有声明就引用`，则说明人脸没有被识别到。调换`--h`和`--w`顺序试一试。
		
2. 其它
    * 摄像头实时捕捉
        `python demo.py --mode=webcam`
    * 同时保存视频与fbx动画
        `python demo.py --mode=webcam --save-video=True --save-fbx=True`
    * 视频输入
        `python demo.py --mode=video --input-path=benchmark/benchmark_0.mp4`
    * 同时保存转码视频与fbx动画
        `python demo.py --mode=video --input-path=xxx.mp4 --save-video=True --save-fbx=True`
    * 图片输入(暂时未实现)
        `python demo.py --mode=image --input-path=xxx.mp4`
