学完目标检测之后一直想要用自己的数据跑一下模型看看，结果被im2rec坑了好久，看了网上的一些帖子，又回头看了一下源码，终于算是搞懂了。在这里写一下自己遇到的那些坑。

im2rec.py可以实现两种功能，一是制作相应的lst，而是制作rec文件。不管对于图片分类还是目标识别问题，均可以通过lst（通过ImageDetIter()中的path_imglist读取）和rec（通过ImageDetIter()中的path_reclist和path_idxlist读取）读取文件。

1.使用im2rec制作lst文件

调用make_list()方法获得的,其中涉及四个参数：

* recursive：是否递归访问子目录，如果存在多个目录可以设置该参数
* list：:脚本默认为False，所以制作lst时应设置为True
* prefix：需要生成的lst文件的前缀
* root：指定数据集的根目录，其子目录为图片或进一步的子目录

示例： python im2rec.py --recursive=True --list=True test data/images

坑： 不管是相对路径还是绝对路径，路径的格式错误后都会导致生成的lst文件为空。

2.使用im2rec制作rec文件
此时一般需要将–list设置为False（此时使用已经创建好的list进行rec制作），使用已经生成的lst文件生成rec文件。

涉及参数：
* –list 是否创建list文件,默认为False
* –exts 所能接受的图片后缀，默认为jpg和jpeg
* –chunks 分块数量，默认为1
* –train-ratio 训练集所占的比例，默认为1.0
* –test-ratio 测试集所占的比例，默认为0
* –recursive 是否递归的对root下的文件夹进行遍历
* –shuffle 是否打乱list中的图片顺序，默认为True
* –pass-through 是否跳过transform，默认为False
* –resize 是否将短边缩放至设定尺寸，默认为0
* –center-crop 是否进行中心剪裁，默认为False
* –quality 图片解码质量（0-100），默认为95
* –num-thread 编码的线程数，默认为1
* –color 色彩解码模式[-1,0,1]，-1为彩色模式，0为灰度模式，1为alpha模式，默认为1
* –encoding 解码模式（jpeg，png），默认为jpeg
* –pack-label 是否读入多维度标签数据，默认为False （重要，如果进行多标签数据制作或者目标检测的数据制作，那么就必须将其设置为True）

示例： python im2rec.py 41 --recursive=True --pack-label=True test data/images

坑： 一开始的时候以为要在前头加入lst文件的名称，即写成了python im2rec.py test.lst --recursive=True --pack-label=True test data/images
结果程序就报错提示unrecognized arguments

坑： 更正上述错误后，再次尝试，生成了rec文件，但是报错：lst should at least has three parts, but only has 1 parts for [’’]
回头去看了一下自己的lst文件和im2rec的代码，发现生成的lst文件在尾部多了一个空白行，所以最后报错，去除后报错消失。

附录：目标识别相关lst的格式说明
0 4 5 640 480 1 0.1 0.2 0.8 0.9 2 0.5 0.3 0.6 0.8 data/xxx.jpg

以上为一行中的数据，数据间使用 /t 分割

index A B [extra header] (object0), (object1) …   image_path

* index：图片序号 
* A：头字段宽度（minimum2，大小为2+extra header的宽度。extra header：附加字段部分，可选，多为输入图片的width&height。
* B：label字段的宽度，通常情况下为5（一个label+四个坐标数值）。
* object：标记字段，由一个label+四个坐标数值组成。

折腾了大半天的教训==遇到错误不要慌，对着错误去找代码看一般都能找得到原因。
