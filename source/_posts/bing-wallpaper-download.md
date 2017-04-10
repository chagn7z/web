---
title: 必应的每日壁纸下载
---

# 分析

- 使用浏览器打开 `cn.bing.com` ，按下 `F12` 进入调试界面，并切换到网络选项页。
- 刷新页面，捕获到浏览器的请求。找到以下请求：
	`http://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&nc=1491830740475&pid=hp`
	其返回信息为：
	`{"images":[{"startdate":"20170409","fullstartdate":"201704091600","enddate":"20170410","url":"/az/hprichbg/rb/ArcticFoxSibs_ZH-CN7417451993_1920x1080.jpg","urlbase":"/az/hprichbg/rb/ArcticFoxSibs_ZH-CN7417451993","copyright":"弗兰格尔岛上的北极狐，俄罗斯 (© Owen Newman/Getty Images)","copyrightlink":"/search?q=%e5%8c%97%e6%9e%81%e7%8b%90%e7%8b%b8&form=hpcapt&mkt=zh-cn","quiz":"/search?q=Bing+homepage+quiz&filters=WQOskey:%22HPQuiz_20170409_ArcticFoxSibs%22&FORM=HPQUIZ","wp":false,"hsh":"faacad1474cf10a8d5f6f5faf75f037e","drk":1,"top":1,"bot":1,"hs":[]}]}`
	
	判断为获取图片地址的请求，分析后发现通过配置不同的参数就能得到最终的图片地址：
		- idx -> 已今天为起始的天数索引，向前增加，取值范围 `0-8`
		- nc -> 十三位的时间戳，标示当前日期时间
		- n -> 返回的数据长度，即从起始日期开始的天数长度

# 下载

使用 `python` 构造请求下载即可。

```
import time
import requests

IMG_PATH = '/home/usr/bing'

if __name__ == '__main__':
	nc  = int(time.time() * 1000)
	n   = 8
	idx = 0
	post_url = 'http://cn.bing.com/HPImageArchive.aspx?format=js&idx=%d&n=%d&nc=%d&pid=hp' % (idx, n, nc)

	res = requests.get(post_url)
	urls = []
	if res:
		img_json = res.json()
		for row in img_json['images']:
			name = row['url'].rfind('/rb/')
			urls.append([row['startdate'], row['url'].split('/')[-1], 'http://cn.bing.com' + row['url']])

	for img in urls:
		res = requests.get(img[-1])
		with open(IMG_PATH+'/['+img[0]+']'+img[1], 'w') as fd:
			fd.write(res.content)

```


