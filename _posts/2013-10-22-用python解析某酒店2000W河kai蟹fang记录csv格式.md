---
layout : post
category : python
tags : [python]
comments : no
---
近段时间较火的2000W和谐记录（这个只是某酒店的，虽然很不全，但我认识的人已经有躺枪的了，说明数据真实），已经有好心人提供了csv版本的下载，由于河蟹的关系，本博客不提供数据的下载，仅提供csv文件的一种处理方法作为技术交流。

##问题背景
因为csv格式的文件还是好大，用文本编辑器虽然能打开，但机器比较渣，处理速度有限，所以考虑自己编程解决问题。文章很大程度上参考了cao liu 上面的一个技术贴，不引用了（大家懂得）。

##解决方案

###search.py
{% highlight py linenos %}
#!ecoding=utf-8
import csv
import sys

s = 0
filelist = ['1-200W.csv','200W-400W.csv','400W-600W.csv','600W-800W.csv','800W-1000W.csv','1000W-1200W.csv','1200W-1400W.csv','1400W-1600W.csv','1600w-1800w.csv','1800w-2000w.csv','5000.csv']
fpw = open(sys.argv[1] + '.res', 'w')
for i in filelist:
	fp = open(i,'rU')
	reader = csv.reader(fp)
	for line in reader:
		if(line != []):
			#匹配条件
			if(line[0].decode('utf-8')==sys.argv[1].decode('utf-8')):
			for item in line:
				print item.decode('utf-8'),
				fpw.write(item)
				fpw.write(',')
			s += 1
			print
			fpw.write('\n')
	fp.close()
	print "end",i
	print "total=",s
fpw.close() 
{% endhighlight %}

这样，只要执行命令 `python search.py 张三` 即可输出名为张三的人的全部河(kai)蟹(fang)记录，并输出到文件 `张三.res` 中

想一次性查多人？我们写个批处理脚本吧！
###batch.py
{% highlight py linenos %} 
import os

fpr = open('namelist.txt', 'r')

item = fpr.readline()
while item!='':
	print item,
	os.system("python search.py " + item)
	item=fpr.readline()

print "done" 
{% endhighlight %} 

这样，只要在 namelist.txt 中写下待查人的名单，运行脚本即可进行批量查询，并输出到XX.res 中。

###namelist.txt
{% highlight c linenos%} 
张三
李四
王五
刘翔
姚明 
{% endhighlight %}  

done.

