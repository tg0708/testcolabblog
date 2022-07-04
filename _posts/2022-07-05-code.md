# Warnings


```python
import warnings
warnings.filterwarnings('ignore')
```

# Bs4

## 패턴

### 기본 패턴

~~~ python
# find
import requests
from bs4 import BeautifulSoup
res = requests.get("크롤링 할 주소")
soup = BeautifulSoup(res.content, "html.parser")
mydata = soup.find('title')
print(mydata.text)
~~~

~~~ python 
# select
import requests
from bs4 import BeautifulSoup
res = requests.get('https://davelee-fun.github.io/blog/crawl_test_css.html')
soup = BeautifulSoup(res.content,
'html.parser')
items = soup.select('li')
for item in items:
    print (item.get_text())
~~~

### 응답 페이지 확인

~~~ python
import requests
from bs4 import BeautifulSoup

res = requests.get('https://davelee-fun.github.io/xxx')
if res.status_code != 200:
    print ('페이지 없음')
else:
    soup = BeautifulSoup(res.content, 'html.parser')

    data = soup.select('h4.card-text')
    for item in data:
        print (item.get_text())
~~~

### 여러 페이지 

~~~ python
import requests
from bs4 import BeautifulSoup

for page_num in range(10):
    if page_num == 0:
        res = requests.get('https://davelee-fun.github.io/')
    else:
        res = requests.get('https://davelee-fun.github.io/page' + str(page_num + 1))
    soup = BeautifulSoup(res.content, 'html.parser')

    data = soup.select('h4.card-text')
    for item in data:
        print (item.get_text().strip())
~~~

### 엑셀 저장


```python
! pip install openpyxl
```

    Requirement already satisfied: openpyxl in c:\users\tgkang\anaconda3\lib\site-packages (3.0.9)
    Requirement already satisfied: et-xmlfile in c:\users\tgkang\anaconda3\lib\site-packages (from openpyxl) (1.1.0)


~~~ python
import openpyxl

def write_excel_template(filename, sheetname, listdata):
    excel_file = openpyxl.Workbook()
    excel_sheet = excel_file.active
    excel_sheet.column_dimensions['A'].width = 100
    excel_sheet.column_dimensions['B'].width = 20
    
    if sheetname != '':
        excel_sheet.title = sheetname
    
    for item in listdata:
        excel_sheet.append(item)
    excel_file.save(filename)
    excel_file.close()
~~~

~~~ python
import requests
from bs4 import BeautifulSoup

product_lists = list()

for page_num in range(10):
    if page_num == 0:
        res = requests.get('https://davelee-fun.github.io/')
    else:
        res = requests.get('https://davelee-fun.github.io/page' + str(page_num + 1))
    soup = BeautifulSoup(res.content, 'html.parser')

    data = soup.select('div.card')
    for item in data:
        product_name = item.select_one('div.card-body h4.card-text')
        product_date = item.select_one('div.wrapfooter span.post-date')
        product_info = [product_name.get_text().strip(), product_date.get_text()] # 리스트
        product_lists.append(product_info)
write_excel_template('tmp.xlsx', '상품정보', product_lists)
~~~

## find

~~~ python
data = soup.find('p', class_='cssstyle') # 태그, 클래스
data = soup.find('p', 'cssstyle') # 태그, 클래스
data = soup.find('p', attrs = {'align': 'center'}) # 태그, 속성
data = soup.find(id='body') # 아이디
data = soup.find('h3','tit_view')
data = soup.find('div', 'layer_util layer_summary')
~~~

## select

~~~ python
items = soup.select('.course') # 클래스
items = soup.select('#start') # 아이디
items = soup.select('td[valign="top"]') # 태그, 특정 속성
items = soup.findAll("td", {"valign" : re.compile(r".*")}) # 정규 표현식
items = soup.select('li.course.paid') # 태그, 클래스1, 클래스2
items = soup.select('html body h1') # 하위 태그
items = soup.select('ul > li') # 직계 하위 태그
items = soup.select('ul#hobby_course_list li.course') # 태그, 아이디, 하위 태그, 클래스
item = soup.select_one('ul#dev_course_list > li.course.paid')

~~~

## 활용예제

~~~ python
# G마켓 베스트 상품
url = "http://corners.gmarket.co.kr/Bestsellers?viewType=G&groupCode=G06"
res = requests.get(url)
if res.status_code != 200:
    print("응답 없음")
else :
    soup = BeautifulSoup(res.content, 'html.parser')
    
bestlist = soup.select('.best-list li')
for idx, item in enumerate(bestlist):
    item_list = item.select_one('div .itemname').text.strip()
    price_list = item.select_one('div .s-price span').text.strip()
    print(idx+1,item_list," - ", price_list)
~~~

# Selenium

## 기본 패턴


```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time
# 드라이버 생성
# chromedriver 설치된 경로를 정확히 기재해야 함
chromedriver = r'C:\\Users\\tgkang\\Documents\\크롤링2\\103\\chromedriver.exe'

driver = webdriver.Chrome(service=Service(chromedriver))
```

