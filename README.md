# SoftVC VITS Singing Voice Conversion Inference Tutorial

## 💬 关于 Python 版本问题

在进行测试后，我们认为`Python 3.8.9`能够稳定地运行该项目  
推荐使用conda创建虚拟环境来运行

## Miniconda安装

点击[这里](https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe)下载windows64版Miniconda  
请务必选择add path

## 创建一个Python 3.8虚拟环境

打开Anaconda Prompt，输入第一条指令创建一个名为sovits的python3.8虚拟环境

```shell
conda create -n sovits python=3.8
```

输入第二条指令进入名为sovits的虚拟环境

```shell
conda activate sovits
```

此时可以看见Anaconda Prompt前面的(base)变成(sovits)，表明成功进入虚拟环境

```shell
(sovits) C:\Users\nia>
```

## 进入so-vits-svc-4.1-Stable仓库地址

在Anaconda Prompt里使用cd命令进入你的so-vits-svc-4.1-Stable仓库地址

```shell
cd path/to/your/so-vits-svc-4.1-Stable
```

---------------------------
举例，我在启动Anaconda Prompt之后，我的当前目录在

```shell
(sovits) C:\Users\nia>
```

我的so-vits-svc-4.1-Stable仓库在

```shell
D:\Library\ShizukuLulu\Project_AIlulu\so-vits-svc-4.1-Stable
```

我要在Anaconda Prompt输入

```shell
D:
cd D:\Library\ShizukuLulu\Project_AIlulu\so-vits-svc-4.1-Stable
```

最终跳转到对应仓库中

```shell
(sovits) D:\Library\ShizukuLulu\Project_AIlulu\so-vits-svc-4.1-Stable>
```

注意，因为跨了磁盘，所以在使用cd命令前，要输入磁盘号+冒号先跳转到对应的磁盘

## 安装requirements（运行代码所需的环境依赖）

在Anacoda Prompt里复制粘贴以下代码并回车安装依赖

```shell
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 模型&配置文件放置

### **若使用 contentvec 作为声音编码器**

`vec768l12`与`vec256l9` 需要该编码器

+ contentvec ：[checkpoint_best_legacy_500.pt](https://ibm.box.com/s/z1wgl1stco8ffooyatzdwsqn2psd9lrr)
  + 放在`pretrain`目录下

或者下载下面的 ContentVec，大小只有 199MB，但效果相同：

+ contentvec ：[hubert_base.pt](https://huggingface.co/lj1995/VoiceConversionWebUI/resolve/main/hubert_base.pt)
  + 将文件名改为`checkpoint_best_legacy_500.pt`后，放在`pretrain`目录下

### 训练好的模型和配置文件

+ 模型文件 `G_XXXXX.pth`
  + 放在`logs/44k`目录下
+ 配置文件 `config.json`
  + 放在`config`目录下

### 推理的音频放置

+ 例：`test.wav`
  + 放在`raw`目录下

## 🤖 推理

使用 [inference_main.py](https://github.com/svc-develop-team/so-vits-svc/tree/4.1-Stableinference_main.py)

```shell
# 例
python inference_main.py -m "logs/44k/G_30000.pth" -c "configs/config.json" -n "test.wav" -t 0 -s "lulu"
```

必填项部分：

+ `-m` | `--model_path`：模型路径
+ `-c` | `--config_path`：配置文件路径
+ `-n` | `--clean_names`：wav 文件名列表，放在 raw 文件夹下
+ `-t` | `--trans`：音高调整，支持正负（半音）
+ `-s` | `--spk_list`：合成目标说话人名称
+ `-cl` | `--clip`：音频强制切片，默认 0 为自动切片，单位为秒/s

可选项部分：部分具体见下一节

+ `-lg` | `--linear_gradient`：两段音频切片的交叉淡入长度，如果强制切片后出现人声不连贯可调整该数值，如果连贯建议采用默认值 0，单位为秒
+ `-f0p` | `--f0_predictor`：选择 F0 预测器，可选择 crepe,pm,dio,harvest,rmvpe,fcpe, 默认为 pm（注意：crepe 为原 F0 使用均值滤波器）
+ `-a` | `--auto_predict_f0`：语音转换自动预测音高，转换歌声时不要打开这个会严重跑调
+ `-cm` | `--cluster_model_path`：聚类模型或特征检索索引路径，留空则自动设为各方案模型的默认路径，如果没有训练聚类或特征检索则随便填
+ `-cr` | `--cluster_infer_ratio`：聚类方案或特征检索占比，范围 0-1，若没有训练聚类模型或特征检索则默认 0 即可
+ `-eh` | `--enhance`：是否使用 NSF_HIFIGAN 增强器，该选项对部分训练集少的模型有一定的音质增强效果，但是对训练好的模型有反面效果，默认关闭
+ `-shd` | `--shallow_diffusion`：是否使用浅层扩散，使用后可解决一部分电音问题，默认关闭，该选项打开时，NSF_HIFIGAN 增强器将会被禁止
+ `-usm` | `--use_spk_mix`：是否使用角色融合/动态声线融合
+ `-lea` | `--loudness_envelope_adjustment`：输入源响度包络替换输出响度包络融合比例，越靠近 1 越使用输出响度包络
+ `-fr` | `--feature_retrieval`：是否使用特征检索，如果使用聚类模型将被禁用，且 cm 与 cr 参数将会变成特征检索的索引路径与混合比例

浅扩散设置：

+ `-dm` | `--diffusion_model_path`：扩散模型路径
+ `-dc` | `--diffusion_config_path`：扩散模型配置文件路径
+ `-ks` | `--k_step`：扩散步数，越大越接近扩散模型的结果，默认 100
+ `-od` | `--only_diffusion`：纯扩散模式，该模式不会加载 sovits 模型，以扩散模型推理
+ `-se` | `--second_encoding`：二次编码，浅扩散前会对原始音频进行二次编码，玄学选项，有时候效果好，有时候效果差

### 推理完成的结果位置

+ 例：`test.wav_0key_lulu_sovits_pm.wav`
  + 位于`results`目录下
