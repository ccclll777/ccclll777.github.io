---
title: 爬取boss直聘岗位数据进行数据分析
date: 2020-04-27 10:50:34
categories: 
- 数据分析
tags:
- python
- 爬虫
---
爬取boss直聘岗位数据，按照数据分析的流程，分析数据，从大量数据中得出结论
<!-- more -->
## 项目地址
https://github.com/ccclll777/bosszp
**如果觉得有用，请点个star**
## 爬取数据
首先使用python（使用resquest库，beautifulsoup4库）爬取了boss直聘网上的数据，我们根据城市和岗位名称作为关键字进行搜索，然后爬取了近3w条数据，包括岗位名称，岗位的薪水，招聘公司，公司的标签，学历要求，工作经验要求，公司所在城市。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427103918277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
boss直聘有反爬措施，需要自己在网页登陆后，获取cookie，加到请求的header中
```python
import random
import re

import pymysql
import requests
from bs4 import BeautifulSoup

cookie = '网页登陆后，复制cookie到这里'
conn = pymysql.connect(
    host="xxxx",
    user="xxx", password="xxx",
    database="xxx",
    charset="utf8")
cursor = conn.cursor()

base_url = "https://www.zhipin.com"

job_type = ["Java", "PHP", "web前端", "iOS", "Android", "算法工程师", "数据分析师", "数据架构师", "数据挖掘", "人工智能", " 机器学习", "深度学习"]
city_name = ["北京", "上海", "广州", "深圳", "杭州", "天津", "西安", "苏州", "武汉", "厦门", "长沙", "成都", "郑州", "重庆"]
city_num = ["c101010100", "c101020100", "c101280100", "c101280600", "c101210100", "c101030100", "c101110100",
            "c101190400", "c101200100", "c101230200",
            "c101250100", "c101270100", "c101180100", "c101040100"]


def get_user_agent():
    user_list = [
        "Opera/9.80 (X11; Linux i686; Ubuntu/14.10) Presto/2.12.388 Version/12.16",
        "Opera/9.80 (Windows NT 6.0) Presto/2.12.388 Version/12.14",
        "Mozilla/5.0 (Windows NT 6.0; rv:2.0) Gecko/20100101 Firefox/4.0 Opera 12.14",
        "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.0) Opera 12.14",
        "Opera/12.80 (Windows NT 5.1; U; en) Presto/2.10.289 Version/12.02",
        "Opera/9.80 (Windows NT 6.1; U; es-ES) Presto/2.9.181 Version/12.00",
        "Opera/9.80 (Windows NT 5.1; U; zh-sg) Presto/2.9.181 Version/12.00",
        "Opera/12.0(Windows NT 5.2;U;en)Presto/22.9.168 Version/12.00",
        "Opera/12.0(Windows NT 5.1;U;en)Presto/22.9.168 Version/12.00",
        "Mozilla/5.0 (Windows NT 5.1) Gecko/20100101 Firefox/14.0 Opera/12.0",
        "Opera/9.80 (Windows NT 6.1; WOW64; U; pt) Presto/2.10.229 Version/11.62",
        "Opera/9.80 (Windows NT 6.0; U; pl) Presto/2.10.229 Version/11.62",
        "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; fr) Presto/2.9.168 Version/11.52",
        "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; de) Presto/2.9.168 Version/11.52",
        "Opera/9.80 (Windows NT 5.1; U; en) Presto/2.9.168 Version/11.51",
        "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; de) Opera 11.51",
        "Opera/9.80 (X11; Linux x86_64; U; fr) Presto/2.9.168 Version/11.50",
        "Opera/9.80 (X11; Linux i686; U; hu) Presto/2.9.168 Version/11.50",
        "Opera/9.80 (X11; Linux i686; U; ru) Presto/2.8.131 Version/11.11",
        "Opera/9.80 (X11; Linux i686; U; es-ES) Presto/2.8.131 Version/11.11",
        "Mozilla/5.0 (Windows NT 5.1; U; en; rv:1.8.1) Gecko/20061208 Firefox/5.0 Opera 11.11",
        "Opera/9.80 (X11; Linux x86_64; U; bg) Presto/2.8.131 Version/11.10",
        "Opera/9.80 (Windows NT 6.0; U; en) Presto/2.8.99 Version/11.10",
        "Opera/9.80 (Windows NT 5.1; U; zh-tw) Presto/2.8.131 Version/11.10",
        "Opera/9.80 (Windows NT 6.1; Opera Tablet/15165; U; en) Presto/2.8.149 Version/11.1",
        "Opera/9.80 (X11; Linux x86_64; U; Ubuntu/10.10 (maverick); pl) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (X11; Linux i686; U; ja) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (X11; Linux i686; U; fr) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 6.1; U; zh-tw) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 6.1; U; zh-cn) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 6.1; U; sv) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 6.1; U; en-US) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 6.1; U; cs) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 6.0; U; pl) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 5.2; U; ru) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 5.1; U;) Presto/2.7.62 Version/11.01",
        "Opera/9.80 (Windows NT 5.1; U; cs) Presto/2.7.62 Version/11.01",
        "Mozilla/5.0 (Windows NT 6.1; U; nl; rv:1.9.1.6) Gecko/20091201 Firefox/3.5.6 Opera 11.01",
        "Mozilla/5.0 (Windows NT 6.1; U; de; rv:1.9.1.6) Gecko/20091201 Firefox/3.5.6 Opera 11.01",
        "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; de) Opera 11.01",
        "Opera/9.80 (X11; Linux x86_64; U; pl) Presto/2.7.62 Version/11.00",
        "Opera/9.80 (X11; Linux i686; U; it) Presto/2.7.62 Version/11.00",
        "Opera/9.80 (Windows NT 6.1; U; zh-cn) Presto/2.6.37 Version/11.00",
        "Opera/9.80 (Windows NT 6.1; U; pl) Presto/2.7.62 Version/11.00",
        "Opera/9.80 (Windows NT 6.1; U; ko) Presto/2.7.62 Version/11.00",
        "Opera/9.80 (Windows NT 6.1; U; fi) Presto/2.7.62 Version/11.00",
        "Opera/9.80 (Windows NT 6.1; U; en-GB) Presto/2.7.62 Version/11.00",
        "Opera/9.80 (Windows NT 6.1 x64; U; en) Presto/2.7.62 Version/11.00",
        "Opera/9.80 (Windows NT 6.0; U; en) Presto/2.7.39 Version/11.00"
    ]
    user_agent = random.choice(user_list)
    return user_agent


def get_page(url):
    headers = {
        'user-agent': "Opera/9.80 (X11; Linux i686; Ubuntu/14.10) Presto/2.12.388 Version/12.16",
        'accept': "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
        'accept-encoding': "gzip, deflate, br",
        'accept-language': "zh-CN,zh;q=0.9,en;q=0.8",
        'cookie': cookie,
        'cache-control': "no-cache",
        'referer': 'https://www.zhipin.com/?ka=header-home'

    }

    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            response.encoding = response.apparent_encoding
            return response.text


    except requests.ConnectionError as e:
        print('Error', e.args)


def translate(str):
    line = str.strip()  # 处理前进行相关的处理，包括转换成Unicode等
    pattern = re.compile('[^\u4e00-\u9fa50-9]')  # 中文的编码范围是：\u4e00到\u9fa5
    zh = " ".join(pattern.split(line)).strip()
    outStr = zh  # 经过相关处理后得到中文的文本
    return outStr


def get_job(url, conn, cursor, city_name_x):
    html = get_page(url)
    soup = BeautifulSoup(html, 'lxml')
    job_all = soup.find_all('div', class_="job-primary")
    if (job_all == []):
        print("cookie已过期")
    for job in job_all:
        try:
            # 职位名
            job_title = job.find('span', class_="job-name").string
            # 薪资
            job_salary = job.find('span', class_="red").string
            # 职位标签
            job_tag1 = job.p.text
            # 公司
            job_company = job.find('div', class_="company-text").a.text
            # 招聘详情页链接
            job_url = base_url + job.find('div', class_="company-text").a.attrs['href']
            # 公司标签
            job_tag2 = job.find('div', class_="company-text").p.text
            # 发布时间
            job_time = job.find('span', class_="job-pub-time").text

            job_acquire = translate(str(job.find('p')))
            print(job_title, job_salary, job_tag1, job_company, job_url, job_tag2, job_time, job_acquire, city_name_x)
            store_data(job_title, job_salary, job_tag1, job_company, job_url, job_tag2, job_time, job_acquire,
                       city_name_x, conn, cursor, )

        except Exception as e:
            print(str(e))


def store_data(job_title1, job_salary1, job_lable1, job_company1, job_url1, job_company_tag1, job_time1, job_acquire1,
               company_city1, conn, cursor):
    try:
        cursor.execute(
            'insert into job_data (job_title,job_salary,job_lable,job_company,job_url,job_company_tag,job_time,job_acquire,company_city) '
            'values ("{}","{}","{}","{}","{}","{}","{}","{}","{}")'.format(job_title1, job_salary1, job_lable1,
                                                                           job_company1, job_url1,
                                                                           job_company_tag1, job_time1,
                                                                           job_acquire1, company_city1))
    except:
        print("存入数据库失败")

    conn.commit()


city_no = 5  # 城市编号
page = str(1)
key = job_type[11]
url = base_url + "/" + "c101190100" + "/?" + "query=" + key + "&page=" + page + "&ka=page-" + page
print(url)
get_job(url=url, conn=conn, cursor=cursor, city_name_x="南京")

cursor.close()
conn.close()

```
爬取的数据直接存入了数据库