## Autoinstaller 

~~~ python
from selenium import webdriver
import chromedriver_autoinstaller
import os

from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

# Check if chrome driver is installed or not
chrome_ver = chromedriver_autoinstaller.get_chrome_version().split('.')[0]
driver_path = f'./{chrome_ver}/chromedriver.exe'
if os.path.exists(driver_path):
    print(f"chrom driver is insatlled : {driver_path}")
else:
    print(f"install the chrome driver(ver : {chrome_ver})")
    chromedriver_autoinstaller.install(True)
driver = webdriver.Chrome(service=Service(driver_path))
~~~

## Options

~~~ python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import warnings
import chromedriver_autoinstaller
warnings.filterwarnings('ignore')

options = Options()

options.add_argument('headless') # headless 모드
options.add_argument('window-size=1920*1080')
options.add_argument('--start-maximized') # 최대화
options.add_argument('--start-fullscreen') # 풀스크린 코드

options.add_argument('--mute-audio') #브라우저에 음소거 옵션을 적용합니다.
options.add_argument('incognito') #시크릿 모드의 브라우저가 실행됩니다.

options.add_argument('disable-gpu')
options.add_argument('User-Agent:Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36')
options.add_argument('lang=ko-KR')

# 자동화 문구 제거
options.add_experimental_option("useAutomationExtension", False)
options.add_experimental_option("excludeSwitches", ['enable-automation'])

# 디버거 모드 - 안됨
options.add_experimental_option("debuggerAddress", "127.0.0.1:9222")

driver_path = './103/chromedriver.exe'
driver = webdriver.Chrome(executable_path= driver_path, options= options)

~~~

## Driver

### Assert

~~~ python
# Selenium은 웹테스트를 위한 프레임워크로 다음과 같은 방식으로 웹테스트를 자동으로 진행함 (참고)
print (driver.title)
assert "Teddy" in driver.title
~~~

### Size

~~~ python
# 웹페이지 전체 사이즈
driver.maximize_window()
# 웹페이지 전체 사이즈
driver.minimize_window()
# 웹페이지 사이즈 조절
driver.set_window_size(1000,1000)
# 풀스크린
driver.fullscreen_window()
~~~

### Handle

~~~ python
# 현재 핸들중인 창 목록 조회
driver.window_handles
driver.window_handles[0] # 첫번째 창
driver.window_handles[1] # 두번째 창
driver.window_handles[-1] # 가장 최근에 열린창
~~~


### Switch

~~~ python
# switch
driver.switch_to.window(driver.window_handles[1])
# iframe으로 이동
driver.switch_to.frame('iframe name')
# 상위 iframe으로 이동
driver.switch_to.parent_frame()
# 초기 frame으로 이동
driver.switch_to.default_content()
~~~

### Screenshot

~~~ python
# 해당 엘리멘트 스크린샷 후 저장
element.screenshot("gd.png") # 특정 태그가 차지하는 만큼 스크린 샷
# body로 지정시 전체 스크린 샷 
element = driver.find_elements(By.TAG_NAME, "body")
element.screenshot("test.png")
~~~


### URL

~~~ python
# 현재 url 가져오기
driver.current_url
~~~

###  Title

~~~ python
# 웹페이지 타이틀 가져오기
driver.title
~~~

### Clear text

~~~ python
# input 텍스트 초기화
element.clear()
~~~

### Javascript

~~~ python
# user agent 가져오기
driver.execute_script('return navigator.userAgent')
driver.execute_script("window.scrollTo(0,Y)") # Y까지 스크롤 내리기
driver.execute_script("window.scrollTo(0, document.body.scrollHeight)") # 끝까지 스크롤
~~~


## Find_element

### ID, CSS_SELECTOR

~~~ python
elem = driver.find_element(By.ID,"navbarMediumish")
elems = driver.find_elements(By.CSS_SELECTOR, "div.card-body > h4")
~~~

### Get_attribute

~~~ python
# 특정 attribute 
elem = driver.find_elements(By.TAG_NAME, "meta")
for item in elem:
    data = item.get_attribute('content')
    print(data)
~~~

### Image

~~~ python
# 이미지 URL 추출
elems = driver.find_elements(By.CSS_SELECTOR, "div.wrapthumbnail img")
sources = list()
for elem in elems:
    sources.append(elem.get_attribute('src'))
~~~

~~~ python
# 이미지 다운 받기
from urllib.request import urlretrieve
image_path = r"C:\Users\tgkang\Documents\크롤링2\103\\"
for index, source in enumerate(sources):
    urlretrieve(source,  image_path + "image" + str(index) + "." + source.split(".")[-1])
~~~

