# ECDICT

Free English to Chinese Dictionary Database.

## 简介

这是一份英文->中文字典的双解词典数据库，根据各类考试大纲和语料库词频收录数十万条各类单词的英文和中文释义，并按照各类考试大纲和词频进行标注。

最初开发看书软件时需要给软件添加一个内嵌字典，在网上找到了一份别人提供的 EDictAZ.txt 的文本文件，里面有差不多两万英文单词的释义，于是开始用这个文件来提供字典查询，用着用着不够用了，又找到一份四六级到 GRE 包含释义的词汇表，但是缺少音标，于是写了个爬虫从各种资料里面把音标给爬下来，外加自己补充了一些组成了一份三万基本词汇的数据库。

其后数年根据各种资料和网友贡献词库增长到 10 万左右，又找到 Linux 下面的 cdict-1.0-1.rpm 这个开源字典数据（mdict 的主词库也是根据 cdict 转换得到），并按照英国国家语料库的前 16 万单词进行校对，补全很多语料库里词频较高但是却没有收录的词条。

## 单词标注

给数据库中每个单词标注：是否是各类考试大纲词汇？以及他们在 BNC 和其他语料库里的词频顺序。BNC 词频统计的是最近几百年的历史各类英文资料，而当代语料库只统计了最近 20 年的，为什么两者都要提供呢？

很简单，quay（码头）这个词在当代语料库里排两万以外，你可能觉得是个没必要掌握的生僻词，而 BNC 里面却排在第 8906 名，基本算是一个高频词，为啥呢？可以想象过去航海还是一个重要的交通工具，所以以往的各类文字资料对这个词提的比较多，你要看懂 19 世纪即以前的各类名著，你会发现 BNC 的词频很管用。

而你要阅读各类现代杂志，当代语料库的作用就体现出来了，比如 Taliban（塔利班），在 BNC 词频里基本就没收录（没进前 20 万词汇），而在当代语料库里，它已经冒到 6089 号了，高频中的高频。

BNC 较为全面和传统，针对性学习能帮助你阅读各类国外帝王将相的文学名著，当代语料库较为现代和实时，以和科技紧密相关。所以两者搭配，干活不累。

经过标注后，你查任何一个单词，都会告诉你这个词汇是不是四六级词汇？雅思词汇？柯林斯星级是多少？是否是牛津 3000 核心词汇？传统词频和现代词频各是多少？这样单词的重要程度你就能了解个大概了。

其次是标注动词的各个时态，每个动词有现在分词，过去式，过去分词，第三人称现在时等四种时态，用 NodeBox 和 WordNet 工具包可以方便的查询每个动词的变化形式，如此你查单词时能够给你显示出来各种变体，同时将这些变体作为新的生词再次加入到字典中，大部分字典都没法查询动词的各种时态，ECDICT 可以让你方便的查询一万多个动词（几乎所有动词）的所有派生词。

## 数据格式

采用 CSV 文件存储所有词条数据，用 UTF-8 进行编码，用 Excel 的话，别直接打开，否则编码是错的。在 Excel 里选择数据，来自文本，然后设定逗号分割，UTF-8 编码即可。

| 字段        | 解释                                                       |
| ----------- | ---------------------------------------------------------- |
| word        | 单词名称                                                   |
| phonetic    | 音标，以英语英标为主                                       |
| definition  | 单词释义（英文），每行一个释义                             |
| translation | 单词释义（中文），每行一个释义                             |
| pos         | 词语位置，用 "/" 分割不同位置                              |
| collins     | 柯林斯星级                                                 |
| oxford      | 是否是牛津三千核心词汇                                     |
| tag         | 字符串标签：zk/中考，gk/高考，cet4/四级 等等标签，空格分割 |
| bnc         | 英国国家语料库词频顺序                                     |
| frq         | 当代语料库词频顺序                                         |
| exchange    | 时态复数等变换，使用 "/" 分割不同项目，见后面表格          |
| detail      | json 扩展信息，字典形式保存例句（待添加）                  |
| audio       | 读音音频 url （待添加）                                    |

提供一段 Python 程序来读取这些数据，可以用它转换到 SQLite 和 MySQL 的数据库里，方便你用更高级的方法筛选查询。词条数据大小写不敏感，不论从查询还是排序，还是编程接口。

## 词形变化

