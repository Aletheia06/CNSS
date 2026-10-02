# 异构计算环境
这部分其实我很早以前就部署过，过程也比较简单。首先是通过powershell下载wsl，因为可以直连Nvidia显卡(这部分我记得之前非常折腾，最开始我用的是VMware，那玩意根本不能直连显卡，后来才知道WSL可以)。
使用wsl --install即可安装wsl，选择Ubuntu22.04安装，我设置主机名为HPC  
然后在powershell里用wsl -d HPC指定主机名就行  
接下来执行nvidia-smi  
```
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 610.51                 KMD Version: 610.71        CUDA UMD Version: 13.3     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 5060 ...    On  |   00000000:01:00.0 Off |                  N/A |
| N/A   58C    P2             11W /   60W |     447MiB /   8151MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

可以看到这样的界面，即说明已经可以识别到NVIDIA的显卡了，我这里是5060  
然后需要安装NVIDIA CUDA的软件源。我是通过这两条指令完成的。
```
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
```

然后sudo apt update  

接下来需要安装CUDA Toolkit，因为WSL提供了cuda-toolkit-13-3的软件包，所以直接安装，运行  
```
sudo apt install cuda-toolkit-13-3
```
就行了  
我们还需要配置CUDA的环境变量，所以用这三条指令  
```
echo 'export PATH=/usr/local/cuda-13.3/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-13.3/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc

source ~/.bashrc
```

这时候检查一下nvcc能不能用，我们试一下  
```
nvcc --version
```

发现没有问题，得到了  
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2026 NVIDIA Corporation
Built on Tue_Jun_09_02:43:40_PM_PDT_2026
Cuda compilation tools, release 13.3, V13.3.73
Build cuda_13.3.r13.3/compiler.38244171_0
```

这里环境部分就配置好了  

# pytorch环境部署

我们首先需要python环境，用pip安装python，我现在已经安装好了  
然后需要安装pytorch。我们先进入python的虚拟环境中。我发现之前没有venv，所以要运行  
```
sudo apt install python3-venv
```
然后就是老生常谈的两步  
```
python3 -m venv pytorch-env
source pytorch-env/bin/activate
```
接下来用安装pytorch的CUDA版本
非常大，要等好久  

嗯，后来发现下载不动了，只好换镜像
```
pip install torch torchvision \
  -i https://pypi.tuna.tsinghua.edu.cn/simple \
  --extra-index-url https://mirrors.tuna.tsinghua.edu.cn/pytorch-wheels/cu132
```

好了，检查一下
```
(pytorch-env) liang@LAPTOP-Liang:~$ python3 -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"
True
NVIDIA GeForce RTX 5060 Laptop GPU
```

没问题了，pytorch安装完毕。  
最后复制题目给的代码进去，运行  
```
(pytorch-env) liang@LAPTOP-Liang:~/CUDAPractice$ python3 test.py
CUDA Available: True
Hello
HPC!
```
