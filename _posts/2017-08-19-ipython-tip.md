---                                                                                                
layout: post                                                                                        
title: "구글 학술검색 인용순 정렬하기"                
featured-img: python
categories: [scholar,google,python]                                                                       
mathjax: true
---           
# 구글 학술검색 인용순으로 정렬하기 Python

> 인용순으로 검색하는 기능이 없나 검색했다가  
> 영어로 된 사이트에서 인용횟수 누르면 된다고 하는 글을 대충 읽고  
> "오! 된다" 이러고 포스팅을 하고 있던 내가 한심하다.  
> 스크린샷을 보고 뭔가 이상한 걸 느끼고 두시간 정도 삽질을 해서 방법을 찾았다.  
> `python`을 이용해서 크롤링을 한다음에 정렬을 한 뒤 엑셀 파일을 만드는 방식인 것 같다.
> 
> ~사실 다른 학술지DB를 사용하는게 골치 안 아프고 좋다...~~  
> 근데 다른 DB들 검색기능이 하도 그지같아서 그냥 이 코드를 수정해서 썼다.~  
> **수정이 귀찮으면 마지막 부분에 있는 코드와 사용법을 참조하여 사용하자.**

## 사전지식

`python`, `pip`, `git hub` 이 세가지가 필요하다. 간단하게 만들 수도 있을것 같긴한데, 파이썬으로 응용 프로그램 만드는 방법을 몰라서 안타깝다.

## 다운로드