## 数据清洗
首先，需要根据岗位名称，将实习的岗位去掉，因为实习的岗位工资可能较低，并且可能工资是按天结算，会影响最后的统计结果。所以我们将岗位名称中带有实习的岗位从数据中去掉了。

其次，需要根据岗位名称提取出对应的岗位类别，并且将工资的单位变成“元”，并且将最低工资，最高工资分开，如果类似一年14薪这种，需要将每个月的工资乘相对应的比例，换算成一年12薪，方便我们计算（虽然税钱会增高）。
清洗后的数据：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104213107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
## 数据统计
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104227874.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
我们将数据库中的数据，读取出来写如csv文件，用pandas库进行数据分析。然后用pyecharts进行图像的绘制

```python
import pandas as pd
data = pd.read_csv('job_data_clean_price.csv')
```

**（1）各大城市互联网行业薪资水平**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104244684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
我计算了以上几个城市的最低平均工资，最高平均工资，和平均工资。
统计发现，北京，上海，杭州，深圳的互联网工资水平最高。除北京工资最高之外，没想到的是杭州竟然能超过上海，达到第二高的水平。
苏州，虽然说互联网行业发达，可能由于物价较低的缘故，工资水平没有很高。

```python
from  pyecharts import Bar
average_salary = data.groupby('company_city')['job_ave_salary'].mean()#平均工资
average_low_salary = data.groupby('company_city')['job_low_salary'].mean()#最低平均工资
average_high_salary = data.groupby('company_city')['job_high_salary'].mean()#最高平均工资
x = average_salary.reset_index()['company_city'].tolist()
y1 = average_salary.reset_index()['job_ave_salary'].tolist()
y2 = average_low_salary.reset_index()['job_low_salary'].tolist()
y3 = average_high_salary.reset_index()['job_high_salary'].tolist()
print(x)

bar = Bar(title = "互联网行业发达城市平均工资",width = 1000)
bar.add(name = "各城市平均工资", x_axis = x, y_axis = y1)
bar.add(name = "各城市最低平均工资", x_axis = x, y_axis = y2,is_xaxis_boundarygap =True)
bar.add(name = "各城市最高平均工资", x_axis = x, y_axis = y3,is_xaxis_boundarygap =True)

# 导出绘图html文件，可直接用浏览器打开
bar.render('各城市工资.html')
bar
```

