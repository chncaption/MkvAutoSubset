# Mkv Auto Subset

![GitHub release (latest SemVer including pre-releases)](https://img.shields.io/github/v/release/MkvAutoSubset/MkvAutoSubset?include_prereleases)

ASS字幕字体子集化 MKV批量提取/生成

## 什么叫字幕字体子集化
- 这里说的字幕特指ass(ssa)这种带有特效的文本字幕;
- ass字幕会引用一些字体,这些字体在播放器所在的系统里可能有安装,也可能没有;
- 为了实现在任意地方都能有完整的视觉体验,可以把字幕以及字幕里所引用的字体文件一起打包进mkv文件里;
- 以上的操作存在一个问题,有些字幕会引用很多字体,这些字体文件体积动辄几十MB,而字幕只用到了其中的几个字而已;
- 比如一个番剧本体200M,但打包了字体文件后变成400M了,这像画吗?
- 综上所述,子集化的目的就是把字体拆包,找出字幕用到的那部分字形并重新打包;
- 好处不仅限于节约存储空间,加快缓冲速度;
- 想想看:在一个只有30Mbps上传的网络环境下,要看上面那个光字体就200M的番剧,这河里吗?

## mkvtool 安装

### 202205新增的Docker镜像使用说明
- 从Dockerhub获取
  ```shell
  TAGNAME=master
  FONT_DIR="/usr/share/fonts/truetype" #字体目录
  CACHE_DIR="${HOME}/.mkvtool/caches"  #缓存目录
  OTHER_DIR="" #其他目录(可选,示例见下节.)
  docker pull ac79b0c6/mkvtool:${TAGNAME} #拉取/更新镜像
  docker run --rm -it -v ${FONT_DIR}:/fonts -v ${CACHE_DIR}:/root/.mkvtool/caches ${OTHER_DIR} ac79b0c6/mkvtool:${TAGNAME} #运行镜像
  ```
- 手动构建&运行
  ```shell
  git clone https://github.com/MkvAutoSubset/MkvAutoSubset.git #克隆项目
  cd MkvAutoSubset #进入项目目录
  sh docker/rebuild.sh #构建镜像
  cp docker/run.sh docker/run_my.sh  #拷贝一份自己的运行脚本
  vi docker/run_my.sh #修改自己的运行脚本(可选)
  sh docker/run_my.sh #运行镜像
  ```
- docker/run_my.sh的修改说明
  * FONT_DIR: 字体文件目录
  * CACHE_DIR: 缓存目录
  * OTHER_DIR: 其他目录(可选)
    * 示例:“-v ${HOME}/work:/work”

### 依赖

- FontTools
  ```shell
  apt install fonttools #Debian/Ubuntu
  apk add py3-fonttools #Alpine
  brew install fonttools #macOS
  pip install fonttools #Use pip
  ```
- MKVToolNix
  ```shell
  apt install mkvtoolnix #Debian/Ubuntu
  apk add mkvtoolnix #Alpine
  brew install mkvtoolnix #macOS
  ```
- ass2bdnxml

  从[这里](https://github.com/Masaiki/ass2bdnxml/releases)获取

#### 关于Windows用户

- 从 [这里](https://www.python.org/downloads) 下载并安装Python
- 命令提示符(CMD)里参考上面使用pip的方式安装FontTools依赖
- 从 [这里](https://github.com/Masaiki/ass2bdnxml/releases) 获取ass2bdnxml
- 从 [这里](https://www.fosshub.com/MKVToolNix.html) 下载并安装MKVToolNix
- 保证以上两个依赖项的相关可执行文件(_ttx.exe_,_pyftsubset.exe_,_mkvextract.exe_,_mkvmerge.exe_,_ass2bdnxml.exe_)在 **path** 环境变量里

### 本体

- 有安装Go的情况:
  ```shell
  go install github.com/MkvAutoSubset/MkvAutoSubset/mkvtool@latest #安装和更新
  ```

- Arch Linux用户(通过Arch User Repository):
  - 点击[这里](https://aur.archlinux.org/packages/mkvtool/) 查看具体信息或使用AUR Helper
  ```shell
  yay -S mkvtool #yay
  paru -S mkvtool #paru
  ```

- 手动安装:

  [点此下载](https://github.com/MkvAutoSubset/MkvAutoSubset/releases/latest)
- 适用于Win64的GUI版及动态链接库

  [点此下载](https://github.com/MkvAutoSubset/MkvAutoSubset/releases/gui)
## mkvtool 功能及使用示例
- 2022.04更新的ASS转PGS说明
  ```shell
  mkvtool -a2p -apc -pr 1920*1080 -pf 23.976 ...xxx...
  
  #-a2p: 启用ass转pgs(依赖ass2bdnxml)
  #-apc: 使pgs字幕与子集化后的ass字幕共存(该选项会影响混流行为)
  #-pr: 设置pgs字幕的分辨率(例如"720p,1080p,2k"或者类似"720*480")
  #-pf: 设置pgs字幕的帧率(例如"23.976, 24, 25, 30, 29.97, 50, 59.94, 60"或者类似"15/1")
  ```
- 2022.05更新的不覆盖已有文件说明(默认覆盖模式,影响标准工作流以及"-c","-m"模式.)
  ```shell
  mkvtool -no ...xxx...
  
  #-no: 当存在"-no"参数时将不覆盖已有文件
  ```
- 2022.05更新的检查模式说明(默认开启严格模式,影响子集化进程)
  ```shell
  mkvtool -ncks ...xxx...
  
  #检查模式:在字体子集化前检查对应字体文件是否确实包含所需要的字形,如有遗失会提醒.
  #严格模式:如目标字体文件未包含所需的全部字形则会跳过匹配.
  #-ncks: 当存在"-ncks"参数时将禁用严格模式
  #-nck: 当存在"-nck"参数时将禁用检查模式
  ```
- 2022.05新增的字体信息查看模式说明
  ```shell
  mkvtool -i path
  
  #path: 字体文件路径,输出字体文件包含的所有名称和族信息.
  ```
- 2022.05新增的生成测试视频说明(仅子集化模式可用)
  ```shell
  mkvtool -t "-" ...xxx...
  
  #-t: 当"-t"参数不为空时,将在子集化输出目录创建测试视频.
  #当"-t"的值为"-"时,将创建和字幕等长的空视频(依赖ffmpeg).
  #-b: 当存在"-b"参数时,会将ass字幕烧录进视频(依赖ffmpeg).
  #-e: 启用烧录模式时,指定编码器，如"libx264","h264_qsv","nvenc_h264"等,默认为"libx264".
  ```
- 2022.04新增的不重命名字幕说明(影响子集化进程)
  ```shell
  mkvtool -n ...xxx...
  
  #-n: 当存在"-n"参数时将仅创建字体的子集化版本.不对字体名称进行修改.
  ```
- 2022.04新增的输出MKS格式说明(影响"-m","-c"模式)
  ```shell
  mkvtool -mks ...xxx...
  
  #-mks: 启用MKS格式输出
  ```
- 缓存相关(缓存会影响工作流,即无需额外准备在缓存内的字体.)
  - 创建字体缓存(推荐缓存 [超级字体整合包 XZ](https://www.dmhy.org/topics/view/516705_XZ.html) "完整包"可完全缓存,但不保证所有字体都能成功子集化.)
    ```shell
    mkvtool -cc -s input #从${input}获取字体信息并创建缓存
  
    #可选"-cp"参数:指定缓存文件的保存目录.
    #可选"-clean"参数:清空缓存目录.
    ```
  - 取得一个(或目录里所有)ass字幕文件所需要的全部字体
    ```shell
    mkvtool -l -s input #从${input}目录获取
    mkvtool -l -f file #从${file}文件获取
  
    #可选"-cfc"参数:当"-cfc"存在时,将从字体缓存中复制需要的字体到指定目录.
    #可选"-co"参数:指定字体复制的目标目录.
    #可选"-cp"参数:指定要使用的缓存目录.
    ```
- 标准工作流
  ```shell
  mkvtool -s bangumi 
  
  #从${bangumi}文件夹抽取所有mkv文件的字幕和字体,
  #遇到ass字幕就自动进行子集化,
  #输出替换字幕和字体后的新mkv文件.
  #-data参数默认值为"${workdir}/data",指定提取mkv的输出文件夹.
  #-dist参数默认值为"${workdir}/dist",指定重组后mkv的输出文件夹.
  ```
- 从单个(或文件夹的)mkv文件里抽取字幕和字体*并创建子集化后的版本(可选)*
  ```shell
  mkvtool -d -f file.mkv #单个文件
  mkvtool -d -s bangumi #文件夹
  
  #可选"-n"参数:当"-n"存在时,只抽取内容,不进行子集化操作.
  #可选"-data"参数,指定输出文件夹,默认输出到"${workdir}/data".
  ```
- 检测单个(或文件夹的)mkv文件字幕和字体,判断是否需要子集化.
  ```shell
  mkvtool -q -f file.mkv #单个文件,会直接输出是否需要子集化
  mkvtool -q -s bangumi #文件夹,会将需要子集化的文件列表输出至"${workdir}/list.txt".
  ```
- 将子集化后的字幕与字体替代原有的内容
  ```shell
  mkvtool -m -s bangumi -data data -dist dist
  
  #-data参数默认值为"${workdir}/data",字幕和字体的数据文件夹.
  #-dist参数默认值为"${workdir}/dist",重组后mkv的输出文件夹.
  #假设bangumi文件夹里的目录结构如下所示:
  #bangumi
  # |-- S01
  # ||-- abc S01E01.mkv
  # ||-- abc SxxExx.mkv
  # |-- SP.mkv
  # |-- xx.mkv
  #那么对应的data文件夹的目录结构应该是如下的所示:
  #data
  # |-- S01
  # ||-- abc S01E01
  # |||-- ...
  # |||-- subsetted
  # |||-- xxx.sub
  # ||-- abc SxxExx
  # |||-- ...
  # |||-- subsetted
  # |||-- xxx.sub
  # |||-- ...
  # |-- SP
  # |||-- ...
  # |||-- subsetted
  # |||-- xxx.sub
  # |||-- ...
  # |-- xx
  # |||-- ...
  # |||-- subsetted
  # |||-- xxx.sub
  # |||-- ...
  
  #*奇淫巧技:指定一个没有任何内容的data文件夹,将输出一个"干净"的mkv文件.
   ```
- 从一组文件夹获得情报并生成一组mkv
  ```shell
  mkvtool -c -s bangumi
  
  #可选"-clean"参数:当"-clean"存在时,将清空原有的字幕和字体(默认为追加).
  #bangumi文件夹里的目录结构应如下所示:
  #bangumi
  # |-- v
  # ||-- aaa.mkv
  # ||-- bbb.mp4
  # ||-- ccc.avi
  # |-- s
  # ||-- aaa.ass
  # ||-- aaa.srt
  # ||-- aaa.sup
  # ||-- aaa.xxx
  # ||-- bbb.xxx
  # ||-- ccc.xxx
  # |-- f
  # ||-- abc.ttf
  # ||-- def.ttc
  # ||-- ghi.otf
  # ||-- ...
  
  #若遇到ass字幕会自动进行子集化操作.
  #成品会放在"${bangumi}/o"文件夹中.
  ```
- 对一个(或多个)ass字幕进行字体子集化
  ```shell
  mkvtool -a aaa.ass -a bbb.ass -af fonts -ao output [-ans]
  
  #"-a"参数为ass字幕文件路径,可复用.
  #"-af"参数为字体文件夹路径,默认值为"${workdir}/fonts".
  #"-ao"参数为子集化成品输出路径,默认值为"${workdir}".
  #*当"-ans"参数存在时输出文件夹为"${output}",否则为${output}/subsetted".
  #*由于会预先清空${output}文件夹,为了安全请慎用"-ans".
  ```
  
### 一些碎碎念
- "-cp"参数:手动指定缓存文件路径,当提供的字体目录里缺少字体时,会尝试在缓存里查找.
- "-log"参数:输出终端输出到指定文件,空为不输出,默认为空.
- "-m","-c"模式下的"-sl","-st"参数:
   ```
   -sl:字幕语言.格式为语言缩写如"chi","jpn","eng"等,默认值为"chi".
   -st:字幕标题.该字幕在播放器里显示的标题,默认值为空.
   ```
- 字幕文件名规范:
  ```
  抽取出来的字幕长得像是如下的样子:
  #a_b_c.d
  #:如果文件名以"#"开头,代表这个轨道是默认轨道.
  a:轨道编号(在"-c"模式里,这里应该和视频文件的文件名相同.)
  b:字幕语言代码
  c:字幕标题
  d:字幕文件后缀名
  
  那么,请体会在"-c"模式中,以下的命名方式所带来的便利:
  |-- v
  ||-- aaa.mp4
  |-- s
  ||-- #aaa_chi_简体中文.ass
  ||-- aaa_chi_繁體中文.srt
  ||-- aaa_jpn_日本語.sup
  ||-- aaa_eng_English.srt
  ```
- 字幕语言代码表:

  [点此获取](https://www.science.co.il/language/Codes.php)
  
  
## Warning
**No escape of specsial chars and quotes to avoid string splitting and sub folders**
Track names with `/` or other special chars will break mkvtool. Same goes for font name wirh `'!#` or other special chars. Arguments are not qouted & escaped for cli mkvmerge
