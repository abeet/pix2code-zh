# pix2code
**从界面截图生成代码**

[![License](http://img.shields.io/badge/license-APACHE2-blue.svg)](LICENSE.txt)

* 在[youtube上](https://youtu.be/pqKeXkhFA3I)看本系统的视频演示
* 在这儿可以找到论文 [https://arxiv.org/abs/1705.07962](https://arxiv.org/abs/1705.07962)
* 项目官网: [https://uizard.io/research#pix2code](https://uizard.io/research#pix2code)

## 摘要
计算机开发人员经常将设计师设计的图形用户界面（GUI）截图通过编译计算机代码应用到软件、网站和移动应用程序中。在本文中，我们展示了给定图形用户界面图像作为输入，深度学习技术可以被用来自动生成代码。我们的模型能够从单一输入图像中生成针对三种不同平台（即 iOS、Android 和基于 Web 的技术）的代码，其准确率超过 77％。

## 引用

```
@article{beltramelli2017pix2code,
  title={pix2code: Generating Code from a Graphical User Interface Screenshot},
  author={Beltramelli, Tony},
  journal={arXiv preprint arXiv:1705.07962},
  year={2017}
}
```

## 免责声明
以下软件仅为教育目的而共享。作者及其附属机构不以任何方式对因使用或无法使用本软件而造成的任何损害负责，包括任何直接、间接、特殊、附带或后果性的任何性质的损害。

pix2code项目是一个研究项目，演示了深层神经网络从视觉输入中生成代码的应用。当前的实现并不能在所有方面都符合预期，所以不能用在真实环境中生成代码。我们尽可能强调这个项目是实验性的，是为教育目的而分享。源代码和数据集都是为了促进机器智能的研究而提供的，而不是为最终用户设计的。

## 安装
### 先决条件

- Python 2 or 3
- pip

### 安装依赖

```sh
pip install -r requirements.txt
```

## 用法

准备数据：
```sh
# 重组和解压数据
cd datasets
zip -F pix2code_datasets.zip --out datasets.zip
unzip datasets.zip

cd ../model

# 分离训练集数据和评估集数据，确保在评估集数据中没有训练集数据
# 用法: build_datasets.py <input path> <distribution (default: 6)>
./build_datasets.py ../datasets/ios/all_data
./build_datasets.py ../datasets/android/all_data
./build_datasets.py ../datasets/web/all_data

# 转换在训练数据集中的图片（规范像素值为0~1之间的小数、调整图片尺寸为256x256）为numpy数组（如果你想要上传数据集到云端来训练你的模型，请使用较小的文件）
# 用法: convert_imgs_to_arrays.py <input path> <output path>
./convert_imgs_to_arrays.py ../datasets/ios/training_set ../datasets/ios/training_features
./convert_imgs_to_arrays.py ../datasets/android/training_set ../datasets/android/training_features
./convert_imgs_to_arrays.py ../datasets/web/training_set ../datasets/web/training_features
```

训练模型:
```sh
mkdir bin
cd model

# 指定训练数据的输入路径 和 保存训练模型及元数据的输出路径
# 用法: train.py <input path> <output path> <is memory intensive (default: 0)> <pretrained weights (optional)>
./train.py ../datasets/web/training_set ../bin

# 对已经预处理为数组的图片进行训练
./train.py ../datasets/web/training_features ../bin

# 使用generator（避免载入所有数据到内存中）进行训练（推荐）
./train.py ../datasets/web/training_features ../bin 1

# 在已经训练过的权重上进行训练
./train.py ../datasets/web/training_features ../bin 1 ../bin/pix2code.h5
```

对一批界面生成代码:
```sh
mkdir code
cd model

# 生成DSL代码(.gui后缀的文件), 默认搜索方法为 greedy（贪婪）
# 用法: generate.py <trained weights path> <trained model name> <input image> <output path> <search method (default: greedy)>
./generate.py ../bin pix2code ../gui_screenshots ../code

# 等同于上面的命令
./generate.py ../bin pix2code ../gui_screenshots ../code greedy

# 使用beam搜索并且beam宽度为3，生成 DSL 代码
./generate.py ../bin pix2code ../gui_screenshots ../code 3
```

对单张界面图生成代码:
```sh
mkdir code
cd model

# 生成DSL代码(.gui后缀的文件), 默认搜索方法为 greedy（贪婪）
# 用法: sample.py <trained weights path> <trained model name> <input image> <output path> <search method (default: greedy)>
./sample.py ../bin pix2code ../test_gui.png ../code

# 等同于上面的命令
./sample.py ../bin pix2code ../test_gui.png ../code greedy

# 使用beam搜索并且beam宽度为3，生成 DSL 代码
./sample.py ../bin pix2code ../test_gui.png ../code 3
```

编译生成的DSL代码到目标语言:
```sh
cd compiler

# 编译 .gui 文件为 Android XML UI文件
./android-compiler.py <input file path>.gui

# 编译 .gui 文件为 iOS Storyboard文件
./ios-compiler.py <input file path>.gui

# 编译 .gui 文件为 HTML/CSS (基于Bootstrap样式)
./web-compiler.py <input file path>.gui
```

## FAQ

### pix2code会支持其他目标平台/语言吗？
不，pix2code只是一个研究项目，它将保持论文中所描述的状态。这个项目其实只是对我们在Uizard Technologies所做工作的一个小小展示。当然，我们欢迎你fork，自己在其他目标平台/语言上进行实验。

### 我能在自己的前端项目中用上pix2code吗？
不，pix2code只是实验性的项目，目前还无法让你在特定案例上应用。但我们正在努力争取让它实现商业化。

### 模型的表现是如何进行测量的？
论文中所报告的准确或是错误结果都是在DSL层次上，通过对生成的token和预期的token进行比较而得出的。如果二者存在任何长度上的差异，同样会被认定为错误。

### 训练这个模型要花费多久？
在一块英伟达的Tesla K80 GPU上，要让一个数据集中包括的109 * 10^6条参数最优化，需要花费不到5个小时的时间。因此如果你想在三个目标平台上对这个模型进行训练的话，大概要花费15个小时。

### 我是一名前端开发者？我是不是很快就要失业？
*(我非常真诚地问了这个问题好多遍了……)*

**TL;DR** AI并不会那么快就把前端工程师替代。

即便假定已经存在一个成熟的pix2code版本，在每种不同的平台/语言上生成的代码都能达到100%的准确率，好的前端仍然需要逻辑、互动、高级的图形和动画，以及其他所有用户喜欢的东西。  
我们在 [Uizard Technologies](https://uizard.io) 做这个东西的目的是填平UI/UX设计师与前端开发者之间的鸿沟，而不是去代替他们。我们希望能让设计者更好地创作，同时让开发人员将自己的时间更多地花费在那些核心功能上。  
我们相信未来AI将与人类协作，而不是代替人类。

## 媒体报道

* [Wired UK](http://www.wired.co.uk/article/pix2code-ulzard-technologies)
* [The Next Web](https://thenextweb.com/apps/2017/05/26/ai-raw-design-turn-source-code)
* [Fast Company](https://www.fastcodesign.com/90127911/this-startup-uses-machine-learning-to-turn-ui-designs-into-raw-code)
* [NVIDIA Developer News](https://news.developer.nvidia.com/ai-turns-ui-designs-into-code)
* [Lifehacker Australia](https://www.lifehacker.com.au/2017/05/generating-user-interface-code-from-images-using-machine-learning/)
* [Two Minute Papers](https://www.youtube.com/watch?v=Fevg4aowNyc) (web series)
* [NLP Highlights](https://soundcloud.com/nlp-highlights/17a) (podcast)
* [Data Skeptic](https://dataskeptic.com/blog/episodes/2017/pix2code) (podcast)
* Read comments on [Hacker News](https://news.ycombinator.com/item?id=14416530)