**（2）互联网行业各岗位工资统计**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104300382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
注：这里的算法工程师，包括了各个方面算法岗位的数据，不是针对某个方向的算法岗

我们发现与人工智能有关的岗位工资较高，而数据架构师的学历要求普遍比较高，所以工资高属于正常现象。有关数据分析，数据挖掘的岗位工资也和人工智能不相上下。而较为普通的 Android开发 ios开发 java  php  web前端岗位的工资 在1w元左右，属于较低水平。

```python
job_average_salary = data.groupby('job_type')['job_ave_salary'].mean()#平均工资
job_average_low_salary = data.groupby('job_type')['job_low_salary'].mean()#最低平均工资
job_average_high_salary = data.groupby('job_type')['job_high_salary'].mean()#最高平均工资
x = job_average_salary.reset_index()['job_type'].tolist()
print(x)
y1 = job_average_salary.reset_index()['job_ave_salary'].tolist()
print(y1)
y2 = job_average_low_salary.reset_index()['job_low_salary'].tolist()
print(y2)
y3 = job_average_high_salary.reset_index()['job_high_salary'].tolist()
print(y3)


bar = Bar(title = "互联网各行业平均工资",width = 1000)
bar.add(name = "各岗位平均工资", x_axis = x, y_axis = y1)
bar.add(name = "各岗位最低平均工资", x_axis = x, y_axis = y2,is_xaxis_boundarygap =True)
bar.add(name = "各岗位最高平均工资", x_axis = x, y_axis = y3,is_xaxis_boundarygap =True)

# 导出绘图html文件，可直接用浏览器打开
bar.render('各岗位工资.html')
bar
```

