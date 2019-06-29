# my-python
study
#爬取歌词并保存
#coding:utf-8
#!/usr/bin/python
# -*- coding: latin-1 -*-
import os, sys
import requests
import json
import re

from bs4 import BeautifulSoup
singer_url = 'http://music.163.com/artist?id=999220'  #根据歌手id，获取链接
web_data = requests.get(singer_url)#访问链接
soup = BeautifulSoup(web_data.text, 'lxml')#使用beautifulsoup的lxml解析库
singer_name = soup.select("#artist-name")#获取歌手名字
r = soup.find('ul', {'class': 'f-hide'}).find_all('a')
r = (list(r))
music_id_set = []#歌手的音乐id列表
music_name_set=[]#歌手的音乐名字列表
for each in r:
    song_name = each.text  #歌曲名字
    song_id = each.attrs["href"]#歌曲id
    music_name_set.append(song_name)#存入名字
    music_id_set.append(song_id[9:])#歌曲id，从第九个字符开始取id
#print(music_id_set)
dic = dict(map(lambda x, y: [x, y],  music_id_set,music_name_set))  #将音乐和名字组成一个字典
print(dic)

def get_lyric_by_music_id(music_id):  # 定义通过id得到歌词的函数
        lrc_url = 'http://music.163.com/api/song/lyric?' + 'id=' + str(music_id) + '&lv=1&kv=1&tv=-1'#得到歌词链接

        lyric = requests.get(lrc_url)
        json_obj = lyric.text
        # print(json_obj)
        j = json.loads(json_obj)
        # print(type(j))#打印出来的j类型是字典
        try:  # 防止部分音乐没有歌词的情况
            lrc = j['lrc']['lyric']
            pat = re.compile(r'\[.*\]')
            lrc = re.sub(pat, "", lrc)
            lrc = lrc.strip()
            return lrc
        except KeyError as e:
            pass

f=open("C:/python/wang/contents.txt",'w')#自动创建文本用来保存所有歌词
for i in music_id_set: 
        lyric = get_lyric_by_music_id(i)#根据id得到歌词
        if lyric==None:#没有歌词的情况
            print("No lyric")
            continue
        else:
            print(dic[i])#打印有歌词的曲目
            try:
                for index in lyric:
                     f.write(index)#将歌词存入文本

            except UnicodeEncodeError as u:
                continue
f.close()#没有用with open记得手动关闭

#分词并保存
#!/usr/bin/python
# -*- coding: latin-1 -*-
import os, sys
import requests
import json
import re 
import jieba
import openpyxl
from openpyxl import Workbook
import xlwt
import importlib
importlib.reload(sys)
from xlwt import Workbook

file=open("C:/python/wang/contents.txt",'r')#歌词文本的路径
lyric_str=file.read()#读取文本
seg=jieba.cut(lyric_str)#jieba分词
word_list=[]#存储词语
word_dict={}#字典，存词语及其个数
for each in seg:
   #print(each+' ')
    if len(each)>1:#过滤长度为1的词
            word_list.append(each)#加入到词语列表中

for index in word_list:#遍历词语列表
        if index in word_dict:
            word_dict[index]+=1#根据字典键访问键值，如果该键在字典中，则其值+1
        else:
            word_dict[index]=1#如果键不在字典中，则设置其键值为1

sorted(word_dict.items(),key=lambda e:e[1],reverse=True)#True表示按升序排列

fc=open("C:/python/wang/fenci.txt",'w')#创建文本fenci来存分词的结果
for item in word_dict.items():
        #print(item)
        fc.write(item[0]+str(item[1])+'\n')#将分词词频输出到txt文本中

    #将分词和词频输出到excel中
file=Workbook()
table=file.add_sheet('data')#添加列

ldata = []
num = [a for a in word_dict]
num.sort()#共几个词

for item in num:#频次
            ldata.append(str(word_dict[item]))#次数

for i in range(1000):
            table.write(i,0,num[i])
            table.write(i,1,ldata[i])