### Xpath

   - / : 절대경로를 나타냄 (예: /html/body/div/div)
   - // : 문서내에서 검색 (예: //h1 -> h1 태그를 가진 데이터를 선택)
   - //*[@href] : href 속성이 있는 모든 태그 선택
   - //a[@href='http://google.com'] : a 태그의 href 속성에 http://google.com 속성값을 가진 모든 태그 선택 
   - (//a)[3] : 문서의 세 번째 링크 선택
   - (//table)[last()] : 문서의 마지막 테이블 선택
   - (//a)[position() < 3] : 문서의 처음 두 링크 선택
   - //div[@*] 속성이 하나라도 있는 div 태그 선택

~~~ python
# h1 태그 중 첫번째 태그 가져오기
title = driver.find_element(By.XPATH, "//h1")
# href 속성 모두 선택
datas = driver.find_elements(By.XPATH, '//*[@href]')
# ID = begin 인 속성 모두 찾기
datas = driver.find_elements(By.XPATH, '//*[@id="begin"]')
# class 의 값이 skill-name 인 div 태그들 중에, HTML 코드 위에서 세번째 해당하는 div 태그 선택 
datas = driver.find_elements(By.XPATH, "//div[@class='skill-name']")
# class값이 best-list 이고 그 아래 ul li a 태그 
datas = driver.find_elements(By.XPATH, "//div[@class='best-list']/ul/li/a")

# 첫 번째 데이터 선택
item = driver.find_element(By.XPATH, "(//tr)[position()=1]")
# 3 번째 보다 작은 번째 선택
item = driver.find_element(By.XPATH, "(//tr)[position()<3]")
# 마지막 데이터 선택
item = driver.find_element(By.XPATH, "(//tr)[last()]")

# 속성을 하나 이상 가진 p 태그
item = driver.find_element_by_xpath("//p[@*]")
# 다중 선택
elem = driver.find_element_by_xpath("//*[contains(@class, 'course') and contains(@class, 'paid')]")
~~~

### Title tag  예외


 ~~~ python   
# css selector로 title을 선택해서 text를 뽑으면 나오지 않음
elem = driver.find_element(By.CSS_SELECTOR, "title")
print ('text:', elem.text) # 가져오지 않음(# text는 보통 body안의 내용을 뽑을 때만)
print ('get_attribute:', elem.get_attribute('text')) # 가져와짐
print ('driver.title:', driver.title) 
# elem = driver.find_element_by_css_selector('h1')
elem = driver.find_element(By.CSS_SELECTOR, "h1")
print ('text:', elem.text) # 됨
print ('get_attribute:', elem.get_attribute('text')) # 될 것 처럼 보이지만 안된다.

 ~~~

## Send_key

~~~ python
from selenium.webdriver.common.keys import Keys
# 사용가능한 키 조회
dir(Keys)
# 키 이벤트 전송
elem.send_keys("error@error.com")
# 엔터 입력
elem.send_keys(Keys.RETURN)
~~~

# Scrapy

## 설치

~~~ python
# 윈도우/맥 공통
! pip install scrapy
~~~

~~~ python
# 윈도우에서 정상 설치 안될 시
! pip install --upgrade setuptools
! pip install pypiwin32
! pip install twisted[tls]

~~~

## 프로젝트 생성

~~~ python
# 프로젝트 생성
scrapy startproject ecommerce

# 크롤러 작성
scrapy genspider <크롤러이름> <크롤링주소>
scrapy genspider gmarket "www.gmarket.co.kr"

# 크롤러 실행
scrapy crawl gmarket
~~~


## Scrapy shell

~~~ python
# Scrapy shell 접속
scrapy shell "http://corners.gmarket.co.kr/Bestsellers"
exit # 종료

# response요청한 페이지 보기
view(response)
# response url 확인
response.url
~~~

## element

~~~ python
# css selector
response.css('head > title').get()
response.css('head > title').getall()
response.css('head > title::text').get()

response.css('div.best-list li > a::text').getall()
response.css('div.best-list li > a::text')[1].get()
~~~

~~~ python
# xpath
response.xpath('//div[@class="best-list"]/ul/li/a').getall()
response.xpath('//div[@class="best-list"]/ul/li/a/text()').getall()
~~~

~~~ python
# re 정규표신혁
# \n은 파이썬 3.0 이상은 한글도 포함 but reg 홈페이지에는 반영 안되어 있음
response.css('div.best-list li > a::text')[1].re('(\w+)')
response.xpath('//div[@class="best-list"]/ul/li/a/text()')[1].re('(\w+)')
~~~

# Excel

~~~ python
import win32com.client as win32
excel = win32.gencache.EnsureDispatch('Excel.Application')
wb = excel.Workbooks.Add()
ws = wb.Sheets("Sheet1")
rng = ws.Range("B2")
image = ws.Shapes.AddPicture(r"C:\Users\tgkang\Documents\크롤링2\103\image0.jpg", False, True, rng.Left, rng.Top, 100, 100)
excel.Visible=True
~~~

## openpyxl

### 패턴

~~~ python
# 엑셀 파일 읽기
import openpyxl

excel_file = openpyxl.load_workbook('tmp.xlsx')
excel_sheet = excel_file.active
# excel_sheet = excel_file.get_sheet_by_name('IT뉴스')

# 데이터 읽기
for row in excel_sheet.rows:
    print(row[0].value, row[1].value)

excel_file.close()
~~~

### Syntax

~~~ python
# 파일 가져오기
excel_file = openpyxl.load_workbook(r'C:\Users\tgkang\Documents\크롤링1\tmp.xlsx')
# 파일 생성
excel_file = openpyxl.Workbook()

# 활성화
excel_sheet = excel_file.active
# 시트 이름
excel_sheet.title = 'testsheet'
# 시트 선택
excel_sheet = excel_file["상품정보"]
# sheet name 확인하기
excel_file.sheetnames
                         
# 컬럼 크기 변경
excel_sheet.column_dimensions['A'].width = 100
excel_sheet.column_dimensions['B'].width = 20

# 데이터 입력
excel_sheet.append(["하이"])

# 파일 저장
excel_file.save("피카피카.xlsx")
# 파일 닫기
excel_file.close()
~~~

### 이미지


```python
import win32com.client as win32
excel = win32.gencache.EnsureDispatch('Excel.Application')
wb = excel.Workbooks.Add()
ws = wb.Sheets("Sheet1")
rng = ws.Range("B2")
image = ws.Shapes.AddPicture(r"C:\Users\tgkang\Documents\크롤링2\103\image0.jpg", False, True, rng.Left, rng.Top, 100, 100)
excel.Visible=True
```

# openAPI

## naver

### 기본 패턴

~~~ python
import requests
import pprint

client_id = 'BTMVavws8Is7jmVpUcSL'
client_secret = 'sDgiapg86l'

naver_open_api = 'https://openapi.naver.com/v1/search/shop.json?query=갤럭시노트10'
header_params = {"X-Naver-Client-Id":client_id, "X-Naver-Client-Secret":client_secret}
res = requests.get(naver_open_api, headers=header_params) # header에 아이디 넣어보냄

if res.status_code == 200:
    data = res.json()
    for index, item in enumerate(data['items']):
        print (index + 1, item['title'], item['link'])
else:
    print ("Error Code:", res.status_code)

~~~

### 엑셀 저장

~~~ python
import requests
import openpyxl

client_id = 'CgZgjTdS7F2naaLEhWRg'
client_secret = 'oCwEtEw08Y'
start, num = 1, 0 # 시작 페이지, 인덱스 설정

excel_file = openpyxl.Workbook()
excel_sheet = excel_file.active
excel_sheet.column_dimensions['B'].width = 100 # 셀 너비 조정
excel_sheet.column_dimensions['C'].width = 100
excel_sheet.append(['랭킹', '제목', '링크'])

for index in range(10):
    start_number = start + (index * 100)
    naver_open_api = 'https://openapi.naver.com/v1/search/shop.json?query=샤오미&display=100&start=' + str(start_number)
    header_params = {"X-Naver-Client-Id":client_id, "X-Naver-Client-Secret":client_secret}
    res = requests.get(naver_open_api, headers=header_params)
    if res.status_code == 200:
        data = res.json()
        for item in data['items']:
            num += 1
            excel_sheet.append([num, item['title'], item['link']])
    else:
        print ("Error Code:", res.status_code)

excel_file.save('IT.xlsx')
excel_file.close()
~~~

# 정규화

## 문자열 처리


```python
# 특정 문자 넣기
string = "12345"
comma = ','
comma.join(string)
```




    '1,2,3,4,5'




```python
# 특정 문자외 제거
string = "      9999999999999999(Dave)888888888888888888     "
string.strip(" 98()")        # 앞 뒤 괄호를 다 지움
```




    'Dave'




```python
# 문자열 나누기 - 인덱스
string = "Dave goes to Korea"
string.split()[3]
```




    'Korea'



## 정규식

<table>
    <thead>
        <tr style="font-size:1.2em">
            <th style="text-align:center">정규 표현식</th>
            <th style="text-align:center">축약 표현</th>
            <th style="text-align:left">사용 예</th>
        </tr>
    </thead>
    <tbody>
        <tr style="font-size:1.2em">
            <td style="text-align:center">[0-9]</td>
            <td style="text-align:center">\d</td>
            <td style="text-align:left">숫자를 찾음</td>
        </tr>
        <tr style="font-size:1.2em">
            <td style="text-align:center">[^0-9]</td>
            <td style="text-align:center">\D</td>
            <td style="text-align:left">숫자가 아닌 것을 찾음(텍스트, 특수 문자, white space(스페이스, 탭, 엔터 등등)를 찾을 때)</td>
        </tr>
        <tr style="font-size:1.2em">
            <td style="text-align:center">[ \t\n\r\f\v]</td>
            <td style="text-align:center">\s</td>
            <td style="text-align:left">white space(스페이스, 탭, 엔터 등등) 문자인 것을 찾음</td>
        </tr>
        <tr style="font-size:1.2em">
            <td style="text-align:center">[^ \t\n\r\f\v]</td>
            <td style="text-align:center">\S</td>
            <td style="text-align:left">white space(스페이스, 탭, 엔터 등등) 문자가 아닌 것을 찾음(텍스트, 특수 문자, 숫자를 찾을 때)</td>
        </tr>
        <tr style="font-size:1.2em">
            <td style="text-align:center">[A-Za-z0-9]</td>
            <td style="text-align:center">\w</td>
            <td style="text-align:left">문자, 숫자를 찾음</td>
        </tr>
        <tr style="font-size:1.2em">
            <td style="text-align:center">[^A-Za-z0-9]</td>
            <td style="text-align:cㅡenter">\W</td>
            <td style="text-align:left">문자, 숫자가 아닌 것을 찾음</td>
        </tr>
    </tbody>
</table>

### sub


```python
# re.sub
string = '(초급) - 강사가 실제 사용하는 자동 프로그램 소개 [2]'
import re
print(re.sub('\[[0-9]+\]', '', string))
print(re.sub('프로그램', '모듈', string)) # 찾아 바꾸기
```

    (초급) - 강사가 실제 사용하는 자동 프로그램 소개 
    (초급) - 강사가 실제 사용하는 자동 모듈 소개 [2]



```python
import re
pattern2 = re.compile('-')
subed = pattern2.sub('*', '801210-1011323')  # sub(바꿀문자열, 본래문자열)
subed
```




    '801210*1011323'



### pattern


```python
# pattern 적용
pattern = re.compile('D.A')  # .은 모든 숫자 및 문자
print(pattern.search("DAA")) # 해당
print(pattern.search("D1A")) # 해당
print(pattern.search("D00A")) # 해당 x
print(pattern.search("d0A")) # 해당 x
print(pattern.search("d0A D1A 0111")) # 해당
```

    <re.Match object; span=(0, 3), match='DAA'>
    <re.Match object; span=(0, 3), match='D1A'>
    None
    None
    <re.Match object; span=(4, 7), match='D1A'>



```python
# 특수문자 적용 \ 사용
pattern = re.compile('D\.A') # 정말 \ 기호 적용
print(pattern.search("D.A")) # 해당
print(pattern.search("DDA")) # 해당 x 
```

    <re.Match object; span=(0, 3), match='D.A'>
    None



```python
# 특수문자 적용 [] 사용
pattern = re.compile('D[.]A') # 정말 \ 기호 적용
print(pattern.search("D.A")) # 해당
print(pattern.search("DDA")) # 해당 x 
```

    <re.Match object; span=(0, 3), match='D.A'>
    None


### match 와 search 함수
* match : 문자열 처음부터 정규식과 매칭되는 패턴을 찾아서 리턴
* search : 문자열 전체를 검색해서 정규식과 매칭되는 패턴을 찾아서 리턴


```python
import re
pattern = re.compile('[a-z]+') 
matched = pattern.match('Dave')
print(matched)
searched = pattern.search("Dave")
print(searched)
```

    None
    <re.Match object; span=(1, 4), match='ave'>


### findall
정규표현식과 매칭되는 모든 문자열을 리스트 객체로 리턴함


```python
import re
pattern = re.compile('[a-z]+')
findalled = pattern.findall('Game of Life in Python')
print (findalled)
```

    ['ame', 'of', 'ife', 'in', 'ython']



```python
# findall 활용
import re
pattern = re.compile('[a-z]+')
findalled = pattern.findall('GAME')
if len(findalled) > 0:
    print ("정규표현식에 맞는 문자열이 존재함")
else:
    print ("정규표현식에 맞는 문자열이 존재하지 않음")
```

    정규표현식에 맞는 문자열이 존재하지 않음


### split


```python
import re
pattern2 = re.compile(':')
splited = pattern2.split('python:java')
splited
```




    ['python', 'java']



### ? , \* , +
* ? 는 앞 문자가 0번 또는 1번 표시되는 패턴 (없어도 되고, 한번 있어도 되는 패턴)
* \* 는 앞 문자가 0번 또는 그 이상 반복되는 패턴
* \+ 는 앞 문자가 1번 또는 그 이상 반복되는 패턴


```python
pattern = re.compile('D?A')   
print(pattern.search("A"))
print(pattern.search("DA"))
print(pattern.search("DDDDDDA"))
```

    <re.Match object; span=(0, 1), match='A'>
    <re.Match object; span=(0, 2), match='DA'>
    <re.Match object; span=(5, 7), match='DA'>



```python
pattern = re.compile('D*A')    
print(pattern.search("DA"))
print(pattern.search("DDDDDDDDDDDDDDDDDDDDDDDDDDDDA"))
```

    <re.Match object; span=(0, 1), match='A'>
    <re.Match object; span=(0, 2), match='DA'>
    <re.Match object; span=(0, 29), match='DDDDDDDDDDDDDDDDDDDDDDDDDDDDA'>


### {n}, {m,n}
* {n} : 앞 문자가 n 번 반복되는 패턴
* {m, n} : 앞 문자가 m 번 반복되는 패턴부터 n 번 반복되는 패턴까지


```python
# {n}
pattern = re.compile('AD{2}A')
print(pattern.search("ADA"))
print(pattern.search("ADDA"))
print(pattern.search("ADDDA"))
```

    None
    <re.Match object; span=(0, 4), match='ADDA'>
    None



```python
# {m,n}
pattern = re.compile('AD{2,6}A')    # {m,n} 은 붙여 써야 함 {m, n} 으로 쓰면 안됨(특이함)
print(pattern.search("ADDA"))
print(pattern.search("ADDDA"))
print(pattern.search("ADDDDDDA"))
```

    <re.Match object; span=(0, 4), match='ADDA'>
    <re.Match object; span=(0, 5), match='ADDDA'>
    <re.Match object; span=(0, 8), match='ADDDDDDA'>


### [ ] 괄호 내 문자
* 예:  [abc] 는 a, b, c 중 하나가 들어 있는 패턴을 말함


```python
pattern = re.compile('[abcdefgABCDEFG]')    
print(pattern.search("a1234"))
print(pattern.search("z1234")  )
```

    <re.Match object; span=(0, 1), match='a'>
    None


### [a-zA-Z0-9]


```python
pattern = re.compile('[a-zA-Z0-9]') 
print(pattern.search("1234---") )
print(pattern.search("---------------!@#!@$!$%#%%%#%%@$!$!---") )
```

### [^]


```python
pattern = re.compile('[^a-zA-Z0-9]') 
pattern.search("---------------!@#!@$!$%#%%%#%%@$!$!---") 
```




    <re.Match object; span=(0, 1), match='-'>




```python
pattern = re.compile('[^ \t\n\r\f\v]') 
pattern.search("-") 
```

### 가-힣


```python
pattern = re.compile('[가-힣]') 
pattern.search("안") 
```




    <re.Match object; span=(0, 1), match='안'>



## 활용예제


```python
# 주민 등록 번호
import openpyxl
work_book = openpyxl.load_workbook(r'C:\Users\tgkang\Documents\크롤링1\data_kr.xlsx')
work_sheet = work_book.active
for each_row in work_sheet.rows:
    print(re.sub('-[0-9]{7}', '-*******', each_row[1].value))

work_book.close()
```

    주민등록번호
    800215-*******
    821030-*******
    841230-*******
    790903-*******
    800125-*******
    820612-*******



```python
import requests
from bs4 import BeautifulSoup
```


```python

```

    1 [대우]대우 에어 써큘레이터DEF-KC1020스탠드선풍기 공기순환  -  35,900원
    2 [위닉스](공식인증점) 위닉스 뽀송 제습기 10리터 DXAE100-JWK  -  229,000원
    3 [대웅모닝컴](행사) 대웅모닝컴 14형 스탠드 선풍기 (신제품 입고)  -  29,900원
    4 [마이크로소프트]Xbox 충전식 배터리 +USB C타입 케이블  -  29,800원
    5 숲속바람 스탠드 선풍기 2022 신형 가정용선풍기 14형  -  29,900원
    6 [뽀송]공식인증점)위닉스 NEW 17L 제습기 DN3E170-LWK 1등급  -  379,000원
    7 [대웅모닝컴]대웅 3D 입체회전 리모컨 스탠드 써큘레이터 선풍기  -  39,800원
    8 [신일전자][신일] [화이트]  에어서큘레이터(SIF-FA800B)  -  109,200원
    9 [윈드피아](특가) 22년형 가정용 업소용 스탠드선풍기 WA-170  -  29,900원
    10 [르젠]내일도착르젠2세대 앱연동 BLDC 선풍기 LZEF-DC180 화이트  -  69,800원
    11 [대웅모닝컴]대웅 가정용 스탠드선풍기  키높이선풍기  -  28,800원
    12 [휘센]LG전자 휘센 제습기 DQ202PGUA (OK)  -  619,000원
    13 [보본]무선 캠핑 선풍기 탁상용 휴대용 캠핑용 타프팬  -  39,900원
    14 [신일전자]신일 기본형 선풍기 ---10% 다운로드 쿠폰---  -  57,900원
    15 [르젠]22년형+15%쿠폰) 르젠 APP연동 좌우회전 저소음 선풍기  -  79,800원
    16 [신일][신일] 2022년형 BLDC air S8 써큘레이터 (베이지/딥그린/라이트핑크)  -  134,400원
    17 [윈드피아]가정용 업소용 스탠드 리모컨 선풍기 인기상품 1700R  -  36,900원
    18 [한일]한일 2022년 신상품 35cm 기계식선풍기 EFe-G014  -  44,900원
    19 [위닉스](공식인증점) 위닉스 제습기 16리터 DO2E160-JWK  -  359,000원
    20 [뽀송]공식인증) 위닉스 1등급 제습기 16리터 DN2H160-IWK  -  359,000원
    21 [르젠]22년형+10%쿠폰) 르젠 리모컨 써큘레이터 저소음 선풍기 LZEF-AR03  -  49,800원
    22 [신일전자]프리미엄 BLDC 써큘레이터형 스탠드 선풍기 SIF-T14SH  -  139,900원
    23 [위닉스]10%쿠폰)인증점 위닉스 뽀송 제습기 10L DXAH100-JWK  -  219,000원
    24 [위닉스](인증점)위닉스 뽀송 제습기 16리터 DO2E160-JWK 1등급  -  359,000원
    25 [솔러스에어]1+1 2개 무선 선풍기 탁상용 미니 휴대용 캠핑 벽걸이  -  59,900원
    26 [뽀송]공식인증점)위닉스 NEW 17L 제습기 DN3E170-LWK 1등급  -  409,000원
    27 [삼성전자]삼성 윈도우핏 에어컨 길이 연장 키트 60cm  -  85,000원
    28 [엔보우]모노 탁상용 무선 휴대용 선풍기 2세대 1+1(할인 행사)  -  29,900원
    29 [통돌이]갤러리아 LG 일반 세탁기 TR12WL 12kg/화이트  -  383,000원
    30 [신일전자]신일 14형 좌석용 선풍기 SIF-14HKW 신일선풍기 4엽  -  62,000원
    31 [위닉스](공식인증점) 위닉스 뽀송 제습기 10L DXAH100-JWK  -  237,000원
    32 [필립스]PHILIPS 전기면도기 SkinIQ 5000 S5582/36 오션블루  -  159,000원
    33 [자우버]렌즈케어 200매 렌즈클리너 일회용 안경닦이 티슈  -  9,030원
    34 [숲속바람]Forest Wind 저소음 5엽 스탠드 선풍기 2022신상품  -  32,580원
    35 [신일전자][신일] (인기모델)[블랙]  에어서큘레이터(SIF-FB500A)  -  101,200원
    36 [제이닉스]제이닉스 14인치 스탠드 선풍기 가정용 JYF-KN4523  -  29,800원
    37 [샤오미]미밴드7 한글판 AOD탑재 공식수입 블랙/국내AS가능  -  59,800원
    38 [LG전자]LG전자 휘센 제습기 20L DQ202PGUA  -  699,000원
    39 [쿠쿠]본사직영 20L 전자레인지 CMW-A201DW 화이트  -  56,000원
    40 크레마 S (crema S) 블랙 / 화이트  -  199,000원
    41 [위닉스]DXSM170-IWK 위닉스 뽀송 제습기 17L / LK  -  372,750원
    42 [삼성전자]갤럭시 A53 5G SM-A536N 128G 자급제 _RM  -  507,130원
    43 [제이엠더블유]JMW M5001A PLUS BLDC 드라이기 거치대 세트+스타벅스  -  81,000원
    44 [로보락](혜택가149만원) 로보락 S7 MaxV Ultra 로봇청소기 울트라 자동세척  -  1,590,000원
    45 [위닉스][공식파트너]위닉스 뽀송 10리터 제습기 DXAE100-JWK  -  227,940원
    46 [레드울프]S22 노트20/노트10/S21/S20/S10/A53/A32 울트라 가죽  -  19,900원
    47 [삼성전자]삼성 갤럭시버즈2 블루투스 이어폰 SM-R177  -  94,640원
    48 [모이스]추가 10%쿠폰) 고급형 미니 제습기 저소음 소형 원룸 사무실 작은방  -  49,900원
    49 [삼성전자]삼성전자 DDR4 16G PC4-25600 (정품)-PL  -  76,890원
    50 [에코백스]쿠폰 139만원) (비밀쿠폰)에코백스 X1 옴니 로봇청소기 듀얼스테이션  -  1,590,000원
    51 S22 울트라 노트20/노트10/S21/S20/A53 A32 A23 가죽  -  19,900원
    52 [르젠][르젠] [빠른배송] 22년 신상 리모컨 써큘레이터 선풍기 LZEF-DC260  -  69,800원
    53 [캐리어]6평형 인버터 벽걸이 에어컨 ORCD061FAWWSD 22년최신형  -  529,000원
    54 [위니아]인증 위니아 가정용제습기 EDH8DNWB 8L  -  175,020원
    55 [유닉스]메탈티 무광블랙 1600W 헤어 드라이기 UN-A1610 접이식  -  23,900원
    56 유니맥스 UMF-R5314LDC 12단 BLDC모터 서큘형 선풍기  -  72,900원
    57 [테팔](10%중복할인) 블렌더 믹서기 블렌드포스 플러스 BL4258  -  42,900원
    58 CAXA UP 이영애 카사업 하트 페이스 탄력 기기 x 가히 에센스  -  189,000원
    59 [비달사순]비달사순 2000W 전문가용 헤어드라이기 VSD5129K  -  22,800원
    60 [일렉트로룩스]파워PRO 18V 무선청소기 ZB3411 (BEST 인기)  -  169,000원
    61 [르젠]22년형+15%쿠폰) 르젠 APP연동 입체회전 저소음 선풍기 LZDF-TR08  -  89,800원
    62 [쿠쿠]본사직영 CRP-CHP1010FD 10인용 IH전기압력 밥솥  -  249,000원
    63 [로지텍]로지텍코리아 정품 무선 마우스 MX MASTER 3S /무소음  -  139,000원
    64 [샤오미]MI 스마트 무선 선풍기 4세대 프로 PRO +  MI FAN2 PRO  -  111,300원
    65 [솔러스에어]1+1 2개 세트 휴대용 미니 무선 탁상용 캠핑 선풍기  -  54,900원
    66 [위닉스]공식인증점 2022년 신상품 제습기 17L DN3E170-LWK 1등급 뽀송  -  422,490원
    67 [휘센]LG전자 휘센 제습기 DQ162PGUA (OK)  -  509,400원
    68 [신일전자]신일 강화유리 무선 전기주전자 SEP-GW90  -  28,000원
    69 [노브랜드]노브랜드 표준형 선풍기 (FN280N)  -  34,800원
    70 [레노버]레노버 아이디어패드 Slim3 14ALC R5DOS4GB 샌드 41만  -  469,650원
    71 [삼성전자]846리터 3도어 양문형 냉장고 RS84T5041M9 공식인증점  -  1,199,000원
    72 [삼성전자]인버터 제습기 AY18BG7500GBD 1등급 혜택가 37.9만원  -  407,520원
    73 [신일전자]신일 12인치 선풍기 SIF-12MMC 5엽 가정용 스탠드형  -  49,800원
    74 [한경희생활과학]한경희생활과학 스탠드 스팀다리미 HESI-D1600WT  -  129,000원
    75 [스카이]핏 S 미니 블루투스 5.3 오픈형 무선이어폰  -  29,800원
    76 [쿠쿠](혜택가 92650원) 본사직영 BLDC 서큘레이터 선풍기 CF-AC1410WH  -  109,000원
    77 핸드폰 갤럭시S22울트라 S21 S20 노트20 노트10 노트9  -  9,900원
    78 [파세코]파세코 접이식 12인치 써큘레이터 PDF-MSFB1120W  -  119,000원
    79 [SK매직]20L 전자레인지 MWO-20M8A01 1년 무상A/S  -  63,700원
    80 [퀸메이드]초절전 BLDC 리모컨 선풍기 제로 써큘레이터 화이트  -  69,900원
    81 삼성전자 DDR4 8G PC4-25600 (정품)-PL  -  35,270원
    82 [삼성전자]삼성 윈도우핏 에어컨 길이 연장 키트 90cm  -  109,000원
    83 [쿠쿠]본사직영 CRP-HQB0310FS 3인용 IH전기압력 밥솥  -  135,000원
    84 [위닉스]DXJH193-KWK 뽀송 제습기 19L 81.5㎡/LK  -  489,020원
    85 [삼성전자]삼성 갤럭시버즈 프로 ANC 블루투스 이어폰 SM-R190  -  139,000원
    86 [신일전자][신일]  2022NEW [베이지]  BLDC 에어서큘레이터 AIR S8 (SIF-T09B  -  129,400원
    87 [신일전자]신일 선풍기 SIF-P14PCB 스탠드형 써큘레이터형  -  64,900원
    88 [뽀송]공식인증점)위닉스 17L제습기 {DN3E170-LWK} NEW/1등급  -  430,000원
    89 삼성 스탠드 선풍기 SFN-T35GFST  -  88,990원
    90 [신일전자]초미풍 스탠드 선풍기 SIF-D14BS 혜택가 59420원  -  69,900원
    91 [삼성전자]오디세이 G5 C34G55T 86.4cm 게이밍모니터 165Hz  -  599,000원
    92 [윈드피아]가정용선풍기 업소용선풍기 스탠드 선풍기 WA-370블랙  -  23,900원
    93 [삼성전자]삼성전자 WindowFit 창문형에어컨 AW05A5171EZA /MS  -  551,000원
    94 [샤오미]미밴드7 블랙 한글판 스트랩증정 국내AS  -  59,800원
    95 [삼성전자]삼성 공식인증 MicroSD EVO Plus 512GB MB-MC512KA EL  -  58,710원
    96 [위닉스]_위닉스 인버터 제습기 19L 1등급 DXJH193-KWK  -  499,000원
    97 [로이체]바람쎈 에어 써큘레이터/선풍기 RC-50  -  34,900원
    98 [신일전자]신일 BLDC 무소음 리모컨 선풍기 SIF-DC514NK  -  125,100원
    99 [신일전자]BLDC 입체회전 써큘레이터 SIF-DPNW90 혜택가  -  149,000원
    100 [쿠쿠]본사직영 20리터 전자레인지 CMW-A201DB 블랙  -  56,000원


    "\ndd = ['가']\nfor item in dd:\n    print(dd)\n"


