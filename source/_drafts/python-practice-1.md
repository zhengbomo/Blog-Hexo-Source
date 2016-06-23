---
title: python-practice-1
date: 2016-06-04 15:22:56
updated: 2016-06-04 15:22:56
categories:
tags:
---

python实战-爬取糗事百科
在掌握了python的基础之后，我们可以来练习一些简单的实战操作，下面演示如何爬取糗事百科最热门的100条段子中，并找出最好笑的10条段子

## 准备：
1、下面假定已经掌握了python的基本语法
2、假定已经掌握了urllib，urllib2，BeautifulSoup库的基本使用，参见
3、python操作sqlite数据库

## 分析：
1. 由于我们不需要爬去页面内的链接，所以只需要爬取单个网页内容，分析，入库即可

先定义几个方法辅助
```python
# 下载html
def download_html(url):
  try:
    header = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.122 Safari/537.36 SE 2.X MetaSr 1.0'}
    request = urllib2.Request(url, None, header)
    response = urllib2.urlopen(request)
    html = response.read()
    return html
  except Exception, e:
    if hasattr(e, 'code') && hasattr(e, 'reason'):
      print 'download_html error, code: %s, reason: %s' % (e.code, e.reason)
    else
      print e
    return None

# 分析页面中的所有段子
def get_jokes(html):
  soup = BeautifulSoup(html, 'html.parser', from_encoding="utf8")
  # 获取内容节点
  content_div = soup.find('div', id='content-left')
  jokes_nodes = content_div.find_all('div', recursive=False, class_='article block untagged mb15')

  jokes = []
  for joke_node in jokes_nodes:
    if hasattr(joke_node, 'id'):
      joke = get_joke(joke_node)
      jokes.append(joke)

  return jokes

# 从节点分析出段子内容
def get_joke(node):
  author_node = node.find('div', recursive=False, class_='author clearfix')
  aa = author_node.find_all('a', recursive=False)

  # 处理用户信息
  if len(aa) == 2:
    avatar_url = aa[0].get('href')
    img_node = aa[0].find('img', recursive=False)
    author_avatar = img_node.get('src')
    author = aa[1].text.strip()
  else:
    # 匿名用户
    spans = author_node.find_all('span', recursive=False)
    img_node = spans[0].find('img', recursive=False)
    author_avatar = img_node.get('src')
    author = spans[1].text.strip()

  # 处理内容
  content_node = node.find('div', recursive=False, class_='content')
  content = content_node.text.strip()

  # 处理好笑数与评论数
  stats_node = node.find('div', recursive=False, class_='stats')

  vote_node = stats_node.find('span', recursive=False, class_='stats-vote')
  vote_node = vote_node.find('i', recursive=False, class_='number')
  vote = int(vote_node.text)

  comments_node = stats_node.find('span', recursive=False, class_='stats-comments')
  comments = int(comments_node.find('i', recursive=True, class_='number').text)

  author = author
  author_avatar = author_avatar
  avatar_url = 'http://www.qiushibaike.com' + avatar_url
  content = content
  vote = vote
  comments = comments

  return (author, author_avatar, avatar_url, content, vote, comments)

```

核心函数定义完成，现在把上面方法整合起来
```python
# 获取100也热门段子
def get_joke_100_page:
  jokes = []
  for i in range(1, 101):
    url = 'http://www.qiushibaike.com/text/page/%d' % page
    html = download_html(url)
    page_jokes = get_jokes(html)
    jokes.extend(page_jokes)

  # 前得到100页的段子
  funny_jokes = get_most_funny_jokes(jokes)

  # 输出
  output_joke_list(funny_jokes)

# 获取好笑数最多的count条段子
def get_most_funny_jokes(jokes, count):
  return sorted(jokes, key=lambda joke:joke[4], reverse=True)[1, count]

# 输出段子列表
def output_joke_list(jokes):
  for joke in funny_jokes:
    print 'vote = %s, content = %s' % (joke[4], joke[3])
```

如果我们还需要把结果保存到数据库的话，使用下面函数入库和打印，我们定义一个类用于处理数据库相关的操作
```python
class JokeDb(object):
  """数据库管理 """
  def __init__(self, db_path):
    super(, self).__init__()
    self.db_path = db_path
    self.connect = sqlite3.connect(self.db_path)
    create_joke_table()

  def create_joke_table(self):
    cursor = connect.cursor()

    # 判断表是否存在
    sql = 'SELECT COUNT(*) FROM sqlite_master where type=\'table\' and name=\'joke\''
    cursor.execute(sql)
    count = cursor.fetchone()
    if count[0] == 0:
      # 表不存在，则创建
      sql = '''CREATE TABLE joke (id INTEGER PRIMARY KEY AUTOINCREMENT,
                                  author text,
                                  author_avatar text,
                                  avatar_url text,
                                  content text,
                                  vote INTEGER,
                                  comments INTEGER
                                  )
      '''
      cursor.execute(sql)
      connect.commit()
      cursor.close()

  def insert_joke(self, joke):
    sql = 'INSERT INTO joke (author, author_avatar, avatar_url, content, vote,  comments) values (%s, %s, %s, %s, %d, %d)' % (joke.author, joke.author_avatar， joke.avatar_url, joke.content, joke.vote, joke.comments)
    cursor = connect.cursor()
    cursor.execute(sql)
    connect.commit()
    cursor.close()

  def insert_jokes(self, jokes):
    cursor = connect.cursor()
    for joke in jokes:
      sql = 'INSERT INTO joke (author, author_avatar, avatar_url, content, vote,  comments) values (%s, %s, %s, %s, %d, %d)' % (joke.author, joke.author_avatar， joke.avatar_url, joke.content, joke.vote, joke.comments)
      cursor.execute(sql)
    connect.commit()
    cursor.close()

  def get_most_funny_jokes:
    #查询结果集使用Row对象，可以获取的信息更全
    connect.row_factory = sqlite3.Row
    sql = 'SELECT * FROM joke as j ORDER BY j.vote LIMIT 10'
    cursor = connect.cursor()
    cursor.execute(sql)
    values = cursor.fetchall()
    for value in values:
      print value['content']
    cursor.close()
```