某个动词的各种时态是什么？某个形容词的比较级和最高级又是什么？某个名词的复数呢？这个单词是由哪个单词怎么演变而来的？它的原型单词（Lemma）是什么？

可能大家注意到上表有一个 `Exchange` 字段，它就是来做这个事情的，这是本词典一大特色之一，格式如下：

```text
类型1:变换单词1/类型2:变换单词2
```

比如 perceive 这个单词的 exchange 为：

```text
d:perceived/p:perceived/3:perceives/i:perceiving
```

意思是 perceive 的过去式（`p`） 为 perceived，过去分词（`d`）为 perceived, 现在分词（'i'）是 perceiving，第三人称单数（`3`）为 perceives。冒号前面具体项目为：

| 类型 | 说明                                                       |
| ---- | ---------------------------------------------------------- |
| p    | 过去式（did）                                              |
| d    | 过去分词（done）                                           |
| i    | 现在分词（doing）                                          |
| 3    | 第三人称单数（does）                                       |
| r    | 形容词比较级（-er）                                        |
| t    | 形容词最高级（-est）                                       |
| s    | 名词复数形式                                               |
| 0    | Lemma，如 perceived 的 Lemma 是 perceive                   |
| 1    | Lemma 的变换形式，比如 s 代表 apples 是其 lemma 的复数形式 |

这个是根据 BNC 语料库和 NodeBox / WordNet 的语言处理工具生成的，有了这个 Exchange ，你的 App 能为用户提供更多信息。

## 编程接口

代码 stardict.py 用于操作该数据（兼容 Python 2/3），同时实现三个类：

| 类名      | 说明                                                                    |
| --------- | ----------------------------------------------------------------------- |
| DictCsv   | 用于读写 ecdict.csv 数据格式文件                                        |
| StarDict  | 用来读写 SQLite 词典数据文件，数据字段和上面相同，接口也和 CSV 版本相同 |
| DictMySQL | MySQL 版本的数据文件读写，同样字段和接口和上面两者相同                  |

以上三个类都统一提供如下接口：

| 接口        | 说明                                                                                        |
| ----------- | ------------------------------------------------------------------------------------------- |
| query       | 查询单词，可以查整数 id（CSV 里 id 为行号，其他两者是自增量）或单词字符串，返回 Python 字典 |
| match       | 单词匹配，匹配最相似的前 N 个单词                                                           |
| query_batch | 批量查询                                                                                    |
| count       | 返回数据库词条总数                                                                          |
| register    | 注册新单词                                                                                  |
| update      | 更新单词数据，除了 id, word 两个字段外其他都可以更新                                        |
| remove      | 删除单词                                                                                    |
| commit      | 提交更改                                                                                    |

在 stardict.tools 下面还有很多帮助类的接口，便于你维护词典数据：

1. 导出两个字典数据的差异
2. 导入差异数据到一个字典
3. 导出成 StarDict（星际译王）的原格式（用于 DictEditor 生成字典）
4. 导出成 mdict 的 .mdx 的源文件（用于 MdxBuilder 生成 mdx 字典）
5. 词典格式 .csv, .db (Sqlite), mysql 之间数据互转。

## 词干查询

这个词干不是背单词时候的词根，而是 lemma。每个单词有很多变体，你编写一个抓词软件抓到一个过去式的动词 gave，如果字典里面没有的话，就需要词干数据库来查询，把 gave 转变为 give，再查词典数据库。

我扫描了 BNC 语料库全部 1 亿个词条语料生成的 lemma.en.txt 就是用来做这个事情，stardict.py 中 LemmaDB 这个类就是用来加载该数据并进行分析的。

你或许希望统计某些文档的词频，然后针对性的学习，那么你需要先将文章中出现的词先转换成该词的原型（lemma），网上有很多算法做这个事情，但是都不靠谱，最靠谱的方式就是数据库直接查询，著名的拼写检查库 hunspell 库就是这么干的。

用 LemmaDB 类可以方便的查询 ['gave', 'taken', 'looked', 'teeth'] 的 lemma 是 ['give', 'take', 'look', 'tooth']，也可以查找 'take' 这个词的若干种变体。

这个 lemma.en.txt 涵盖了 BNC 所有语料的各种词汇变形，95%的情况下你可以查到你想要的，这个作为首选方法，查不到再去依靠各种算法（判断词尾 -ed，-ing 等），最可靠的是数据库，算法次之。

## 单词词性

