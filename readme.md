## 一、使用平台

Windows 10、VMware workstation+ Ubuntu14 LTS

(1) Soundrecorder 测试下能否使用  
(2) 安装libasound2依赖包
```
sudo apt-get install libasound2-dev
```

## 二、CMUSphinx语音识别工具包

Pocketsphinx — 用C语言编写的轻量级识别库

Sphinxbase — Pocketsphinx所需要的支持库

Sphinx3 — 为语音识别研究用C语言编写的解码器

CMUclmtk — 语言模型工具

Sphinxtrain — 声学模型训练工具  

Acoustic and Language Models / Mandarin — 普通话语音模型

下载网址：http://sourceforge.net/projects/cmusphinx/files/

以上对应所使用的版本如下：

pocketsphinx-0.6.1（pocketsphinx_0.6.1-1.tar.gz）

sphinxbase-0.6.1（sphinxbase-0.6.1.tar.gz）

sphinx3-0.8（sphinx3-0.8.tar.bz2）

cmuclmtk（cmusphinx-cmuclmtk.tar.gz）

SphinxTrain-1.0（SphinxTrain-1.0.tar.bz2）  

Acoustic and Language Models/Mandarin/zh_broadcastnews_16k_ptm256_8000.tar.bz2

实现语音识别只需要pocketsphinx，sphinxbase就ok了
cmuclmtk，SphinxTrain-1.0是训练语言模型用的，我们是用现成，成熟的普通话语言模型。
sphinx3-0.8是编程的时候用的，目前没用过

## 三、安装pocketsphinx

由于pocketsphinx依赖于另外一个库Sphinxbase,所以先需要安装Sphinxbase。

(1) 安装Sphinxbase
```
tar xzf sphinxbase-0.6.1.tar.gz

cd sphinxbase-0.6

./configure

make

sudo make install
```
默认安装在/usr/local/bin下面，ls可查看。

(2) 安装pocketsphinx
```
export  LD_LIBRARY_PATH=/usr/local/lib

export  PKG_CONFIG_PATH=/usr/local/lib/pkgconfig

cd  pocketsphinx-0.6.1

./configure

make

sudo make install
```
完成安装,在/usr/local/bin下面可以看到三个新生成的文件，
```
cd  /usr/local/bin

ls

pocketsphinx_batch

pocketsphinx_continuous

pocketsphinx_mdef_convert
```
测试下安装结果
pocketsphinx_continuous

若出现如下信息，说明安装成功。
```
INFO: cmd_ln.c(512): Parsing command line:  
pocketsphinx_continuous  
Current configuration:  
[NAME]              [DEFLT]             [VALUE]  
-adcdev                       
-agc              none             none  
-agcthresh     2.0         2.000000e+00  
-alpha           0.97              9.700000e-01  
-argfile                  
-ascale          20.0              2.000000e+01  
-backtrace     no          no  
-beam           1e-48            1.000000e-48  
-bestpath      yes         yes  
-bestpathlw  9.5         9.500000e+00  
-bghist          no          no  
-ceplen          13          13  
-cmn             current          current  
-cmninit 8.0         8.0  
…………………………………这里省略一万字…………………………………

INFO: ngram_search_fwdtree.c(333): after: 457 root, 13300 non-root channels, 26 single-phone words  
INFO: ngram_search_fwdflat.c(153): fwdflat: min_ef_width = 4, max_sf_win = 25  
Warning: Could not find Mic element  
INFO: continuous.c(261): pocketsphinx_continuous COMPILED ON: Feb 21 2011, AT: 22:31:47  
READY....  
```
这时候其实可以识别语音了，不过是默认自带的英文识别，我们要的是中文识别。

## 四、编写自己的命令集（可以忽略这部，这部生成的文件我已经给你了）
```
<s>上一首</s>
<s>下一首</s>
<s>停止</s>
<s>播放</s>
<s>暂停</s>
```
保存为command.txt  
在[Sphinx Knowledge Base Tool](http://www.speech.cs.cmu.edu/tools/lmtool.html)上点Browse，提交command.txt，
在线生成语言模型文件。这里只要生成的lm文件,命名为0506.lm。

从这里下载pocketsphinx-win32,解压后在/model/lm/zh_cn有个mandarin_notone.dic的文件，
打开后，搜索command.txt里面的词，然后替换相应的内容，替换后的内容如下
```
</S>    EH S
<S>    EH S
上一首  sh ang y i sh ou
下一首  x ia y i sh ou
停止    t ing zh ib
播放    b o f ang
暂停    z an t ing
```
保存为0506.dic

## 五、开始测试
解压zh_broadcastnews_ptm256_8000  
把0506.dic，0506.lm，zh_broadcastnews_ptm256_8000放到 /usr/local/bin/  

在/usr/local/bin/目录下执行
```
pocketsphinx_continuous -hmm zh_broadcastnews_ptm256_8000 -lm 0506.lm -dict 0506.dic
```
看到ready后说我们的命令就可以识别了。  

### 多说两句  
sphinx语音识别其实是基于统计语言模型的  
它主要靠language model（lm），Hidden Markov Model（hmm）模型识别语音。  

其中lm模型是统计语料得到的模型，语料就是用于训练的文本库，Dic里面保存的就是训练用的语料库里出现过的语料和对应的发音  
而lm模型里存的是语料的组合概率。  

我只是调用在线生成的lm模型和我们自己写的字典dic，加上别人做好的hmm模型，这样就可以识别到我们的中文命令……  

## 六、编程测试
1，之前给你的两份文件和一个文件夹的内容放到以下的目录，并给予足够的读写权限（我很暴力地给了-R 777）  
/usr/local/share/pocketsphinx/model/hmm/zh_broadcastnews_ptm256_8000  
/usr/local/share/pocketsphinx/model/lm/0506.lm  
/usr/local/share/pocketsphinx/model/lm/0506.dic

2，编译的源文件目录随意，编译命令如下
```
gcc -o test_ps test_ps.c -DMODELDIR=\"`pkg-config --variable=modeldir pocketsphinx`\" `pkg-config --cflags --libs pocketsphinx sphinxbase`
```
3，编译完之后运行程序，看到ready后说话即可识别  
识别的命令就是之前的几个，播放，暂停，停止，上一首，下一首  

## 参考文献
[Sphinx武林秘籍](http://www.cnblogs.com/huanghuang/archive/2011/07/14/2106579.html)  
[Sphinx语音识别学习记录（四）-小范围语音中文识别](http://www.cnblogs.com/yin52133/archive/2012/07/12/2588201.html)  
[语音识别的基础知识与CMUsphinx介绍](http://blog.csdn.net/zouxy09/article/details/7941585)  
[PocketSphinx语音识别系统的编程](http://blog.csdn.net/zouxy09/article/details/7978108)  
[Android平台使用PocketSphinx做离线语音识别，小范围语音99%识别率](http://zuoshu.iteye.com/blog/1463867)  