file.save("C:/python/wang/fenci.xls")#表格结果存入电脑

#制作扇形图
#!/usr/bin/python
# -*- coding: latin-1 -*-
import sys
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import csv
from wordcloud import WordCloud
plt.rcParams['font.sans-serif']=['Simhei'] #解决中文显示问题，目前只知道黑体可行
plt.rcParams['axes.unicode_minus']=False #解决负数坐标显示问题

filename='C:/python/wang/fenci.csv'#分词的excel结果路径
with open(filename) as f:
	reader=csv.reader(f)#按行读取
	header_row=next(reader)
	highs={}#字典，存储频次跟词语
	for row in reader:
		if int(row[1])>20:#选取出现频次大于20的词语
			highs[row[0]]=row[1]
	print(highs)
#绘制扇形图
labels=[]
labels=highs.keys()#数
data=[]
data=highs.values()#值
plt.pie(data,labels=labels,autopct='%1.2f%%')
plt.axis('equal')#加equal使图形呈现圆形，否则呈椭圆
plt.legend()
plt.show()

#制作词云
#!/usr/bin/python
# -*- coding: latin-1 -*-
import os
from collections import Counter
import  jieba
from wordcloud import WordCloud
import matplotlib.pyplot as plt
from scipy.misc import imread
from pylab import mpl

#第一步：定义停用词库
def stopwordslist(filepath):
    stopwords = [line.strip() for line in open(filepath, 'r').readlines()]
    return stopwords
stopwords=stopwordslist('C:/python/wang/contents.txt')

# 第二步：读取文件，分词，生成all_words列表，用停用词检查后生成新的all_words_new
all_words=[]
outstr = ''
filename="C:/python/wang/contents.txt"#歌词所在文件路径
with open(filename) as f:#读取歌词
        lyrics=f.read()
        data=jieba.cut(lyrics)#分词
        all_words.extend(set(data))#存词
for word in all_words:
    if word not in stopwords:
        if word != '\t':
            outstr += word
            outstr += " "
all_words_new= outstr.split(" ") #转成列表
#第三步：对all_words中的词计数，并按照词频排序
count=Counter(all_words_new)
result=sorted(count.items(), key=lambda x: x[1], reverse=True)
#print(result)
#第四步，词云显示
#将频率变成字典
word_dic=dict(count.items())
# 使matplotlib模块能显示中文
mpl.rcParams['font.sans-serif'] = ['SimHei'] # 指定默认字体
mpl.rcParams['axes.unicode_minus'] = False # 解决保存图像是负号'-'显示为方块的问题
color_mask=imread('C:/python/123.jpg') #背景图
cloud=WordCloud(
    font_path='C:\Windows\Fonts\SimHei.TTF',#字体路径
    width=600,
    height=480,
    background_color='white',#背景色
    mask=color_mask,
    max_words=350,
    max_font_size=150)
world_cloud=cloud.fit_words(word_dic)
world_cloud.to_file('C:/python/wang/karry.jpg')#将生成的词云存入电脑
plt.imshow(world_cloud)

#情绪分析
#--coding:GBK -- 
import jieba
import numpy as np
import os,sys
import pandas as pd
from snownlp import SnowNLP
import csv
import matplotlib.pyplot as plt
kk='C:/python/fenci.csv'#读取分词后的表格
score1=0#存正面词
score2=0#存负面词
with open(kk) as f:
	reader=csv.reader(f)#按行读取
	for row in reader:
		header_row=next(reader)
		s=SnowNLP(row[0])
		score=s.sentiments#利用snownpl给词语的特性打分
		if score>=0.6:
			score1+=1
		else:
		    score2+=1
print(score1,score2)
num=[score1,score2]#数
data=["positive","negative"]#名
plt.pie(num,labels=data,autopct='%1.2f%%')
plt.axis('equal')#加equal使图形呈现圆形，否则呈椭圆
plt.legend()
plt.show()
