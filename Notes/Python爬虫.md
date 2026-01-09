---
date created: 2025-12-27 23:03:49
date modified: 2026-01-07 21:34:49
---

# Python 爬虫

## 1 学习资料 
- [python爬虫基础](https://www.bilibili.com/video/BV1d54y1g7db?spm_id_from=333.788.videopod.episodes&vd_source=98a57e80832443af94457bc03e77e21a&p=5)
- [大厂Python程序猿](https://www.bilibili.com/video/BV1SLN4zREdh?spm_id_from=333.788.player.switch&vd_source=98a57e80832443af94457bc03e77e21a&p=7)

## 2 网页类型：
- 静态网页  
![[Pasted image 20251228003607.jpg]]
- 前端JS渲染网页  
![[Pasted image 20251228003740.jpg]]

## 3 HTTP协议
-  ![[Pasted image 20251228011018.jpg]]  
    ![[Pasted image 20251227230842.jpg]]
- ![[Pasted image 20251228011133.jpg]]

## 4 HTML基础
- [08:11](https://www.bilibili.com/video/BV1SLN4zREdh/?p=11&t=491.282523#t=08:11.28) CSS选择器

## 5 REQUESTS模块
- GET方法  
```python
import requests

#爬取百度的页面源代码
url = "http://www.baidu.com"

resp = requests.get(url)
resp.encoding = "utf-8"
print(resp.text) #拿到页面源代码
```

- 配置`User-Agent`  



	```python
	import requests

	content = input('请输入你要检索的内容:')
	url = f"https://www.sogou.com/web?query={content}"
	headers = {
		#添加一个请求头信息. UA
		"User-Agent" : "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36"
	}
	
	#处理一个小小的反爬  
	resp = requests.get(url, headers = headers)
	print(resp.text)
	print(resp.request.headers) # 可以查看到请求头信息	
	```	
	
- 默认请求头  
	![[Pasted image 20251228013410.png]]
	
- GET多参数 [08:47](https://www.bilibili.com/video/BV1SLN4zREdh/?p=11&t=527.503442#t=08:47.50) 
	```python
	import requests
	
	url = "https://movie.douban.com/chart" 
	params = {
		"type": "13",
		"interval id": "100:90",
		"action": "",
		"start": "0",
		"limit": "20"
	}
	headers = {
		"User-Agent" : "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36"
	}
	
	resp = requests.get(url, params = params, headers = headers) # 处理一个小小的反爬
	print(resp.text)	
	```

- POST方法
```python
import requests

url = "https://fanyi.baidu.com/sug"
data = {
	"kw" : input("请输入一个单词")
}

resp = requests.post(url, data = data)
print(resp.text) # 拿到的是文本字符串
print(resp.json()) # 此时拿到的直接是json数据
```

## 6 数据解析
[00:30](https://www.bilibili.com/video/BV1SLN4zREdh/?p=11&t=30.605531#t=30.61)
### 6.1 re正则表达式(重要)  
- 练习网站：`https://tool.oschina.net/regex`  
- 重点：`元字符`,`量词`,`贪婪与惰性匹配`  
![[Pasted image 20251228214411.jpg]]  
![[Pasted image 20251228214647.jpg]]
> `\s`不匹配换行符  
- python的`RE`模块  
```python
import re

result = re.findall("a", "我是一个 abcdeafg")
print(result)
print("-------------------")

iter = re.finditer(r"\d+", "我今年18岁, 我有200000000块")
for item in iter:
    print(item.group()) # 从匹配到的结果中拿到数据
print("-------------------")

searchRet = re.search(r"\d+", "我叫周杰伦, 今年32岁, 我的班级是3年2班")
print(searchRet.group()) # search只返回第一个匹配到的结果
print("-------------------")

#预加载，提前把正则对象加载完毕 
obj = re.compile(r"\d+")
result = obj.findall("我叫周杰伦, 今年32岁, 我的班级是5年4班")
print(result)
```

使用正则表达式分组提取数据  
```python
import re

# 提取数据时使用命名捕获组（group name）
S = """
<div class='西游记'><span id='10010'>中国联通</span></div>
<div class='西游记'><span id='10086'>中国移动</span></div>
"""

pattern = re.compile(r"<span id='(?P<id>\d+)'>(?P<name>.*?)</span>")

for match in pattern.finditer(S):
    item_id = match.group("id")
    print(item_id)
    name = match.group("name")
    print(name)
    print("-----")
```

- 实战1：使用RE爬取`豆瓣电影TOP250数据`
```python
import requests
import re

url = "https://movie.douban.com/top250"
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36"
}

resp = requests.get(url, headers=headers)
resp.encoding = 'utf-8'  # 解决乱码问题.
pageSource = resp.text
# print(pageSource)

obj = re.compile(r'<div class="item">.*?<span class="title">(?P<name>.*?)</span>.*?<p>.*?导演: (?P<dao>.*?)&nbsp;.*?<br>.*?(?P<year>.*?)&nbsp;.*?<span class="rating_num" property="v:average">(?P<score>.*?)</span>.*?<span>(?P<num>.*?)人评价</span>', re.S)
f = open("top250.csv", mode="w", encoding='utf-8')

result = obj.finditer(pageSource)
for item in result:
    name = item.group("name")
    dao = item.group("dao")
    year = item.group("year").strip()  # 去掉字符串左右两端的空白
    score = item.group("score")
    num = item.group("num")
    print(f"{name},{dao},{year},{score},{num}")
    f.write(f"{name},{dao},{year},{score},{num}\n")  # 如果觉着low. 可以更换成csv模块, 进行数据写入

f.close()
resp.close()
print("豆瓣TOP250提取完毕.")
```

- 实战2：使用RE爬取`电影天堂-2021必看热片数据`(非首页直接可获取到目标数据)
```python
import requests
import re

url = "https://www.dy2018.com/"
resp = requests.get(url)
resp.encoding = "gbk"
# print(resp.text)

# 1. 提取2025必看热片部分的HTML代码, 注意当前年份
obj1 = re.compile(r"2025必看热片.*?<ul>(?P<html>.*?)</ul>", re.S)
result1 = obj1.search(resp.text)
html = result1.group("html")
# print(html)

# 2. 提取a标签中的href的值
obj2 = re.compile(r"<li><a href='(?P<href>.*?)' title")
result2 = obj2.finditer(html)

# 3. 访问href提取电影名称和下载地址
obj3 = re.compile(r'<div id="Zoom">.*?片　　名　(?P<movie>.*?)<br />.*?<td style="WORD-WRAP: break-word" bgcolor="#fdfddf"><a href="(?P<download>.*?)">', re.S)
for item in result2:
    # print(item.group("href"))
    # 拼接子页面的url|
    child_url = url.strip("/") + item.group("href")
    child_resp = requests.get(child_url)
    child_resp.encoding = 'gbk'
    # print(child_resp.text)

    result3 = obj3.search(child_resp.text)
    movie = result3.group("movie")
    download = result3.group("download")
    print(movie, download)
    print("---------")
```

### 6.2 bs4解析
### 6.3 xpath解析(重要)
### 6.4 pyquery解析
