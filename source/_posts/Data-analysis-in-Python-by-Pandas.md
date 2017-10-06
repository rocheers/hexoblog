---
title: Data analysis in Python by Pandas
date: 2017-10-02 00:03:37
tags:
    - Python
    - Pandas
    - Data analysis
categories:
    - Data analysis
---

Python在数据科学领域的应用真的是越来越普及，得益于Python相对来讲通俗易懂的语言风格，语法简单且容易入门的特性，给很多数据科学领域的朋友，减轻了一部分学习编程语言的繁重。`Pandas` + `NumPy` + `Matplotlib`，这三者的结合基本可以胜任任意简单的数据分析和可视化的任务。复杂一点的可能还会需要`SciPy`的帮助。

# 本文目的
这次，我打算用一篇长文来记录一下自己是如何利用Pandas进行数据分析的。网上有很多的`Pandas`入门教程，因此我这里并不打算针对所有Pandas的基础操作描述的那么清楚，还是希望更多的表达一些对于数据分析的想法和实现。

广义上，数据分析其实包含了从导入数据->清洗数据->**分析数据**->展示数据，这一从头到尾的流程。狭义上，数据分析指的就是中间分析数据这一块内容。本文按照广义上的数据分析的过程来一步步探讨。

接下来我们就正式开始本次数据分析之旅。

# 正文
下面的这一段代码主要是包的调用和一些环境配置，`Seaborn`是也是一个plot包，可用来画出比`Matplotlib`更漂亮的图，它本身是基于`Matplotlib`设计的，对`NumPy`和`Pandas`都有很好的支持。这里我就不做过多解释了，对`Seaborn`有兴趣的朋友可以留言咨询或者自行探索。
```python
import pandas as pd
from matplotlib import pyplot as plt
import seaborn as sns

%matplotlib inline

sns.set(rc={"figure.figsize": (10, 6.25)})
sns.set_style("darkgrid")
colors = ["windows blue", "amber", "faded green", "greyish", "dusty purple", "red violet", "marine", "jungle green", "chocolate brown", "dull pink", "reddish orange"]
sns.set_palette(sns.xkcd_palette(colors))
```