数据库中有一个字段 pos，就是中文里面的词性，动词还是名词，英文叫做 [pos](https://www.nltk.org/book/ch05.html) ，句子中的位置。同样是扫描语料库生成的，比如：

fuse：pos = `n:46/v:54`

代表 fuse 这个词有两个位置（词性），n（名词）占比 46%，v（动词）占比 54%，根据后面的比例，你可以知道该词语在语料库里各个 pos 所出现的频率。关于 pos 里各个字母的含义还可以看 [这里](http://www.natcorp.ox.ac.uk/tools/xaira_search.xml?ID=POS) 和 [这里](http://www.natcorp.ox.ac.uk/docs/c5spec.html)。

## 词典使用

同时支持 CSV, SQLite, MySQL 三种格式，github 上放的字典数据是 .csv，因为基于文本，便于 PR 和更改看 differ，但是本地使用 csv 很痛苦，文件大了打开速度很慢。所以自己使用时，一般都是转换成本地的 SQLite 数据库，这样快速很多，基本没有等待，查单词也很迅速。

日常使用，比如整理自己的卡片，生成 anki 数据等，SQLite 是首选，stardict.py 中提供完整的 SQLite 字典接口。如果自己要修订，建议先新生成一个比较小的 .csv，配合本地的大的 SQLite .db 数据库一起用，查一个单词时先查小的 .csv 里面有没有，没有的话再去查 SQLite 的库去，这样现在 .csv 里面把修订的工作干了，自己用一段时间没大问题，数据比较稳定的时候，一次性从 .csv 中合并到大的 SQLite 数据库中。
不合并也行，同时使用两个库。

如果你要提交 PR，可以把你的整个 SQLite 数据库导回 .csv 格式，然后在网上提交 differ 或者 patch。本地可能有多个 SQLite 数据库，一个就是这个 ECDICT 的数据，一些可能是你从其他什么某些资料里面导出来的比较好的释义例句等，就是你日积月累整理收集的各种材料。

生成 Anki 卡片的时候，你可以优先使用你自己的库的信息，你自己的库里没有了，再找 ECDICT。而 ECDICT 里面的各种词频标注，考试大纲标注，也可以给你提供不同层次的参考。比如你想把托福里面去除六级的词汇筛选出来（很多重合），这时 EDICT 本身的标注信息就能让你方便的完成这个工作了，你也可以把词频三万以下的单词导出来成为 Excel，进行更多处理。

**最新版数据太大，我已经把数据库压缩成 stardict.7z 了，外面默认的 ecdict.csv 算是一个基础版本（76 万词条）。**

## 模糊匹配

数据库中有一个隐藏字段，叫做 `sw`，这是 strip-word 的缩写，意思是单词字符串经过 strip 以后的结果。这里的 strip 不是 python string.strip 那样简单的把头部尾部的空格去除，而是去除整个字符串中非字母和数字的部分，并将字母转为小写，对应代码为：

```python
def stripword(word):
    return (''.join([ n for n in word if n.isalnum() ])).lower()
```

CSV 数据库虽然没有记录该字段，但每次加载到内存以后会自动为每个单词生成该字段，而 SQLite/MySQL 数据库则是建表的时候就包含该字段，插入单词时为该单词自动计算后插入，以后不再改变。

这是参考 Mdx 词典格式引入的模糊匹配键值，当你使用 stardict.py 中的 match 方法时，如果第三个参数为 True，将会使用 `sw` 字段进行匹配。默认 False 使用 `word` 字段匹配时，是严格匹配字符串，那么你搜索 "long-time" 这个单词，只能匹配到 "long-time" 开头的所有单词，比如：

    long-time, long-time base, long-time period, long-time cycle, ...

但是如果进行模糊匹配，按照 `sw` 字段匹配 "long-time" 这个单词，将会搜索出所有 `sw` 以 "longtime" 开头的单词，比如：

    long-time, longtime, long time, long-time base, longtime base, ...

单词随着时间的推移，最开始是一个词组，然后变为一个减号链接的组合词，最后用的多了，减号也省了。很多词典里不一定能完全包括 long-time/long time/longtime 这类词的所有形态，但首次查询搜索失败时，可以 match 一下相同 sw 的单词，然后能够定位到该单词的其他形态。

很多词典，如星际译王都不包含 strip 字段，因此常常词典中有该单词，但是由于你输入了错误的形态，导致你查不出来，而 ECDICT 作为一个负责任的词典数据库，能够让你通过 `sw` 这个字段任意查询单词的各种形态。

## 文字处理

linguist.py 里面有一些简单的 WordNet, NodeBox 封装。

## 简明英汉增强版

使用 ECDICT 的数据，生成了《[简明英汉字典增强版](https://github.com/skywind3000/ECDICT/wiki/%E7%AE%80%E6%98%8E%E8%8B%B1%E6%B1%89%E5%AD%97%E5%85%B8%E5%A2%9E%E5%BC%BA%E7%89%88)》的字典词库，可以在 GoldenDict, 欧陆, MDict, StarDict, BlueDict, EDWin 里面加载，还有 Kindle 格式，可以在 Kindle 里面挂载。这是全网收词量最多的本地化词典，再也不用联网查单词忍受网速慢，广告多和效率低的问题了。

[简明英汉增强版-说明和下载](https://github.com/skywind3000/ECDICT/wiki/%E7%AE%80%E6%98%8E%E8%8B%B1%E6%B1%89%E5%AD%97%E5%85%B8%E5%A2%9E%E5%BC%BA%E7%89%88)

## 文档索引

- [项目维基](https://github.com/skywind3000/ECDICT/wiki)
- [简明英汉增强版](https://github.com/skywind3000/ECDICT/wiki/%E7%AE%80%E6%98%8E%E8%8B%B1%E6%B1%89%E5%AD%97%E5%85%B8%E5%A2%9E%E5%BC%BA%E7%89%88)
- [简明英汉增强版-欧陆专版](https://github.com/skywind3000/ECDICT/wiki/%E7%AE%80%E6%98%8E%E8%8B%B1%E6%B1%89%E5%A2%9E%E5%BC%BA%E7%89%88%EF%BC%88%E6%AC%A7%E9%99%86%EF%BC%89)
- [选词来源](https://github.com/skywind3000/ECDICT/wiki/%E9%80%89%E8%AF%8D)
- [双解释义](https://github.com/skywind3000/ECDICT/wiki/%E5%8F%8C%E8%A7%A3%E9%87%8A%E4%B9%89)

## 欢迎贡献

采用 CSV 格式正是为了方便 GitHub 上提交 PR，管理 differ，欢迎大家提交各类词条增补。

## TODO

- ~~搜索并校对：所有动词（3 月 21 日根据 NodeBox 工具包校对完成）~~
- ~~搜索并校对：所有副词（3 月 22 日完成校对）~~
- ~~搜索并校对：所有形容词（3 月 22 日完成校对）~~
- ~~搜索并校对：所有动物名词（http://lib.colostate.edu/wildlife/atoz.php?letter=all）~~
- ~~搜索并校对：所有植物名词（http://davesgarden.com/guides/botanary/vbl/a/）~~
- ~~搜索并校对：所有地理名词（http://www.itseducation.asia/geography/a.htm）~~
- ~~搜索并校对：所有地名（https://en.wikipedia.org/wiki/Lists_of_cities_by_country）~~
- ~~补充完成非核心词汇的英文释义~~
- ~~补充各个单词的位置信息~~
- ~~补充动词的时态语态变种信息~~
- 继续修订核心两万词汇的释义准确性

## HISTORY

- 2017-4-20 在网友大力支持下，版本 1.0.14 发布，收词 222 万，同时生成 mdx 词典，见 release 页面
- 2017-4-7 收录开源词典 《[屌丝字典](https://github.com/fxsjy/diaosi)》的英汉部分。
- 2017-3-31 修正当代语料库词频数据和部分 BNC 数据。
- 2017-3-29 整理完所有动词的衍生形式
- 2017-3-28 整理文档，补全网友提供的部分释义包括一些地名和人名
- 2017-3-27 再次根据 BNC，并且自动生成字典中缺少的动词，名词和形容词的各种时态语态。
- 2017-3-26 全面统计 BNC 原始语料库中一亿个单词，生成单词变化数据库 lemma.en.txt。
- 2017-3-23 继续使用 WordNet 补全约 1 万个新增单词的英文定义。
- 2017-3-22 使用 NodeBox 校对完成所有副词和形容词。
- 2017-3-21 使用 NodeBox 校对完所有动词，并且添加动词各种时态。

## Applications
[T.vim](https://github.com/sicong-li/T.vim) vim 的翻译插件。
