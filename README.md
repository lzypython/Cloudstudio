本项目来源于[《腾讯云 Cloud Studio 实战训练营》](https://marketing.csdn.net/p/06a21ca7f4a1843512fa8f8c40a16635)的参赛作品，该作品在腾讯云 [Cloud Studio](https://www.cloudstudio.net/?utm=csdn) 中运行无误。

@[TOC]
# 一、	爬虫
## 1.1爬虫代码
```
import requests
import pandas as pd
url_dict = {
	'全站': 'https://api.bilibili.com/x/web-interface/ranking/v2?rid=0&type=all',
	'动画': 'https://api.bilibili.com/x/web-interface/ranking/v2?rid=1&type=all',
	'生活': 'https://api.bilibili.com/x/web-interface/ranking/v2?rid=160&type=all',
	'动物圈': 'https://api.bilibili.com/x/web-interface/ranking/v2?rid=217&type=all',
	'娱乐': 'https://api.bilibili.com/x/web-interface/ranking/v2?rid=5&type=all',
	'影视': 'https://api.bilibili.com/x/web-interface/ranking/v2?rid=181&type=all',
	'原创': 'https://api.bilibili.com/x/web-interface/ranking/v2?rid=0&type=origin',
}
headers = {
	'Accept': 'application/json, text/plain, */*',
	'Origin': 'https://www.bilibili.com',
	'Host': 'api.bilibili.com',
	'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Safari/605.1.15',
	'Accept-Language': 'zh-cn',
	'Connection': 'keep-alive',
	'Referer': 'https://www.bilibili.com/v/popular/rank/all'
}

for i in url_dict.items():
	url = i[1]  # url地址
	tab_name = i[0]  # tab页名称
	title_list = []
	play_cnt_list = []  # 播放数
	danmu_cnt_list = []  # 播放数
	coin_cnt_list = []  # 投币数
	like_cnt_list = []  # 点赞数
	dislike_cnt_list = []  # 点踩数
	share_cnt_list = []  # 分享数
	favorite_cnt_list = []  # 收藏数
	author_list = []
	score_list = []
	video_url = []
	try:
		r = requests.get(url, headers=headers)
		print(r.status_code)
		# pprint(r.content.decode('utf-8'))
		# r.encoding = 'utf-8'
		# pprint(r.json())
		json_data = r.json()
		list_data = json_data['data']['list']
		for data in list_data:
			title_list.append(data['title'])
			play_cnt_list.append(data['stat']['view'])
			danmu_cnt_list.append(data['stat']['danmaku'])
			coin_cnt_list.append(data['stat']['coin'])
			like_cnt_list.append(data['stat']['like'])
			dislike_cnt_list.append(data['stat']['dislike'])
			share_cnt_list.append(data['stat']['share'])
			favorite_cnt_list.append(data['stat']['favorite'])
			author_list.append(data['owner']['name'])
			score_list.append(data['score'])
			video_url.append('https://www.bilibili.com/video/' + data['bvid'])
			print('*' * 30)
	except Exception as e:
		print("爬取失败:{}".format(str(e)))

	df = pd.DataFrame(
		{'视频标题': title_list,
		 '视频地址': video_url,
		 '作者': author_list,
		 '综合得分': score_list,
		 '播放数': play_cnt_list,
		 '弹幕数': danmu_cnt_list,
		 '投币数': coin_cnt_list,
		 '点赞数': like_cnt_list,
		 '点踩数': dislike_cnt_list,
		 '分享数': share_cnt_list,
		 '收藏数': favorite_cnt_list,
		 })
	df.to_csv('B站TOP100-{}.csv'.format(tab_name), encoding='utf_8_sig')  # utf_8_sig修复乱码问题
	print('写入成功: ' + 'B站TOP100-{}.csv'.format(tab_name))
```
## 1.2爬虫结果
 ![Alt text](image.png)
得到的是一个总站、六个分区的热门视频内容，存储在csv文件中。一共七个csv文件。打开全站文件可以看到：
 ![Alt text](image-1.png)
csv文件中存储这当前区的视频标题，地址、作者、播放数、弹幕数、投币数等信息，可以利用这些数据进行数据处理操作。
# 二、数据可视化部分
## 2.1主站分析饼状图
### 2.1.1主站分析饼状图代码
```
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
plt.rcParams['font.sans-serif']=['SimHei']
plt.rcParams['axes.unicode_minus']=False
# 全站饼状图
Total_station = pd.read_csv("B站TOP100-全站.csv")
num_dic = {}
# play_num = Total_station["播放数"]
barrage_num = Total_station["弹幕数"]
coin_num = Total_station["投币数"]
like_num = Total_station["点赞数"]
share_num = Total_station["分享数"]
collection_num = Total_station["收藏数"]
# num_dic["播放数"] = sum(play_num)
num_dic["弹幕数"] = sum(barrage_num)
num_dic["投币数"] = sum(coin_num)
num_dic["点赞数"] = sum(like_num)
num_dic["分享数"] = sum(share_num)
num_dic["收藏数"] = sum(collection_num)
Num = sum(num_dic.values())
# 单个数据
data = list(num_dic.values())
# 数据标签
labels = list(num_dic.keys())
# 各区域颜色
colors = ['green', 'orange', 'red', 'purple', 'blue']
# 数据计算处理
sizes = [data[0] / Num * 100, data[1] / Num * 100, data[2] / Num * 100, data[3] / Num * 100, data[4] / Num * 100]
# 设置突出模块偏移值
expodes = (0, 0, 0, 0.1, 0)
# 设置绘图属性并绘图
plt.pie(sizes, explode=expodes, labels=labels,shadow=True,autopct="%3.1f%%", colors=colors)
## 用于显示为一个长宽相等的饼图
plt.axis('equal')
plt.title("主站分析饼状图",fontsize=20)
# 保存并显示
plt.savefig('主站分析饼状图.png')
plt.show()
```
### 2.1.2主站分析饼状图结果
 ![Alt text](image-2.png)
## 2.2各站对比垂直图
### 2.2.1各站对比垂直图代码
```
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
plt.rcParams['font.sans-serif']=['SimHei']
plt.rcParams['axes.unicode_minus']=False


all_list =['视频标题','视频地址','作者','综合得分','播放数','弹幕数','投币数','点赞数','点踩数','分享数','收藏数']
all_dic = {}
Total_station = pd.read_csv("B站TOP100-全站.csv")
animal = pd.read_csv("B站TOP100-动物圈.csv")
animation = pd.read_csv("B站TOP100-动画.csv")
original = pd.read_csv("B站TOP100-原创.csv")
entertainment = pd.read_csv("B站TOP100-娱乐.csv")
film_television = pd.read_csv("B站TOP100-影视.csv")
life = pd.read_csv("B站TOP100-生活.csv")
# all_dic["全站"] = sum(Total_station["播放数"])
# 垂直各站对比图
all_dic["动物圈"] = sum(animal["播放数"])
all_dic["动画"] = sum(animation["播放数"])
all_dic["原创"] = sum(original["播放数"])
all_dic["娱乐"] = sum(entertainment["播放数"])
all_dic["影视"] = sum(film_television["播放数"])
all_dic["生活"] = sum(life["播放数"])
y1 = list(all_dic.values())
x = np.arange(len(y1))
plt.bar(x=x,height=y1,width=0.4)
a = [0,1,2,3,4,5]
labels = ['动物圈', '动画', '原创', '娱乐','影视','生活']
plt.xticks(a,labels,rotation = 10)
plt.xlabel('不同区名称',fontsize=10)
plt.ylabel('播放总数',fontsize=10)
plt.title("不同区前一百播放总数对比",fontsize=20)
plt.savefig("垂直各站对比图.jpg", dpi=300)
# plt.show()
```
2.2.2各站对比垂直图结果
 ![](image-3.png)
## 2.3词云分析
### 2.3.1词云分析代码
```
import wordcloud as wc
import jieba
from PIL import Image
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
Total_station = pd.read_csv("B站TOP100-全站.csv")
f = open('temp.txt',mode='w')
title = Total_station["视频标题"][:5:]
author = Total_station["作者"]
for i in title:
    f.write(i)  # write 写入
            #关闭文件
for i in author:
    f.write(i)  # write 写入
f.close()
with open("temp.txt", mode="r", encoding="GBK") as fp:
    content = fp.read()  # 读取文件内容
res = jieba.lcut(content)  # 中文分词
text = " ".join(res)  # 用空格连接所有的词
mask = np.array(Image.open("背景.jpg"))  # 指定词云图效果
word_cloud = wc.WordCloud(font_path="msyh.ttc", mask=mask)  # 创建词云对象
word_cloud.generate(text)  # 生成词语
plt.imshow(word_cloud)  # 显示词云图
word_cloud.to_file("词云分析.png")  # 保存成图片
plt.show()  # 显示图片2.4.2词云分析结果
```
 ![Alt text](image-4.png)
# 三、代码讲解
## 3.1爬虫
	首先那么需要在那么自己电脑上安装request和pandas库，如果你们是anaconda环境的话，它应该自己自带这两个库，不用再另外安装，没有这两个库的话，要自行安装，对应教程可以上ＣＳＤＮ或者Ｂ站里面找一找，教程很多，跟着他后面做就能安装上。
	ｕrl_dict ={}是定义了一个字典，这个字典的键就是分区的名字，值就是对应的url，你也可以理解为它的网址。
	Headers就是起到一个隐藏自己的作用，你在本地pycharm去爬浏览器，如果不加这个headers的话，浏览器很容易就能判断出你是一个爬虫，就把你拒之门外了。这个headers就相当于穿了一个外套，或者你也可以理解为拿到了一个浏览器认可的身份证。有了这个包装，你才可以顺利的去爬取指定的浏览器。
	接下来一个for循环，ur l_dict就是我们上面定义的字典，ur l_dict.items()就是获取它的所有键和值。url即为i[1]，tab_name = i[0]。
	try – except:用于捕获异常，防止爬虫过程中出现异常，这段指令可以让程序更加健壮。
	try里面的内容是整个爬虫的核心：r = requests.get(url, headers=headers)+ json_data = r.json()是获取目标网站的信息，返回的是一个键和值关联的嵌套字典（如下图）
 ![Alt text](image-5.png)
	list_data = json_data['data']['list']是获取键为data的字典里面键为list的值，返回的是一个列表。
	用for循环遍历list_data，将对应数据加到对应列表中，这里涉及到的知识点是列表、字典的索引，以及嵌套字典嵌套列表的索引。
	df = pd.DataFrame将对应字典转化为DataFrame格式，方便之后写入csv文件中。
	最后利用df.to_csv将数据写入csv文件中，utf_8_sig修复乱码问题。再给个提示语句，提示写入完成。
3.2主站分析饼状图
	首先通过pandas读取文件，将弹幕数、投币数、点赞数、分享数、收藏数依次用变量存储起来。
	利用字典将变量与对应变量和一一对应，总和即为data = list(num_dic.values())，数据标签为labels = list(num_dic.keys())。在设置一个颜色列表colors = ['green', 'orange', 'red', 'purple', 'blue']。
	数据计算处理，即求出每一部分占总体的多少，expodes设置模块偏移量。
	plt.pie是用来绘制饼图，在这个函数里面添加数据、标签、颜色等信息。
	再整个图片上添加标题，最后将图片保存后显示出来。
3.3各站对比垂直图
	首先读取各分区的数据，提取不同分区的播放数据，求总和作为该分区的热度。
	垂直对比图用plt.bar来绘制，需要两个基本参数，x和y。x即为不同分区的名称，y即为上面求的热度值。
	利用plt.xlabel、plt.ylabel、plt.title分别添加x，y轴的标题和整张图片标题，最后将图片保存后显示出来。
## 3.4词云分析
	首先你要安装这些依赖库：
 ![Alt text](image-6.png)
你可以先把这些指令在你本地的pycharm里面打一遍，看哪一个报错就去安装哪一个。
	同样，我们读取全站的数据，title = Total_station["视频标题"][:5:]读取热度排名前五的标题，author = Total_station["作者"]读取所有热门作者。
	with open("temp.txt", mode="r", encoding="GBK") as fp:打开temp文件，如果不存在的话就新建，利用for循环将标题和作者信息输入到temp文件中，并最后关闭文件。
	res = jieba.lcut(content)利用jieba分词器进行中文分词，并用空格连接所有词。
	mask = np.array(Image.open("背景.jpg"))指定词云图效果，之后创建词云对象，生成词语并显示词云图。
	最后保存片并显示。
