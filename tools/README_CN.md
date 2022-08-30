# 麒麟桌面操作系统V10 SPX系列X86架构编译记录

## 一、克隆 kaldi 的源码 到本地路径

```shell
$ git clone https://github.com/kaldi-asr/kaldi.git
```

## 二、安装编译所需依赖

```shell
$ sudo apt install -y g++ wget libtool automake autoconf subversion gfortran sox
```

然后进入到Kaldi文件夹，可以观察以下Kaldi的目录结构。

其中 **./tools，./src 和./egs**  三个目录比较重要：

- ./tools 目录存放Kaldi依赖的包
- ./src 目录存放Kaldi的源代码
- ./egs 目录存放Kaldi官方提供的一些例子

## 三、进入kaldi的工程，并切到tools目录下，编译所需要的工具

```shell
$ cd $kaldi-root/tools
$ make -j 4 # 4核编译
```

过程中会下载几个压缩包，完成之后看一下压缩包的大小是否正常，有时会因为网络问题导致压缩包没下载完，
make却完成了。但是后续编译src时却会出问题。

这里给出会下载的压缩包和大小：

```shell
$ du -hs ./*.gz

# 应该显示
# 376K    ./cub-1.8.0.tar.gz
# 1.2M    ./openfst-1.6.7.tar.gz
# 3.0M    ./sctk-20159b5.tar.gz
# 284K    ./sph2pipe-2.5.tar.gz
```

如果压缩包下载不完整的话可以删掉（把解压出来的目录也删掉），然后重新make。
```shell
$ rm -f *.gz
$ make -j4
```


如果因为网络问题无法下载，可以根据网址用别的方法下载再拷贝过来。比如修改Makefile的下载版本号：

```shell
$ vi tools/Makefile

# 修改CUB版本从1.8.0 =》 1.11.0
#CUB_VERSION ?= 1.8.0
CUB_VERSION ?= 1.11.0
```

## 四、在tools目录下，检查所需依赖

```shell
cd $kaldi-root/tools
extras/check_dependencies.sh

# 显示 extras/check_dependencies.sh: Intel MKL does not seem to be installed.
# 提示缺少mkl，根据提示安装就好
# 运行下面的命令进行安装

extras/install_mkl.sh

# 安装完成后，重新运行

extras/check_dependencies.sh

# 显示 extras/check_dependencies.sh: all OK.
```
提示缺什么就安装什么，都满足会显示 all ok。

## 五、切换目录到src下面，进行编译kaldi的c++源码

```shell
cd $kaldi-root/src
./configure
# 运行完应该显示编译需要的工具和路径
# g++编译器、openfst、cub、mkl、cuda-toolkit、线性带数库、
# openfst、cub、线性带数库，都在tools目录下
# mkl 也在tools目录下，可以在tools下运行 “extras/install_mkl.sh”进行安装
# mkl是用于cpu加速的
# cuda-toolkit 需要自己先安装好，如果需要使用GPU训练的话必须要配置
# 具体如何安装cuda这里不再赘述
# 一切配置完成后，运行以下命令

# 4核编译，根据自己的cpu数更改。所用的cpu越多编译就越快。这一步需要等一会儿
make -j4  clean depend
make -j4
```
完成上面步骤后，kaldi就编译好了.

## 六、简单的例子验证

```shell
# 进入kaldi的经典例子目录
cd $kaldi-root/egs
# 可以看到kaldi官方给出了非常丰富的例子
ls
# 其中，中文的语音识别：aishell、aishell2、multi_en、thchs30、aidatatang_200zh；
# 英文的语音识别：librispeech、multi_en、timit、fisher ……
# 声纹识别：voxceleb、aishell、cnceleb、sitw ……

# 这里我跑一个最简单的例子yesno
cd yesno/s5

# yesno项目的主体代码都在run.sh中（kaldi中的例子最上层脚本都是使用shell编写）
# 所以直接运行run.sh就可以了
./run.sh

# 脚本的第一步就是下载一个yesno的数据集，等就完事了。如果下载特别慢也可以自己用迅雷下载，
# yesno 这是一个非常小的数据集，每一条记录都是一系列yes或者no的语音，标注是由文件名来标注的。
# 全部都是.wav格式的音频文件。可以打开一个文件听一听，发现是一个老男人连续不停地说yes或者no，每个文件说8次。
# 文件名中，0代表那个位置说的是no，1代表说的是yes。这个实验没有单独的标注文件，直接采用的是文件名来标注的。
#
# 下载地址在run.sh里面很容易找到。下载完后根据run.sh的代码放到指定位置，然后接着运行run.sh
# kaldi中的开源数据一般都在openslr网站上，这个网站在国内有个镜像。
# https://openslr.magicdatatech.com/ 用这个网站下载应该会快一些
```

运行完成后，经过一段时间的训练和测试，可以看到运行结果，最后一行显示yesno的测试集准确率为100%，WER为0。到此kaldi安装就基本ok了。

![image](https://user-images.githubusercontent.com/43199883/187359949-b61e0142-0fa4-4367-8749-80941d074b44.png)

WER为0.00。看来这个例子识别的还是挺准的。

PS:WER（Word Error Rate）是字错误率，是一个衡量语音识别系统的准确程度的度量。其计算公式是WER=(I+D+S)/NWER=(I+D+S)/N，

其中I代表被插入的单词个数，D代表被删除的单词个数，S代表被替换的单词个数。也就是说把识别出来的结果中，多认的，少认的，

认错的全都加起来，除以总单词数。这个数字当然是越低越好。