**（3）各个学历的平均工资**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104322623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
统计发现 专科，本科到硕士的工资水平逐步升高，但是如果博士可以毕业，工资水平直线猛增，基本能达到4w的月薪左右，从本科到硕士的提升也比较明显，所以有能力的人还是继续深造为好。

```python
job_education_average_salary = data.groupby('job_education')['job_ave_salary'].mean()#平均工资
job_education_average_low_salary = data.groupby('job_education')['job_low_salary'].mean()#最低平均工资
job_education_average_high_salary = data.groupby('job_education')['job_high_salary'].mean()#最高平均工资
x = job_education_average_salary.reset_index()['job_education'].tolist()
print(x)
y1 = job_education_average_salary.reset_index()['job_ave_salary'].tolist()
print(y1)
y2 = job_education_average_low_salary.reset_index()['job_low_salary'].tolist()
print(y2)
y3 = job_education_average_high_salary.reset_index()['job_high_salary'].tolist()
print(y3)


bar = Bar(title = "互联网行业各个学历平均工资",width = 1000)
bar.add(name = "各学历平均工资", x_axis = x, y_axis = y1)
bar.add(name = "各学历最低平均工资", x_axis = x, y_axis = y2,is_xaxis_boundarygap =True)
bar.add(name = "各学历最高平均工资", x_axis = x, y_axis = y3,is_xaxis_boundarygap =True)

# 导出绘图html文件，可直接用浏览器打开
bar.render('各学历工资.html')
bar
```

（4）不同工作经验要求的工资
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104339614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

应届生的工资水平还是比较低的，如果能有，随着工作经验的升高，尤其是工作五年以上的，工资水平将会直线上升，但是应届生和1年之内工作经验的学生，差距不大。

```python
job_experience_average_salary = data.groupby('job_experience')['job_ave_salary'].mean()#平均工资
job_experience_average_low_salary = data.groupby('job_experience')['job_low_salary'].mean()#最低平均工资
job_experience_average_high_salary = data.groupby('job_experience')['job_high_salary'].mean()#最高平均工资
x = job_experience_average_salary.reset_index()['job_experience'].tolist()
print(x)
y1 = job_experience_average_salary.reset_index()['job_ave_salary'].tolist()
print(y1)
y2 = job_experience_average_low_salary.reset_index()['job_low_salary'].tolist()
print(y2)
y3 = job_experience_average_high_salary.reset_index()['job_high_salary'].tolist()
print(y3)


bar = Bar(title = "互联网行业不同工作经验平均工资",width = 1000)
bar.add(name = "不同工作经验平均工资", x_axis = x, y_axis = y1)
bar.add(name = "不同工作经验最低平均工资", x_axis = x, y_axis = y2,is_xaxis_boundarygap =True)
bar.add(name = "不同工作经验最高平均工资", x_axis = x, y_axis = y3,is_xaxis_boundarygap =True)

# 导出绘图html文件，可直接用浏览器打开
bar.render('不同工作经验工资.html')
bar
```

（5）各个经验段要求的占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104353558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
我们发现，1-3年工作经验的要求人数最多。应届生的要求比例较少，可能招聘网上，校招的岗位较少吧。5-10年的工作经验的人还是比较少的，互联网行业1-5年工作经验的人还是占大多数的。

```python
from pyecharts import Pie
experience_count = data.groupby('job_experience')['job_ave_salary'].count()#平均工资
print(experience_count.reset_index()['job_experience'].tolist())
print(experience_count.reset_index()['job_ave_salary'].tolist())
attr = experience_count.reset_index()['job_experience'].tolist()
value = experience_count.reset_index()['job_ave_salary'].tolist()

# 初始化图表通用属性
pie = Pie(title = "工作经验要求占比",
          title_pos = 'center', # 标题居中
          title_top = 'bottom', # 标题在底部
          title_color = '#0000ff', # 标题颜色设置为蓝色，256位rgb格式
          background_color = "#aee", # 设置背景颜色，16位rgb格式
          width = 600,height = 420)

pie.add("", attr, value, is_label_show=True)
pie.render('各个工作经验要求占比.html')
pie
```

