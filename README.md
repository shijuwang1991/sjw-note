# sjw-note
配置GCMC环境踩坑之路：（虚拟环境配置cuda10.2, pyhon3.7）
第一坑：
torch==1.0.0和cuda10.2版本不匹配，出现“cuDNN error: CUDNN_STATUS_EXECUTION_FAILED”报错，torch官网上明确规定了版本对应，但我没去了解就瞎装，最后出错了搞得自己很慌。
第二坑：
为解决第一坑torch换成了1.5.1版本，但
torch-scatter==1.1.1
torch-sparse==0.2.3
torch-cluster==1.2.3
torch-spline-conv==1.0.5
但由于这四个版本过低，出现各种‘undefined symbol: _ZN6caffe26detail36_typeMetaDataInstance_preallocated_7E’而且是torch-scatter/…_cpu.so报错。（不太明白为啥是cpu的.so报错，不是cuda的）
第三坑：
为解决第二坑，开始装高版本的四个依赖库。Pip install torch-scatter等 默认安装最新版，但其实并没有最新版的东西，所以下载不了也就安装不上。还一直报gcc什么错误，这里pip真的恶心。最后找到https://pytorch-geometric.com/whl/torch-1.5.0+cu102.html这个网址，下载网址所能提供的最新版，pip install 包名 安装。最终安装列表：
torch-scatter==2.0.5
torch-sparse==0.6.7
torch-cluster==1.5.7
torch-spline-conv==1.2.0
第四坑：
最坑的部分来了。安装torch-geometric时候到了。首先安装最新版1.6.3，发现gcmc源码里到处函数名称不一致或者参数不一致的问题，然后开始看源码，退版本，最终退回到1.4.3版本。这里解决到很迷茫的时候，我都想用最现有库花几周重新写作者的代码了，但吴老师看代码分析能力真的好强，修改了两处layers.py源码。就跑通了，跪服。
修改1： 65行将message_args变量替换为__msg_params__。
修改2：将77行update_args = [kwargs[arg] for arg in self.__update_args__]替换为update_args = self.__distribute__(self.__update_params__, kwargs)。
其他坑：
train.py第28行下面加上torch.cuda.set_device(device)将GPU号设置成功，不然默认在GPU：0训练；还有一些GPU/CPU数据转换问题，遇到啥改啥吧。