Git repository : [https://github.com/WittmannF/sort-google-scholar](https://github.com/WittmannF/sort-google-scholar)

`git clone https://github.com/WittmannF/sort-google-scholar`

`readme.md`에 대충 설명은 나와있는데, 예전 버전으로 만들어서 그런지 몇가지를 수정 해줘야한다.

## 코드수정

일단 코드 전문을 올린다.

{% highlight python  %}
# -*- coding: utf-8 -*-
"""
This code creates a database with a list of publications data from Google 
Scholar.
The data acquired from GS is Title, Citations, Links and Rank.
It is useful for finding relevant papers by sorting by the number of citations
This example will look for the top 100 papers related to the keyword 
'non intrusive load monitoring', so that you can rank them by the number of citations

As output this program will plot the number of citations in the Y axis and the 
rank of the result in the X axis. It also, optionally, export the database to
a .csv file.

Before using it, please update the initial variables

"""

import requests
from bs4 import BeautifulSoup
import matplotlib.pyplot as plt
import pandas as pd


def get_citations(content):
    out = 0
    for char in range(0,len(content)):
        if content[char:char+9] == 'Cited by ':
            init = char+9                          
            for end in range(init+1,init+6):
                if content[end] == '<':
                    break
            out = content[init:end]
    return int(out)
    
def get_year(content):
    for char in range(0,len(content)):
        if content[char] == '-':
            out = content[char-5:char-1]
    if not out.isdigit():
        out = 0
    return int(out)

def get_author(content):
    for char in range(0,len(content)):
        if content[char] == '-':
            out = content[2:char-1]
            break
    return out

# Update these variables according to your requirement
keyword = "'non intrusive load monitoring'" # the double quote will look for the exact keyword,
                                            # the simple quote will also look for similar keywords
number_of_results = 100 # number of results to look for on Google Scholar
save_database = False # choose if you would like to save the database to .csv
path = 'C:/_wittmann/nilm_100_exact_author.csv' # path to save the data

# Start new session
session = requests.Session()

# Variables
links = list()
title = list()
citations = list()
year = list()
rank = list()
author = list()
rank.append(0) # initialization necessary for incremental purposes

# Get content from 1000 URLs
for n in range(0, number_of_results, 10):    
    url = 'https://scholar.google.com/scholar?start='+str(n)+'&q='+keyword.replace(' ','+')
    page = session.get(url)
    c = page.content
    
    # Create parser
    soup = BeautifulSoup(c, 'html.parser')
    
    # Get stuff
    mydivs = soup.findAll("div", { "class" : "gs_r" })
    
    for div in mydivs:
        try:
            links.append(div.find('h3').find('a').get('href'))
        except: # catch *all* exceptions
            links.append('Look manually at: https://scholar.google.com/scholar?start='+str(n)+'&q=non+intrusive+load+monitoring')
        
        try:
            title.append(div.find('h3').find('a').text)
        except: 
            title.append('Could not catch title')
            
        citations.append(get_citations(str(div.format_string)))
        year.append(get_year(div.find('div',{'class' : 'gs_a'}).text))
        author.append(get_author(div.find('div',{'class' : 'gs_a'}).text))
        rank.append(rank[-1]+1)

# Create a dataset and sort by the number of citations
data = pd.DataFrame(zip(author, title, citations, year, links), index = rank[1:], 
                    columns=['Author', 'Title', 'Citations', 'Year', 'Source'])
data.index.name = 'Rank'

data_ranked = data.sort(columns='Citations', ascending=False)
print data_ranked

# Plot by citation number
plt.plot(rank[1:],citations,'*')
plt.ylabel('Number of Citations')
plt.xlabel('Rank of the keyword on Google Scholar')
plt.title('Keyword: '+keyword)

# Save results
if save_database:
    data_ranked.to_csv(path, encoding='utf-8') # Change the path
        
{% endhighlight %}

### pip install

일단 `pip install`을 통해서  
`pandas`, `requests`, `BeautifulSoup`을 설치 해준다.  
`pyplot`은 원래 설치되어있던가?

### 코드 수정

이부분이 귀찮은 분들은 그냥 마지막에 있는 내코드를 가져다가 쓰면 된다. 깃허브에 있는 코드를 수정한건데 라이센스가 따로 명시가 안되있는거같다. 원작자도 엄청 오래 수정안한거 보면 신경 안쓰는 거 같다.

#### 키워드 변경

그리고 검색할 키워드를 `line 51`에 있는 `keyword = "'non intrusive load monitoring'"` 이 부분의 큰따옴표 안에 넣어준다.

**됐다!** 이러고 돌려보면 에러가 난다 .

#### `sort`함수 수정

`pandas`가 버전이 올라가서 `sort`함수가 `sort_values`함수로 바뀌었다. `line 102`의 `data_ranked = data.sort(columns='Citations', ascending=False)` 부분을 `data_ranked = data.sort_values('Citations', ascending=False)` 로 수정해준다.

이러면 일단 돌아가긴 한다. 몇가지 더 바꾸도록 하자.  
  

#### 인자 전달을 통한 실행 단순화

현재는 파일을 열어서 키워드를 바꿔주고 커맨드라인에서 `python sortby_citations_google_scholar.py`을 통해서 실행을 하면 결과가 나오는 형식이다.

내가 미쳤다고 검색할때마다 이걸 열어서 변수를 바꾸고 검색하겠는가. `python filename.py "keyword"`를 통해서 커맨드라인으로 검색할 수 있게 바꾸자. 코드의 앞쪽 부분에 아래와 같이 `sys`를 `import`해준다.

~~~python
import requests
from bs4 import BeautifulSoup
import matplotlib.pyplot as plt
import pandas as pd
import sys #modified
~~~

그리고 `line 51`부근을 아래와 같이 수정하자.

~~~python
# Update these variables according to your requirement
keyword = sys.argv[1] # the double quote will look for the exact keyword,
                      # the simple quote will also     look for similar keywords
~~~

이러면 `python filename.py "keyword"`를 통해서 검색을 할 수 있게 된다.

#### CSV파일 저장

CSV파일을 통해서 피인용수로 정렬된 결과를 얻을 수 있는데 현재는 CSV파일 저장이 되지 않고 화면에만 출력이 된다.  
또한 파일이 저장되는 곳과 파일명이 맘에 안든다. 코드를 수정하여 현재 폴더에 저장되고 파일명에 키워드 명이 들어가도록 수정해보자.

원래 파일의 `line 53`을 보면 아래와 같은 코드가 있다.

~~~python
number_of_results = 100 # number of results to look for on Google Scholar
save_database = False # choose if you would like to save the database to .csv
path = 'C:/_wittmann/nilm_100_exact_author.csv' # path to save the data
~~~

이를 다음과 같이 바꿔주자

~~~python
number_of_results = 100 # number of results to look for on Google Scholar
save_database = True # choose if you would like to save the database to .csv
path = './'+keyword+'_result_sorted_by_citation.csv' # path to save the data
~~~

이를 통해서 현재 폴더에 `keyword__result_sorted_by_citation.csv`저장 가능하다.

## 최종 소스코드과 사용법

### 최종 소스코드

~~~python
#my_search.py
# -*- coding: utf-8 -*-
"""
This code creates a database with a list of publications data from Google 
Scholar.
The data acquired from GS is Title, Citations, Links and Rank.
It is useful for finding relevant papers by sorting by the number of citations
This example will look for the top 100 papers related to the keyword 
'non intrusive load monitoring', so that you can rank them by the number of citations

As output this program will plot the number of citations in the Y axis and the 
rank of the result in the X axis. It also, optionally, export the database to
a .csv file.

Before using it, please update the initial variables

"""
import sys
import requests
from bs4 import BeautifulSoup
import matplotlib.pyplot as plt
import pandas as pd


def get_citations(content):
    out = 0
    for char in range(0,len(content)):
        if content[char:char+9] == 'Cited by ':
            init = char+9                          
            for end in range(init+1,init+6):
                if content[end] == '<':
                    break
            out = content[init:end]
    return int(out)
    
def get_year(content):
    for char in range(0,len(content)):
        if content[char] == '-':
            out = content[char-5:char-1]
    if not out.isdigit():
        out = 0
    return int(out)

def get_author(content):
    for char in range(0,len(content)):
        if content[char] == '-':
            out = content[2:char-1]
            break
    return out

# Update these variables according to your requirement
keyword = sys.argv[1] # the double quote will look for the exact keyword,
                                            # the simple quote will also look for similar keywords
number_of_results = 100 # number of results to look for on Google Scholar
save_database = True # choose if you would like to save the database to .csv
path = './'+keyword+'_result_sorted_by_citation.csv' # path to save the data

# Start new session
session = requests.Session():

# Variables
links = list()
title = list()
citations = list()
year = list()
rank = list()
author = list()
rank.append(0) # initialization necessary for incremental purposes

# Get content from 1000 URLs
for n in range(0, number_of_results, 10):    
    url = 'https://scholar.google.com/scholar?start='+str(n)+'&q='+keyword.replace(' ','+')
    page = session.get(url)
    c = page.content
    
    # Create parser
    soup = BeautifulSoup(c, 'html.parser')
    
    # Get stuff
    mydivs = soup.findAll("div", { "class" : "gs_r" })
    
    for div in mydivs:
        try:
            links.append(div.find('h3').find('a').get('href'))
        except: # catch *all* exceptions
            links.append('Look manually at: https://scholar.google.com/scholar?start='+str(n)+'&q=non+intrusive+load+monitoring')
        
        try:
            title.append(div.find('h3').find('a').text)
        except: 
            title.append('Could not catch title')
            
        citations.append(get_citations(str(div.format_string)))
        year.append(get_year(div.find('div',{'class' : 'gs_a'}).text))
        author.append(get_author(div.find('div',{'class' : 'gs_a'}).text))
        rank.append(rank[-1]+1)

# Create a dataset and sort by the number of citations
data = pd.DataFrame(zip(author, title, citations, year, links), index = rank[1:], 
                    columns=['Author', 'Title', 'Citations', 'Year', 'Source'])
data.index.name = 'Rank'

data_ranked = data.sort_values('Citations', ascending=False)
print data_ranked

# Plot by citation number
plt.plot(rank[1:],citations,'*')
plt.ylabel('Number of Citations')
plt.xlabel('Rank of the keyword on Google Scholar')
plt.title('Keyword: '+keyword)

# Save results
if save_database:
    data_ranked.to_csv(path, encoding='utf-8') # Change the path
       
~~~

### 사용법(Usage)

Usage : `python filename.py "Your_keyword"`  
Result : `./Your_keyword_result_sorted_by_citation.csv`  
실행하면 5초 정도 기다려야한다.  
또한 `python 2.7`로 실행해야한다. `print`함수가 `python 3.0`에서는 바껴서 에러가 난다.

#### 결과 파일

필자는 "lexicon tree decoder"에 대해서 검색했고 결과는 아래와 같다.