**（6）学历要求占比**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104406937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
在我看来，学历要求占比，可能在某种程度上反应了，当前社会上的学历占比，我们发现本科生还是占大多数的，所以本科毕业就就业的压力还是很大的，更体现了，如果有能力读研还是要继续深造的好，毕竟硕士只有百分之10的占比。
博士作为稀缺人才，需求还是很大的。

```python
experience_count = data.groupby('job_education')['job_ave_salary'].count()#平均工资
print(experience_count.reset_index()['job_education'].tolist())
print(experience_count.reset_index()['job_ave_salary'].tolist())
attr = experience_count.reset_index()['job_education'].tolist()
value = experience_count.reset_index()['job_ave_salary'].tolist()

# 初始化图表通用属性
pie = Pie(title = "学历要求占比",
          title_pos = 'center', # 标题居中
          title_top = 'bottom', # 标题在底部
          title_color = '#0000ff', # 标题颜色设置为蓝色，256位rgb格式
          background_color = "#aee", # 设置背景颜色，16位rgb格式
          width = 600,height = 420)

pie.add("", attr, value, is_label_show=True)
pie.render('学历要求占比.html')
pie
```

**（7）知名大厂的平均工资待遇**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104421487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

我们发现，华为的薪资待遇尽然是最高的，而百度由于近几年发展缓慢，甚至倒退，工资水平竟然达到了最低，这个工资水平可能和城市有关，而且也不能说明未来的提升空间，所以可能没有什么价值。

```python
data_company = pd.read_csv('job_data_count_company.csv')
company_ave_salary = data_company.groupby('job_company')['job_ave_salary'].mean()
company_low_salary = data_company.groupby('job_company')['job_low_salary'].mean()
company_high_salary = data_company.groupby('job_company')['job_high_salary'].mean()


x = company_ave_salary.reset_index()['job_company'].tolist()
print(x)
y1 = company_ave_salary.reset_index()['job_ave_salary'].tolist()
print(y1)
y2 = company_low_salary.reset_index()['job_low_salary'].tolist()
y3 = company_high_salary.reset_index()['job_high_salary'].tolist()

bar = Bar(title = "大厂工资待遇",width = 1000)
bar.add(name = "大厂平均工资待遇", x_axis = x, y_axis = y1)
bar.add(name = "大厂最低平均工资待遇", x_axis = x, y_axis = y2,is_xaxis_boundarygap =True)
bar.add(name = "大厂最高平均工资待遇", x_axis = x, y_axis = y3,is_xaxis_boundarygap =True)

bar.render('大厂工资待遇.html')
bar
```

**（8）知名大厂的招聘数量以及比例**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104440100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104443196.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

```python
data_company = pd.read_csv('job_data_count_company.csv')
company_count = data_company.groupby('job_company')['job_ave_salary'].count()


x = company_count.reset_index()['job_company'].tolist()
print(x)
y1 = company_count.reset_index()['job_ave_salary'].tolist()
print(y1)



bar = Bar(title = "大厂招聘数量",width = 1000)
bar.add(name = "大厂招聘数量", x_axis = x, y_axis = y1)
bar.render('大厂招聘数量.html')
bar

# 初始化图表通用属性
pie = Pie(title = "大厂招聘比例",
          title_pos = 'center', # 标题居中
          title_top = 'bottom', # 标题在底部
          title_color = '#0000ff', # 标题颜色设置为蓝色，256位rgb格式
          background_color = "#aee", # 设置背景颜色，16位rgb格式
          width = 600,height = 420)

pie.add("", x, y1, is_label_show=True)
pie.render('大厂招聘比例.html')
pie


```

**（9）各岗位学历占比**
	①java学历占比
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104501997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
②PHP学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104508714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
③Android学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104516685.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
④iOS学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104546906.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
⑤web前端学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104557674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
⑥算法工程师学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104608923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
⑦数据架构师学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104616677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
⑧数据挖掘学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104625518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
⑨数据分析学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104632450.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
⑩人工智能学历要求占比
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427104645909.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
根据统计，在boss直聘上爬取的岗位的学历要求，以本科为主，但是不同岗位的比例不同。相比较而言Java、PHP、安卓、iOS、web前端等岗位对学历要求较低，主要为本科与大专。而算法工程师、数据挖掘、数据架构师、人工智能等岗位对学历要求较高，具体表现为大专要求的岗位大量减少，硕士要求占比猛然加大，甚至出现博士要求，可以看出这些岗位对于技术要求较高。所以要想从事这些岗位，深造可能是更好的选择。