## 导入数据
我这次用的数据是IGN上近20年来的各种平台的游戏，来源于[这里](https://www.kaggle.com/egrinstein/20-years-of-games)。

```python
reviews = pd.read_csv("ign.csv")
```

数据读入之后，我们来看一下这里都有些什么内容。
```python
reviews.head()
```

<!--more-->

<div>
<table class="blueTable">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>score_phrase</th>
      <th>title</th>
      <th>url</th>
      <th>platform</th>
      <th>score</th>
      <th>genre</th>
      <th>editors_choice</th>
      <th>release_year</th>
      <th>release_month</th>
      <th>release_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Amazing</td>
      <td>LittleBigPlanet PS Vita</td>
      <td>/games/littlebigplanet-vita/vita-98907</td>
      <td>PlayStation Vita</td>
      <td>9.0</td>
      <td>Platformer</td>
      <td>Y</td>
      <td>2012</td>
      <td>9</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Amazing</td>
      <td>LittleBigPlanet PS Vita -- Marvel Super Hero E...</td>
      <td>/games/littlebigplanet-ps-vita-marvel-super-he...</td>
      <td>PlayStation Vita</td>
      <td>9.0</td>
      <td>Platformer</td>
      <td>Y</td>
      <td>2012</td>
      <td>9</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Great</td>
      <td>Splice: Tree of Life</td>
      <td>/games/splice/ipad-141070</td>
      <td>iPad</td>
      <td>8.5</td>
      <td>Puzzle</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Great</td>
      <td>NHL 13</td>
      <td>/games/nhl-13/xbox-360-128182</td>
      <td>Xbox 360</td>
      <td>8.5</td>
      <td>Sports</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Great</td>
      <td>NHL 13</td>
      <td>/games/nhl-13/ps3-128181</td>
      <td>PlayStation 3</td>
      <td>8.5</td>
      <td>Sports</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
  </tbody>
</table>
</div>

这里先简单介绍一下每一列都代表什么吧：
- score_phrase – IGN用一个词来评价当前游戏，与得分直接相关；
- title – 游戏名称；
- url – 完整评论的地址；
- platform – 游戏平台（PS4, PC, Xbox, etc.）；
- score – 游戏的具体评分，从1.0到10.0；
- genre – 游戏分类；
- editors_choice – 是否为IGN编辑推荐的游戏，与评分有关系。
- release_year – 游戏发布年份；
- release_month – 发布月份；
- release_day – 发布日期。

我们来看下总共多少个数据。
```python
reviews.shape
```

    (18625, 11)

看来我们这次的数据里一共`18625`条数据，一共`11`列属性。

## 清洗数据
源数据导入后一般来说是不能直接使用的，需要进行一定范围的数据清洗，不过本次的数据基本不需要清洗，收集这个数据的_Eric Grinstein_已经对数据进行了清洗工作。不过这里我们仍需要做一点简单的清洗工作，去除一些我们不需要的内容。

```python
reviews = reviews.iloc[:, 1:]
reviews.head()
```

<div>
<style>
    .dataframe {
      display: block;
      overflow-x: auto;
      white-space: nowrap;
    }

    .dataframe thead tr:only-child th {
        text-align: center;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table class="blueTable">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>score_phrase</th>
      <th>title</th>
      <th>url</th>
      <th>platform</th>
      <th>score</th>
      <th>genre</th>
      <th>editors_choice</th>
      <th>release_year</th>
      <th>release_month</th>
      <th>release_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Amazing</td>
      <td>LittleBigPlanet PS Vita</td>
      <td>/games/littlebigplanet-vita/vita-98907</td>
      <td>PlayStation Vita</td>
      <td>9.0</td>
      <td>Platformer</td>
      <td>Y</td>
      <td>2012</td>
      <td>9</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Amazing</td>
      <td>LittleBigPlanet PS Vita -- Marvel Super Hero E...</td>
      <td>/games/littlebigplanet-ps-vita-marvel-super-he...</td>
      <td>PlayStation Vita</td>
      <td>9.0</td>
      <td>Platformer</td>
      <td>Y</td>
      <td>2012</td>
      <td>9</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Great</td>
      <td>Splice: Tree of Life</td>
      <td>/games/splice/ipad-141070</td>
      <td>iPad</td>
      <td>8.5</td>
      <td>Puzzle</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Great</td>
      <td>NHL 13</td>
      <td>/games/nhl-13/xbox-360-128182</td>
      <td>Xbox 360</td>
      <td>8.5</td>
      <td>Sports</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Great</td>
      <td>NHL 13</td>
      <td>/games/nhl-13/ps3-128181</td>
      <td>PlayStation 3</td>
      <td>8.5</td>
      <td>Sports</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
  </tbody>
</table>
</div>

## 分析数据
数据清洗之后，其实就是分析过程的正式开始。在开始分析过程之前，先说点题外话。我本身对于游戏是很热爱的，从小到大，游戏机，掌机，PC，也拥有过不少的游戏平台。从和父母斗智斗勇中各种争取时间玩的红白机，小霸王学习机，到后来可以躲在被子里玩的GameBoy，但偶尔还得探出头担心父母进屋里发现自己的小秘密；再往后的世嘉，所以又不得不和父母软磨硬泡恳求游戏时间。直至家里第一台PC的出现，基本其他的游戏平台就很少碰了，除了后来的PSP，那是我从GameBoy之后时隔很多年再次拿起掌机玩游戏。要说起游戏，游戏平台，游戏的历史，真的说上三天三夜也说不完，其实这也是我为什么选择这么IGN的这个数据作为数据分析的数据来源。我也很想看看这20年来电子游戏产业的发展和趋势。

好了，咱们言归正传，就我个人而言，拿到这么多的数据之后，第一反应是：这么多的游戏，究竟是分布在了多少平台上呢？我亲身体验过的平台其实并不多，大概10个左右吧。那么这个数据集里究竟包含了多少平台呢？

```python
all_platforms = reviews["platform"].unique()
all_platforms
```

    array(['PlayStation Vita', 'iPad', 'Xbox 360', 'PlayStation 3',
           'Macintosh', 'PC', 'iPhone', 'Nintendo DS', 'Nintendo 3DS',
           'Android', 'Wii', 'PlayStation 4', 'Wii U', 'Linux',
           'PlayStation Portable', 'PlayStation', 'Nintendo 64', 'Saturn',
           'Lynx', 'Game Boy', 'Game Boy Color', 'NeoGeo Pocket Color',
           'Game.Com', 'Dreamcast', 'Dreamcast VMU', 'WonderSwan', 'Arcade',
           'Nintendo 64DD', 'PlayStation 2', 'WonderSwan Color',
           'Game Boy Advance', 'Xbox', 'GameCube', 'DVD / HD Video Game',
           'Wireless', 'Pocket PC', 'N-Gage', 'NES', 'iPod', 'Genesis',
           'TurboGrafx-16', 'Super NES', 'NeoGeo', 'Master System',
           'Atari 5200', 'TurboGrafx-CD', 'Atari 2600', 'Sega 32X', 'Vectrex',
           'Commodore 64/128', 'Sega CD', 'Nintendo DSi', 'Windows Phone',
           'Web Games', 'Xbox One', 'Windows Surface', 'Ouya',
           'New Nintendo 3DS', 'SteamOS'], dtype=object)

这么多的平台……说实话，这里有很多我听都没听过，像`Dreamcast`，`Atari 2600`，`Vectrex`等等，看来这20年，游戏产业的发展还是很多元化的，至少从游戏平台上就可以看出端倪。

有了游戏平台的信息，自然而然地就会问，每个平台大概都出过多少游戏呢？
```python
reviews["platform"].value_counts(dropna=False)
```

    PC                      3370
    PlayStation 2           1686
    Xbox 360                1631
    Wii                     1366
    PlayStation 3           1356
    Nintendo DS             1045
    PlayStation              952
    Wireless                 910
    iPhone                   842
    Xbox                     821
    PlayStation Portable     633
    Game Boy Advance         623
    GameCube                 509
    Game Boy Color           356
    Nintendo 64              302
    Dreamcast                286
    PlayStation 4            277
    Nintendo DSi             254
    Nintendo 3DS             225
    Xbox One                 208
    PlayStation Vita         155
    Wii U                    114
    iPad                      99
    Lynx                      82
    Macintosh                 81
    Genesis                   58
    NES                       49
    TurboGrafx-16             40
    Android                   39
    Super NES                 33
    NeoGeo Pocket Color       31
    N-Gage                    30
    Game Boy                  22
    iPod                      17
    Sega 32X                  16
    Windows Phone             14
    Master System             13
    Arcade                    11
    Linux                     10
    NeoGeo                    10
    Nintendo 64DD              7
    Commodore 64/128           6
    Saturn                     6
    Atari 2600                 5
    WonderSwan                 4
    TurboGrafx-CD              3
    Game.Com                   3
    Atari 5200                 2
    New Nintendo 3DS           2
    Vectrex                    2
    Pocket PC                  1
    WonderSwan Color           1
    Ouya                       1
    Web Games                  1
    SteamOS                    1
    Dreamcast VMU              1
    Windows Surface            1
    DVD / HD Video Game        1
    Sega CD                    1
    Name: platform, dtype: int64

从上面的统计来看，PC端无疑是最大的贡献者，这也可以理解，毕竟个人电脑从上个世纪末开始出现井喷，到后来虽然出货量开始下降，但一直都是人们学习生活娱乐中不可或缺的一部分，并且早期的个人电脑绝大部分都是以Windows为操作系统。不过让我没想到的是`Dreamcast`竟然还有286款游戏，看来是我孤陋寡闻了……

下面来看看排名前十的平台都有哪些。

```python
platforms = reviews["platform"].value_counts()[:10].index.tolist()
platforms
```

    ['PC',
    'PlayStation 2',
    'Xbox 360',
    'Wii',
    'PlayStation 3',
    'Nintendo DS',
    'PlayStation',
    'Wireless',
    'iPhone',
    'Xbox']

既然前十的平台我已经知道了，那么下面来看看每个平台的游戏质量如何，虽然PC端的游戏最多，但不一定好游戏占比就是最多的，对吧？

想知道每个平台的游戏质量如何，我得先从所有的数据中将只属于前十的平台的游戏提取出来。这里我创建一个filter，用来筛选游戏平台。
```python
fil = reviews["platform"] == platforms[0]   # create a filter
fil
```

    0        False
    1        False
    2        False
    3        False
    4        False
    5        False
    6        False
    7         True
    8        False
    9         True
              ...
    18615    False
    18616     True
    18617    False
    18618     True
    18619     True
    18620    False
    18621    False
    18622    False
    18623    False
    18624     True
    Name: platform, Length: 18625, dtype: bool


```python
for platform in platforms[1:]:
    fil |= reviews["platform"] == platform

filtered_reviews = reviews[fil]
```

下面是提取出来的所有数据：
```python
filtered_reviews
```

<div>
<table class="blueTable">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>score_phrase</th>
      <th>title</th>
      <th>url</th>
      <th>platform</th>
      <th>score</th>
      <th>genre</th>
      <th>editors_choice</th>
      <th>release_year</th>
      <th>release_month</th>
      <th>release_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>Great</td>
      <td>NHL 13</td>
      <td>/games/nhl-13/xbox-360-128182</td>
      <td>Xbox 360</td>
      <td>8.5</td>
      <td>Sports</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Great</td>
      <td>NHL 13</td>
      <td>/games/nhl-13/ps3-128181</td>
      <td>PlayStation 3</td>
      <td>8.5</td>
      <td>Sports</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Awful</td>
      <td>Double Dragon: Neon</td>
      <td>/games/double-dragon-neon/xbox-360-131320</td>
      <td>Xbox 360</td>
      <td>3.0</td>
      <td>Fighting</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Amazing</td>
      <td>Guild Wars 2</td>
      <td>/games/guild-wars-2/pc-896298</td>
      <td>PC</td>
      <td>9.0</td>
      <td>RPG</td>
      <td>Y</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Awful</td>
      <td>Double Dragon: Neon</td>
      <td>/games/double-dragon-neon/ps3-131321</td>
      <td>PlayStation 3</td>
      <td>3.0</td>
      <td>Fighting</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Good</td>
      <td>Total War Battles: Shogun</td>
      <td>/games/total-war-battles-shogun/pc-142564</td>
      <td>PC</td>
      <td>7.0</td>
      <td>Strategy</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Good</td>
      <td>Tekken Tag Tournament 2</td>
      <td>/games/tekken-tag-tournament-2/ps3-124584</td>
      <td>PlayStation 3</td>
      <td>7.5</td>
      <td>Fighting</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Good</td>
      <td>Tekken Tag Tournament 2</td>
      <td>/games/tekken-tag-tournament-2/xbox-360-124581</td>
      <td>Xbox 360</td>
      <td>7.5</td>
      <td>Fighting</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Good</td>
      <td>Wild Blood</td>
      <td>/games/wild-blood/iphone-139363</td>
      <td>iPhone</td>
      <td>7.0</td>
      <td>NaN</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>10</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Amazing</td>
      <td>Mark of the Ninja</td>
      <td>/games/mark-of-the-ninja-135615/xbox-360-129276</td>
      <td>Xbox 360</td>
      <td>9.0</td>
      <td>Action, Adventure</td>
      <td>Y</td>
      <td>2012</td>
      <td>9</td>
      <td>7</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Amazing</td>
      <td>Mark of the Ninja</td>
      <td>/games/mark-of-the-ninja-135615/pc-143761</td>
      <td>PC</td>
      <td>9.0</td>
      <td>Action, Adventure</td>
      <td>Y</td>
      <td>2012</td>
      <td>9</td>
      <td>7</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Okay</td>
      <td>Home: A Unique Horror Adventure</td>
      <td>/games/home-a-unique-horror-adventure/pc-137135</td>
      <td>PC</td>
      <td>6.5</td>
      <td>Adventure</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>6</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Great</td>
      <td>Avengers Initiative</td>
      <td>/games/avengers-initiative/iphone-141579</td>
      <td>iPhone</td>
      <td>8.0</td>
      <td>Action</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>5</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Mediocre</td>
      <td>Way of the Samurai 4</td>
      <td>/games/way-of-the-samurai-4/ps3-23516</td>
      <td>PlayStation 3</td>
      <td>5.5</td>
      <td>Action, Adventure</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>3</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Good</td>
      <td>JoJo's Bizarre Adventure HD</td>
      <td>/games/jojos-bizarre-adventure/xbox-360-137717</td>
      <td>Xbox 360</td>
      <td>7.0</td>
      <td>Fighting</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>3</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Good</td>
      <td>JoJo's Bizarre Adventure HD</td>
      <td>/games/jojos-bizarre-adventure/ps3-137896</td>
      <td>PlayStation 3</td>
      <td>7.0</td>
      <td>Fighting</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>3</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Good</td>
      <td>Mass Effect 3: Leviathan</td>
      <td>/games/mass-effect-3-leviathan/xbox-360-138918</td>
      <td>Xbox 360</td>
      <td>7.5</td>
      <td>RPG</td>
      <td>N</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Good</td>
      <td>Mass Effect 3: Leviathan</td>
      <td>/games/mass-effect-3-leviathan/ps3-138915</td>
      <td>PlayStation 3</td>
      <td>7.5</td>
      <td>RPG</td>
      <td>N</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Good</td>
      <td>Mass Effect 3: Leviathan</td>
      <td>/games/mass-effect-3-leviathan/pc-138919</td>
      <td>PC</td>
      <td>7.5</td>
      <td>RPG</td>
      <td>N</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Amazing</td>
      <td>Dark Souls (Prepare to Die Edition)</td>
      <td>/games/dark-souls-prepare-to-die-edition/pc-13...</td>
      <td>PC</td>
      <td>9.0</td>
      <td>Action, RPG</td>
      <td>Y</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Good</td>
      <td>Symphony</td>
      <td>/games/symphony/pc-136470</td>
      <td>PC</td>
      <td>7.0</td>
      <td>Shooter</td>
      <td>N</td>
      <td>2012</td>
      <td>8</td>
      <td>30</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Good</td>
      <td>Tom Clancy's Ghost Recon Phantoms</td>
      <td>/games/tom-clancys-ghost-recon-online/pc-109114</td>
      <td>PC</td>
      <td>7.5</td>
      <td>Shooter</td>
      <td>N</td>
      <td>2012</td>
      <td>8</td>
      <td>29</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Great</td>
      <td>Thirty Flights of Loving</td>
      <td>/games/thirty-flights-of-loving/pc-138374</td>
      <td>PC</td>
      <td>8.0</td>
      <td>Adventure</td>
      <td>N</td>
      <td>2012</td>
      <td>8</td>
      <td>29</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Okay</td>
      <td>Legasista</td>
      <td>/games/legasista/ps3-127147</td>
      <td>PlayStation 3</td>
      <td>6.5</td>
      <td>Action, RPG</td>
      <td>N</td>
      <td>2012</td>
      <td>8</td>
      <td>28</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Great</td>
      <td>World of Warcraft: Mists of Pandaria</td>
      <td>/games/world-of-warcraft-mists-of-pandaria/pc-...</td>
      <td>PC</td>
      <td>8.7</td>
      <td>RPG</td>
      <td>Y</td>
      <td>2012</td>
      <td>10</td>
      <td>4</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Bad</td>
      <td>Hell Yeah! Wrath of the Dead Rabbit</td>
      <td>/games/hell-yeah-wrath-of-the-dead-rabbit/ps3-...</td>
      <td>PlayStation 3</td>
      <td>4.9</td>
      <td>Platformer</td>
      <td>N</td>
      <td>2012</td>
      <td>10</td>
      <td>4</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Amazing</td>
      <td>Pokemon White Version 2</td>
      <td>/games/pokemon-white-version-2/nds-129228</td>
      <td>Nintendo DS</td>
      <td>9.6</td>
      <td>RPG</td>
      <td>Y</td>
      <td>2012</td>
      <td>10</td>
      <td>3</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Good</td>
      <td>War of the Roses</td>
      <td>/games/war-of-the-roses-140577/pc-115849</td>
      <td>PC</td>
      <td>7.3</td>
      <td>Action</td>
      <td>N</td>
      <td>2012</td>
      <td>10</td>
      <td>3</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Amazing</td>
      <td>Pokemon Black Version 2</td>
      <td>/games/pokemon-black-version-2/nds-129224</td>
      <td>Nintendo DS</td>
      <td>9.6</td>
      <td>RPG</td>
      <td>Y</td>
      <td>2012</td>
      <td>10</td>
      <td>3</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Okay</td>
      <td>Drakerider</td>
      <td>/games/drakerider/iphone-135745</td>
      <td>iPhone</td>
      <td>6.5</td>
      <td>RPG</td>
      <td>N</td>
      <td>2012</td>
      <td>10</td>
      <td>3</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>18546</th>
      <td>Great</td>
      <td>Devil Daggers</td>
      <td>/games/devil-daggers/pc-20049771</td>
      <td>PC</td>
      <td>8.5</td>
      <td>Shooter</td>
      <td>N</td>
      <td>2016</td>
      <td>2</td>
      <td>27</td>
    </tr>
    <tr>
      <th>18547</th>
      <td>Good</td>
      <td>Superhot</td>
      <td>/games/superhot/pc-20018899</td>
      <td>PC</td>
      <td>7.5</td>
      <td>Action</td>
      <td>N</td>
      <td>2016</td>
      <td>2</td>
      <td>25</td>
    </tr>
    <tr>
      <th>18549</th>
      <td>Good</td>
      <td>Battleborn</td>
      <td>/games/battleborn/pc-20021225</td>
      <td>PC</td>
      <td>7.1</td>
      <td>Shooter</td>
      <td>N</td>
      <td>2016</td>
      <td>5</td>
      <td>6</td>
    </tr>
    <tr>
      <th>18554</th>
      <td>Good</td>
      <td>The Park</td>
      <td>/games/the-park/pc-20042102</td>
      <td>PC</td>
      <td>7.0</td>
      <td>Adventure</td>
      <td>N</td>
      <td>2016</td>
      <td>5</td>
      <td>4</td>
    </tr>
    <tr>
      <th>18555</th>
      <td>Great</td>
      <td>Hitman: Episode 2</td>
      <td>/games/hitman-episode-2/pc-20051629</td>
      <td>PC</td>
      <td>8.5</td>
      <td>Shooter</td>
      <td>N</td>
      <td>2016</td>
      <td>4</td>
      <td>29</td>
    </tr>
    <tr>
      <th>18557</th>
      <td>Amazing</td>
      <td>Hearts of Iron IV</td>
      <td>/games/hearts-of-iron-iv/pc-20012080</td>
      <td>PC</td>
      <td>9.0</td>
      <td>Strategy</td>
      <td>Y</td>
      <td>2016</td>
      <td>6</td>
      <td>6</td>
    </tr>
    <tr>
      <th>18559</th>
      <td>Okay</td>
      <td>Dangerous Golf</td>
      <td>/games/dangerous-golf/pc-20048436</td>
      <td>PC</td>
      <td>6.0</td>
      <td>Sports, Action</td>
      <td>N</td>
      <td>2016</td>
      <td>6</td>
      <td>3</td>
    </tr>
    <tr>
      <th>18567</th>
      <td>Great</td>
      <td>Offworld Trading Company</td>
      <td>/games/offworld-trading-company/pc-20018639</td>
      <td>PC</td>
      <td>8.0</td>
      <td>Strategy</td>
      <td>N</td>
      <td>2016</td>
      <td>4</td>
      <td>28</td>
    </tr>
    <tr>
      <th>18568</th>
      <td>Okay</td>
      <td>The Walking Dead: Michonne -- Episode 3: What ...</td>
      <td>/games/the-walking-dead-michonne-episode-3/pc-...</td>
      <td>PC</td>
      <td>6.3</td>
      <td>Adventure</td>
      <td>N</td>
      <td>2016</td>
      <td>4</td>
      <td>27</td>
    </tr>
    <tr>
      <th>18570</th>
      <td>Good</td>
      <td>Battlefleet Gothic: Armada</td>
      <td>/games/battlefleet-gothic-armada/pc-20030300</td>
      <td>PC</td>
      <td>7.1</td>
      <td>Strategy</td>
      <td>N</td>
      <td>2016</td>
      <td>4</td>
      <td>22</td>
    </tr>
    <tr>
      <th>18572</th>
      <td>Amazing</td>
      <td>Overwatch</td>
      <td>/games/overwatch/pc-20027413</td>
      <td>PC</td>
      <td>9.4</td>
      <td>Shooter</td>
      <td>Y</td>
      <td>2016</td>
      <td>5</td>
      <td>28</td>
    </tr>
    <tr>
      <th>18575</th>
      <td>Good</td>
      <td>Fallout 4: Nuka World</td>
      <td>/games/fallout-4-nuka-world/pc-20054761</td>
      <td>PC</td>
      <td>7.9</td>
      <td>RPG</td>
      <td>N</td>
      <td>2016</td>
      <td>8</td>
      <td>30</td>
    </tr>
    <tr>
      <th>18578</th>
      <td>Good</td>
      <td>Master of Orion</td>
      <td>/games/master-of-orion-wargaming/pc-20038452</td>
      <td>PC</td>
      <td>7.1</td>
      <td>Strategy</td>
      <td>N</td>
      <td>2016</td>
      <td>8</td>
      <td>26</td>
    </tr>
    <tr>
      <th>18580</th>
      <td>Great</td>
      <td>Quadrilateral Cowboy</td>
      <td>/games/quadrilateral-cowboy/pc-159788</td>
      <td>PC</td>
      <td>8.5</td>
      <td>Puzzle</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>28</td>
    </tr>
    <tr>
      <th>18581</th>
      <td>Great</td>
      <td>Fallout 4: Vault-Tec Workshop</td>
      <td>/games/fallout-4-vault-tec-workshop/pc-20054769</td>
      <td>PC</td>
      <td>8.2</td>
      <td>RPG</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>27</td>
    </tr>
    <tr>
      <th>18583</th>
      <td>Great</td>
      <td>Kentucky Route Zero: Act 4</td>
      <td>/games/kentucky-route-zero-act-4/pc-20046280</td>
      <td>PC</td>
      <td>8.5</td>
      <td>Adventure</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>22</td>
    </tr>
    <tr>
      <th>18586</th>
      <td>Great</td>
      <td>F1 2016</td>
      <td>/games/f1-2016/pc-20054151</td>
      <td>PC</td>
      <td>8.8</td>
      <td>Racing</td>
      <td>N</td>
      <td>2016</td>
      <td>8</td>
      <td>24</td>
    </tr>
    <tr>
      <th>18589</th>
      <td>Amazing</td>
      <td>Deus Ex: Mankind Divided</td>
      <td>/games/deus-ex-mankind-divided/pc-20013794</td>
      <td>PC</td>
      <td>9.2</td>
      <td>Action, RPG</td>
      <td>Y</td>
      <td>2016</td>
      <td>8</td>
      <td>19</td>
    </tr>
    <tr>
      <th>18595</th>
      <td>Bad</td>
      <td>Ghostbusters</td>
      <td>/games/ghostbusters-the-movie/pc-20052317</td>
      <td>PC</td>
      <td>4.4</td>
      <td>Action</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>16</td>
    </tr>
    <tr>
      <th>18596</th>
      <td>Okay</td>
      <td>Necropolis</td>
      <td>/games/necropolis/pc-20030346</td>
      <td>PC</td>
      <td>6.5</td>
      <td>Action, Adventure</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>14</td>
    </tr>
    <tr>
      <th>18598</th>
      <td>Okay</td>
      <td>Furi</td>
      <td>/games/furi/pc-20044439</td>
      <td>PC</td>
      <td>6.8</td>
      <td>Action</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>13</td>
    </tr>
    <tr>
      <th>18600</th>
      <td>Good</td>
      <td>Hitman: Episode 4</td>
      <td>/games/hitman-episode-4/pc-20051637</td>
      <td>PC</td>
      <td>7.4</td>
      <td>Shooter</td>
      <td>N</td>
      <td>2016</td>
      <td>8</td>
      <td>19</td>
    </tr>
    <tr>
      <th>18603</th>
      <td>Good</td>
      <td>Grow Up</td>
      <td>/games/grow-up/pc-20054824</td>
      <td>PC</td>
      <td>7.8</td>
      <td>Platformer</td>
      <td>N</td>
      <td>2016</td>
      <td>8</td>
      <td>18</td>
    </tr>
    <tr>
      <th>18606</th>
      <td>Okay</td>
      <td>Starcraft II: Nova Covert Ops -- Mission Pack 2</td>
      <td>/games/starcraft-ii-nova-covert-ops-mission-pa...</td>
      <td>PC</td>
      <td>6.4</td>
      <td>Strategy</td>
      <td>N</td>
      <td>2016</td>
      <td>8</td>
      <td>4</td>
    </tr>
    <tr>
      <th>18607</th>
      <td>Good</td>
      <td>Pokemon Go</td>
      <td>/games/pokemon-go/iphone-20042699</td>
      <td>iPhone</td>
      <td>7.0</td>
      <td>Battle</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>13</td>
    </tr>
    <tr>
      <th>18613</th>
      <td>Great</td>
      <td>XCOM 2: Shen's Last Gift</td>
      <td>/games/xcom-2-shens-last-gift/pc-20055520</td>
      <td>PC</td>
      <td>8.0</td>
      <td>Strategy</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18616</th>
      <td>Good</td>
      <td>Batman: The Telltale Series -- Episode 1: Real...</td>
      <td>/games/batman-the-telltale-series-episode-1-re...</td>
      <td>PC</td>
      <td>7.5</td>
      <td>Adventure</td>
      <td>N</td>
      <td>2016</td>
      <td>8</td>
      <td>2</td>
    </tr>
    <tr>
      <th>18618</th>
      <td>Amazing</td>
      <td>Starbound</td>
      <td>/games/starbound-2016/pc-128879</td>
      <td>PC</td>
      <td>9.1</td>
      <td>Action</td>
      <td>Y</td>
      <td>2016</td>
      <td>7</td>
      <td>28</td>
    </tr>
    <tr>
      <th>18619</th>
      <td>Good</td>
      <td>Human Fall Flat</td>
      <td>/games/human-fall-flat/pc-20051928</td>
      <td>PC</td>
      <td>7.9</td>
      <td>Puzzle, Action</td>
      <td>N</td>
      <td>2016</td>
      <td>7</td>
      <td>28</td>
    </tr>
    <tr>
      <th>18624</th>
      <td>Masterpiece</td>
      <td>Inside</td>
      <td>/games/inside-playdead/pc-20055740</td>
      <td>PC</td>
      <td>10.0</td>
      <td>Adventure</td>
      <td>Y</td>
      <td>2016</td>
      <td>6</td>
      <td>28</td>
    </tr>
  </tbody>
</table>
<p>13979 rows × 10 columns</p>
</div>

## 展示数据
现在已经有了前十平台的数据，需要思考的就是如何来呈现每个平台的游戏质量呢？当然可以用每个平台的`score`的平均值来对比，但未免有点单薄了。数据属性中有一列是`score_phrase`，用一个单词来形容当前游戏的好坏，与`score`直接挂钩，用这个来展示应该会更容易理解和分析。

这里可以用`Matplotlib.pyplot`的`bar`来画，也可以用`Seaborn`中的`countplot`，后者使用起来更容易方便。
```python
sns.countplot(x="platform", hue="score_phrase", data=filtered_reviews, palette=sns.xkcd_palette(colors));
```

![png](/images/output_13_0.png)

展示的结果如上图所示，我们可以看到PC平台下，`Great`和`Good`这两栏下的游戏数量基本就占了大半，但我并不能说PC端的游戏质量就比其他平台高出一筹，因为我们依然无法判断每个平台下优秀的作品占比如何。这幅图只能直观地告诉我们每个平台下，所有分数的一个分布状况。

所以，下面的工作，我要继续细化一下数据分析和展示的部分。

## 进一步分析与展示数据
因为原先划分的`score_phrase`太多了，我决定将它们重新划为三个部分：好于`Good`的，差于`Okay`的，剩下的就是中间部分。我的这个标准可能比较严格，在我看来，评分`8.0`以上的才算的上是优秀的作品，也就是高于`Good`的；至于那些评分低于`6.0`的，也就是还不到`Okay`的，算作差劲也不算失礼吧。

```python
all_score_phrases = set(reviews["score_phrase"].unique())
bt_good = set(['Great', 'Amazing', 'Masterpiece'])
average = set(['Good', 'Okay'])
wt_okay = all_score_phrases - bt_good - average

def category_score_phrase(value):
    if value in bt_good:
        return "Better than Good"
    elif value in wt_okay:
        return "Worse than Okay"
    else:
        return "Average"

sizes = filtered_reviews["score_phrase"].apply(category_score_phrase).value_counts()
explode = (0, 0.1, 0)
plt.pie(sizes, labels=sizes.index, explode=explode, autopct='%1.2f%%', shadow=True, startangle=90);
```

![png](/images/output_14_0.png)

这里我先用饼图来展示一下前十的平台，整体的游戏质量分布情况。

这里，我创建了一个新列，叫`score_phrase_new`，为了区别原有的`score_phrase`。

```python
filtered_reviews["score_phrase_new"] = filtered_reviews["score_phrase"].apply(category_score_phrase)
filtered_reviews.head()
```

<div>
<table class="blueTable">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>score_phrase</th>
      <th>title</th>
      <th>url</th>
      <th>platform</th>
      <th>score</th>
      <th>genre</th>
      <th>editors_choice</th>
      <th>release_year</th>
      <th>release_month</th>
      <th>release_day</th>
      <th>score_phrase_new</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>Great</td>
      <td>NHL 13</td>
      <td>/games/nhl-13/xbox-360-128182</td>
      <td>Xbox 360</td>
      <td>8.5</td>
      <td>Sports</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
      <td>Better than Good</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Great</td>
      <td>NHL 13</td>
      <td>/games/nhl-13/ps3-128181</td>
      <td>PlayStation 3</td>
      <td>8.5</td>
      <td>Sports</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
      <td>Better than Good</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Awful</td>
      <td>Double Dragon: Neon</td>
      <td>/games/double-dragon-neon/xbox-360-131320</td>
      <td>Xbox 360</td>
      <td>3.0</td>
      <td>Fighting</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
      <td>Worse than Okay</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Amazing</td>
      <td>Guild Wars 2</td>
      <td>/games/guild-wars-2/pc-896298</td>
      <td>PC</td>
      <td>9.0</td>
      <td>RPG</td>
      <td>Y</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
      <td>Better than Good</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Awful</td>
      <td>Double Dragon: Neon</td>
      <td>/games/double-dragon-neon/ps3-131321</td>
      <td>PlayStation 3</td>
      <td>3.0</td>
      <td>Fighting</td>
      <td>N</td>
      <td>2012</td>
      <td>9</td>
      <td>11</td>
      <td>Worse than Okay</td>
    </tr>
  </tbody>
</table>
</div>

先来用数字直观地看一下每个平台下，每个评分阶段的数量。
```python
filtered_reviews.groupby(["platform", "score_phrase_new"]).count()
```

<div>
<table class="blueTable">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>score_phrase</th>
      <th>title</th>
      <th>url</th>
      <th>score</th>
      <th>genre</th>
      <th>editors_choice</th>
      <th>release_year</th>
      <th>release_month</th>
      <th>release_day</th>
    </tr>
    <tr>
      <th>platform</th>
      <th>score_phrase_new</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">Nintendo DS</th>
      <th>Average</th>
      <td>462</td>
      <td>462</td>
      <td>462</td>
      <td>462</td>
      <td>462</td>
      <td>462</td>
      <td>462</td>
      <td>462</td>
      <td>462</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>207</td>
      <td>207</td>
      <td>207</td>
      <td>207</td>
      <td>207</td>
      <td>207</td>
      <td>207</td>
      <td>207</td>
      <td>207</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>376</td>
      <td>376</td>
      <td>376</td>
      <td>376</td>
      <td>375</td>
      <td>376</td>
      <td>376</td>
      <td>376</td>
      <td>376</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">PC</th>
      <th>Average</th>
      <td>1394</td>
      <td>1394</td>
      <td>1394</td>
      <td>1394</td>
      <td>1393</td>
      <td>1394</td>
      <td>1394</td>
      <td>1394</td>
      <td>1394</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>1323</td>
      <td>1323</td>
      <td>1323</td>
      <td>1323</td>
      <td>1322</td>
      <td>1323</td>
      <td>1323</td>
      <td>1323</td>
      <td>1323</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>653</td>
      <td>653</td>
      <td>653</td>
      <td>653</td>
      <td>652</td>
      <td>653</td>
      <td>653</td>
      <td>653</td>
      <td>653</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">PlayStation</th>
      <th>Average</th>
      <td>362</td>
      <td>362</td>
      <td>362</td>
      <td>362</td>
      <td>362</td>
      <td>362</td>
      <td>362</td>
      <td>362</td>
      <td>362</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>313</td>
      <td>313</td>
      <td>313</td>
      <td>313</td>
      <td>313</td>
      <td>313</td>
      <td>313</td>
      <td>313</td>
      <td>313</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>277</td>
      <td>277</td>
      <td>277</td>
      <td>277</td>
      <td>277</td>
      <td>277</td>
      <td>277</td>
      <td>277</td>
      <td>277</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">PlayStation 2</th>
      <th>Average</th>
      <td>716</td>
      <td>716</td>
      <td>716</td>
      <td>716</td>
      <td>716</td>
      <td>716</td>
      <td>716</td>
      <td>716</td>
      <td>716</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>542</td>
      <td>542</td>
      <td>542</td>
      <td>542</td>
      <td>542</td>
      <td>542</td>
      <td>542</td>
      <td>542</td>
      <td>542</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>428</td>
      <td>428</td>
      <td>428</td>
      <td>428</td>
      <td>426</td>
      <td>428</td>
      <td>428</td>
      <td>428</td>
      <td>428</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">PlayStation 3</th>
      <th>Average</th>
      <td>516</td>
      <td>516</td>
      <td>516</td>
      <td>516</td>
      <td>515</td>
      <td>516</td>
      <td>516</td>
      <td>516</td>
      <td>516</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>569</td>
      <td>569</td>
      <td>569</td>
      <td>569</td>
      <td>569</td>
      <td>569</td>
      <td>569</td>
      <td>569</td>
      <td>569</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>271</td>
      <td>271</td>
      <td>271</td>
      <td>271</td>
      <td>271</td>
      <td>271</td>
      <td>271</td>
      <td>271</td>
      <td>271</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Wii</th>
      <th>Average</th>
      <td>551</td>
      <td>551</td>
      <td>551</td>
      <td>551</td>
      <td>547</td>
      <td>551</td>
      <td>551</td>
      <td>551</td>
      <td>551</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>494</td>
      <td>494</td>
      <td>494</td>
      <td>494</td>
      <td>494</td>
      <td>494</td>
      <td>494</td>
      <td>494</td>
      <td>494</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Wireless</th>
      <th>Average</th>
      <td>473</td>
      <td>473</td>
      <td>473</td>
      <td>473</td>
      <td>471</td>
      <td>473</td>
      <td>473</td>
      <td>473</td>
      <td>473</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>308</td>
      <td>308</td>
      <td>308</td>
      <td>308</td>
      <td>306</td>
      <td>308</td>
      <td>308</td>
      <td>308</td>
      <td>308</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>129</td>
      <td>129</td>
      <td>129</td>
      <td>129</td>
      <td>129</td>
      <td>129</td>
      <td>129</td>
      <td>129</td>
      <td>129</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Xbox</th>
      <th>Average</th>
      <td>307</td>
      <td>307</td>
      <td>307</td>
      <td>307</td>
      <td>307</td>
      <td>307</td>
      <td>307</td>
      <td>307</td>
      <td>307</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>160</td>
      <td>160</td>
      <td>160</td>
      <td>160</td>
      <td>160</td>
      <td>160</td>
      <td>160</td>
      <td>160</td>
      <td>160</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Xbox 360</th>
      <th>Average</th>
      <td>631</td>
      <td>631</td>
      <td>631</td>
      <td>631</td>
      <td>631</td>
      <td>631</td>
      <td>631</td>
      <td>631</td>
      <td>631</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>646</td>
      <td>646</td>
      <td>646</td>
      <td>646</td>
      <td>646</td>
      <td>646</td>
      <td>646</td>
      <td>646</td>
      <td>646</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
      <td>354</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">iPhone</th>
      <th>Average</th>
      <td>412</td>
      <td>412</td>
      <td>412</td>
      <td>412</td>
      <td>405</td>
      <td>412</td>
      <td>412</td>
      <td>412</td>
      <td>412</td>
    </tr>
    <tr>
      <th>Better than Good</th>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>315</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
      <td>321</td>
    </tr>
    <tr>
      <th>Worse than Okay</th>
      <td>109</td>
      <td>109</td>
      <td>109</td>
      <td>109</td>
      <td>108</td>
      <td>109</td>
      <td>109</td>
      <td>109</td>
      <td>109</td>
    </tr>
  </tbody>
</table>
</div>

事实上，上面的表格大部分内容也用不上，我们需要的其实就三列：**游戏平台**，**评分阶段**和**数量**。因此我就压缩一下原表格，让它变成下面的样子。
```python
count_df = filtered_reviews.groupby(["platform", "score_phrase_new"]).count().reset_index().iloc[:, :3]
count_df.rename(columns={"score_phrase": "count"}, inplace=True)
count_df
```

<div>
<table class="blueTable">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>platform</th>
      <th>score_phrase_new</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Nintendo DS</td>
      <td>Average</td>
      <td>462</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Nintendo DS</td>
      <td>Better than Good</td>
      <td>207</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nintendo DS</td>
      <td>Worse than Okay</td>
      <td>376</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PC</td>
      <td>Average</td>
      <td>1394</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PC</td>
      <td>Better than Good</td>
      <td>1323</td>
    </tr>
    <tr>
      <th>5</th>
      <td>PC</td>
      <td>Worse than Okay</td>
      <td>653</td>
    </tr>
    <tr>
      <th>6</th>
      <td>PlayStation</td>
      <td>Average</td>
      <td>362</td>
    </tr>
    <tr>
      <th>7</th>
      <td>PlayStation</td>
      <td>Better than Good</td>
      <td>313</td>
    </tr>
    <tr>
      <th>8</th>
      <td>PlayStation</td>
      <td>Worse than Okay</td>
      <td>277</td>
    </tr>
    <tr>
      <th>9</th>
      <td>PlayStation 2</td>
      <td>Average</td>
      <td>716</td>
    </tr>
    <tr>
      <th>10</th>
      <td>PlayStation 2</td>
      <td>Better than Good</td>
      <td>542</td>
    </tr>
    <tr>
      <th>11</th>
      <td>PlayStation 2</td>
      <td>Worse than Okay</td>
      <td>428</td>
    </tr>
    <tr>
      <th>12</th>
      <td>PlayStation 3</td>
      <td>Average</td>
      <td>516</td>
    </tr>
    <tr>
      <th>13</th>
      <td>PlayStation 3</td>
      <td>Better than Good</td>
      <td>569</td>
    </tr>
    <tr>
      <th>14</th>
      <td>PlayStation 3</td>
      <td>Worse than Okay</td>
      <td>271</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Wii</td>
      <td>Average</td>
      <td>551</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Wii</td>
      <td>Better than Good</td>
      <td>321</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Wii</td>
      <td>Worse than Okay</td>
      <td>494</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Wireless</td>
      <td>Average</td>
      <td>473</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Wireless</td>
      <td>Better than Good</td>
      <td>308</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Wireless</td>
      <td>Worse than Okay</td>
      <td>129</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Xbox</td>
      <td>Average</td>
      <td>307</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Xbox</td>
      <td>Better than Good</td>
      <td>354</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Xbox</td>
      <td>Worse than Okay</td>
      <td>160</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Xbox 360</td>
      <td>Average</td>
      <td>631</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Xbox 360</td>
      <td>Better than Good</td>
      <td>646</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Xbox 360</td>
      <td>Worse than Okay</td>
      <td>354</td>
    </tr>
    <tr>
      <th>27</th>
      <td>iPhone</td>
      <td>Average</td>
      <td>412</td>
    </tr>
    <tr>
      <th>28</th>
      <td>iPhone</td>
      <td>Better than Good</td>
      <td>321</td>
    </tr>
    <tr>
      <th>29</th>
      <td>iPhone</td>
      <td>Worse than Okay</td>
      <td>109</td>
    </tr>
  </tbody>
</table>
</div>

数据拿到手了，下面又该是用图形展示数据的时候。这次我们来看一下每一个评分阶段对于各自游戏平台占比究竟是多少。

```python
bar_width = 1
bar_left = [i for i in range(len(count_df) // 3)]
tick_pos = [i + (bar_width / 2) for i in bar_left]
totals = [i + j + k for i, j, k in zip(count_df["count"][::3], count_df["count"][1::3], count_df["count"][2::3])]
ave_perc = [i / j * 100 for i, j in zip(count_df["count"][::3], totals)]
bt_good_perc = [i / j * 100 for i, j in zip(count_df["count"][1::3], totals)]
wt_okay_perc = [i / j * 100 for i, j in zip(count_df["count"][2::3], totals)]
```


```python
f, ax = plt.subplots(1)
ax.bar(bar_left, 
       wt_okay_perc,
       label="Worst than Okay",
       alpha=0.9,
       width=bar_width,
       edgecolor="white")
ax.bar(bar_left,
       ave_perc,
       bottom=wt_okay_perc,
       label="Average",
       alpha=0.9,
       width=bar_width,
       edgecolor="white")
ax.bar(bar_left,
       bt_good_perc,
       bottom=[i+j for i, j in zip(wt_okay_perc, ave_perc)],
       label="Better than Good",
       alpha=0.9,
       width=bar_width,
       edgecolor="white")
plt.xticks(tick_pos, set(count_df["platform"]))
ax.set_ylabel("Percentage")
ax.set_xlabel("")
plt.legend(bbox_to_anchor=(1., 1.))
plt.setp(plt.gca().get_xticklabels(), rotation=45, horizontalalignment="right")
plt.show()
```

![png](/images/output_19_0.png)

我并没有直接把具体百分比的数值标记在上面，不过通过直观的图形依然可以看到一些信息。从图中可以看出来，相对来说，`PlayStation`，`PlayStation2`，`Wii`，`PC`和`Nintendo DS`的游戏质量都是很不错的，高质量游戏占比高，且低质量游戏占比低。`PlayStation3`虽然低质量游戏占比很小，但是高品质游戏也不算很多。`iPhone`和`Xbox`的表现算是最差的了，低质量游戏占比分属最高的一二，高品质游戏也是最低的两个平台。其实`iPhone`是这样的倒是不意外了，因为毕竟`iPhone`平台的起点相对于其他的平台要低很多，基本上三五个人，甚至一个人做出的游戏都有，这样很难保证游戏兼顾趣味性和剧情或者其他方面。在后期维护上面肯定也要比大公司开发的游戏差了很多。遗憾的是`Xbox`竟然也有如此差劲的表现，着实令我难以理解。

# 结尾
至此，我打算分析的内容就呈现完了，这就是我个人拿到数据之后一个简单的想法，然后试着去将这个想法用数据分析的方法展现出来，供自己去理解。后面我还会对这个数据集进一步的分析，比如去探讨一下年份和分数的关系，游戏类别和分数的关系。希望这篇文章可以起到抛砖引玉的作用，能让各位看完之后对于如何开始分析一份数据有自己的想法。

各位看官对于本文有任何不明白的地方，欢迎提问，也欢迎指正和建议。

# Related Links
1. [Pandas Tutorial: Data analysis with Python: Part 1](https://www.dataquest.io/blog/pandas-python-tutorial/)
2. [Pandas Tutorial: Data analysis with Python: Part 2](https://www.dataquest.io/blog/pandas-tutorial-python-2/)
3. [Stacked Percentage Bar Plot In MatPlotLib](https://chrisalbon.com/python/matplotlib_percentage_stacked_bar_plot.html)
