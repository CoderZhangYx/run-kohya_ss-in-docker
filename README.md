# kohya_ss的docker环境搭建指北  

1.克隆项目
```
git clone -b v21.5.2 https://ghproxy.com/https://github.com/bmaltais/kohya_ss.git
cd kohya_ss
```

2.创建Dockerfile 
```
mkdir docker && cd docker  
touch Dockerfile
```
打开Dockerfile文件粘贴下面的内容
```
# pull docker imgage with specified cuda version
FROM nvidia/cuda:11.6.2-cudnn8-devel-ubuntu18.04
ENV TORCH_CUDA_ARCH_LIST="6.0 6.1 7.0 7.5 8.0 8.6+PTX" \
	TORCH_NVCC_FLAGS="-Xfatbin -compress-all" \
	CMAKE_PREFIX_PATH="$(dirname $(which conda))/../" \
	FORCE_CUDA="1"

# nvidia key settings
RUN rm /etc/apt/sources.list.d/cuda.list && \
	apt-key del 7fa2af80 && \
	apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub && \
	apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/7fa2af80.pub

# (Optional)specify apt source mirror
RUN sed -i 's/http:\/\/archive.ubuntu.com\/ubuntu\//http:\/\/mirrors.aliyun.com\/ubuntu\//g' /etc/apt/sources.list

# Install the required packages
RUN apt-get update
RUN apt-get install -y ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libsm6 libxrender-dev libxext6 
RUN apt-get install -y vim tmux wget

# (Optional)install support for chinese language 
# RUN apt-get install -y language-pack-zh-hans
# RUN locale-gen zh_CN.UTF-8 && echo "export LC_ALL=zh_CN.UTF-8">> /etc/profile && /bin/bash -c "source /etc/profile"

RUN wget -c https://repo.anaconda.com/miniconda/Miniconda3-py310_23.1.0-1-Linux-x86_64.sh -O miniconda.sh
RUN bash miniconda.sh -b
RUN rm miniconda.sh
RUN /root/miniconda3/bin/conda init
RUN /bin/bash -c "source ~/.bashrc"
RUN /root/miniconda3/bin/pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

CMD source ~/.bashrc
```

3.本地搭建docker镜像
```
docker build -t lora-training:v0 .
```

4.构建容器并进入
```
docker run -it --name lora --gpus all -v /path/to/your/workspace/:/workspace lora-training:v0 bash
cd /workspace
```

5.构建虚拟环境
```
cd kohya_ss
python -m venv venv
source venv/bin/activate
```

6.安装代码环境  

**(Optional)** 将setup.sh中的所有 *https://github* 加上代理前缀 *https://ghproxy.com/*
```
bash setup.sh -nv
```
**中间要输时区，按自己的输，我输6，19暂无发现问题**
```
bash upgrade.sh
```

7.安装xformers
```
mkdir repositories
cd repositories
git clone https://ghproxy.com/https://github.com/facebookresearch/xformers/
cd xformers

git reset --hard 3633e1afc7bffbe61957f04e7bb1a742ee910ace
git submodule update --init
```
**(Optional)** 如果上一条命令卡住时可以：  
>先ctrl+c中断  
然后将.gitmodules中所有 *https://github* 加上代理前缀 *https://ghproxy.com/*  
然后执行```git submodule sync && git submodule update --init --recursive```  
然后接着执行下面的

```
# PATH环境变量中追加cuda目录，确保编译时能识别镜像预置的cuda11.6
export PATH=$PATH:/usr/local/cuda

# 确保gcc编译时能够识别cuda的头文件
export CPATH=/usr/local/cuda/targets/x86_64-linux/include

pip install -r requirements.txt
pip install -e .
```

8.可以访问了
```
bash ./gui.sh
```

# 本教程参考：  
*https://www.bilibili.com/read/cv21373135/  
https://github.com/open-mmlab/mmdetection/tree/main/docker  
https://blog.csdn.net/DAYUZHIBULESHUI/article/details/126769252*

