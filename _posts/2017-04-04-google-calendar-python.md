---
layout:	post
title:	Python GUI APIs 連結到 Google Service 
date:	2017-04-04
categories: python
---

這篇文章說明如何使用appJar - Python GUI APIs撰寫一個視窗程式, 連結到Google Calendar, 呈現使用者行事曆中的events, 並可以新增event.

首先一些參考資料:
- [Google Calendar API文件](https://developers.google.com/resources/api-libraries/documentation/calendar/v3/python/latest/)
- [Google oauth2client 函式庫](http://oauth2client.readthedocs.io/en/latest/index.html)

### OAuth
在說明程式細節前, 先整理一下關於呼叫Google APIs所需使用到的OAuth 2.0協定. 

OAuth是一種開放式的授權(Authorization)標準，讓使用者可以授權第三方應用, 存取他在某一網站上儲存的私密的資源, 如相片，影片，聯絡人等資訊，而無需使用者將他登入到該網站的名稱和密碼, 透露給第三方應用。 這相當於若是要允許外人進入到家裡, 最不安全的方式是把大門鑰匙交給別人, 比較安全的方式是自己用鑰匙打開門讓人進去. 

基本作法: 這個第三方應用(Consumer, 消費者), 必須先取得資源網站(Service Provider, 服務提供者)所提供的一組tokens(`Consumer Key/Secret`), 如此一來, 這個第三方應用才可以建立與資源網站的連結. 使用者使用這個第三方應用之前, 會先連線到資源網站的畫面去做登入和授權的步驟(使用者登入的密碼是直接在資源網站內輸入, 因此不會洩漏給第三方應用). 授權之後, 這個第三方應用會從資源網站處取得另一個token(`Access Token`). 爾後必須使用這個token, 才能夠去資源網站存取使用者的私密資源.

Google APIs根據第三方應用的程式形式不同, 區分為[三種模式](https://developers.google.com/identity/protocols/OAuth2):
1. OAuth 2.0 for Web Server Applications: 
1. OAuth 2.0 for Installed Applications
1. OAuth 2.0 for Server to Server

這裡測試的Python程式, 採用`OAuth 2.0 for Installed Application`來做驗證, 因此這裡所描述的步驟, 是依據這種形式來作說明.
這種形式有一個假設前提 - 我們所執行的Pyhton程式所在的環境, 允許其開啟一個瀏覽器視窗(為了能夠導引使用者去到資源網站去做授權). 如果不滿足這個假設, 則不能使用這裡描述的作法. 但是Google還有提供一種`OAuth 2.0 for Devices`的形式, 可以針對沒有瀏覽器可以使用的狀況下的應用.

### 建立Credentials

為了讓第三方應用可以使用Google APIs來連結Google各項的雲端服務, 必須先取得一組`Consumer Key/Secret`的token, 在Google上他們稱之為`Client ID`. 因此必須先到[Google API Console](https://console.developers.google.com/apis/credentials)上建立Credentials(Client ID). 選擇`Installed application`做為第三方應用的型態, 然後下載`client_secrets.json`檔案到local目錄, 後面的Python程式會使用它.

### 選擇Redirect URI
檢視`client_secrets.json`檔案內容, 可以看到`Redirect URI`有兩個數值: `urn:ietf:wg:oauth:2.0:oob` and `http://localhost`. 這項設定可以控制Python程式如何從Google Authorization Server得到授權碼(`Access Token`). **第一種方式使用`http://localhost`的設定, 代表Access Token將以redirect到client端的web server的query string parameter的方式, 讓client取得Access Token.** 因此這個方式的前提是python程式可以取得local web server存取的資訊. **另一種方式採用 `urn:ietf:wg:oauth:2.0:oob`, 則會在browser的title bar上顯示Access Token, 但是需要使用者複製這個Access Token然後輸入到python程式中.** 

這裡的步驟採用第一種方式, 亦即不需要使用者額外做cut-and-paste的動作. Google 網站上提供的一個現成的python範例程式

### 安裝Python語言的Google Client Library

```
pip install --upgrade google-api-python-client
```

### python[範例程式](https://developers.google.com/google-apps/calendar/quickstart/python): 使用redirect_url=`http://localhost`
```
from __future__ import print_function
import httplib2
import os

from apiclient import discovery
from oauth2client import client
from oauth2client import tools
from oauth2client.file import Storage

import datetime

try:
    import argparse
    flags = argparse.ArgumentParser(parents=[tools.argparser]).parse_args()
except ImportError:
    flags = None

# If modifying these scopes, delete your previously saved credentials
# at ~/.credentials/calendar-python-quickstart.json
SCOPES = 'https://www.googleapis.com/auth/calendar.readonly' #1
CLIENT_SECRET_FILE = 'client_secrets.json'
APPLICATION_NAME = 'Google Calendar API Python Quickstart'


def get_credentials():
    """Gets valid user credentials from storage.

    If nothing has been stored, or if the stored credentials are invalid,
    the OAuth2 flow is completed to obtain the new credentials.

    Returns:
        Credentials, the obtained credential.
    """
    home_dir = os.path.expanduser('~')
    credential_dir = os.path.join(home_dir, '.credentials')
    if not os.path.exists(credential_dir):
        os.makedirs(credential_dir)
    credential_path = os.path.join(credential_dir,
                                   'calendar-python-quickstart.json')

    store = Storage(credential_path) #2
    credentials = store.get()
    if not credentials or credentials.invalid:
        flow = client.flow_from_clientsecrets(CLIENT_SECRET_FILE, SCOPES) #3
        flow.user_agent = APPLICATION_NAME
        if flags:
            credentials = tools.run_flow(flow, store, flags) #4
        else: # Needed only for compatibility with Python 2.6
            credentials = tools.run(flow, store)
        print('Storing credentials to ' + credential_path)
    return credentials

def main():
    """Shows basic usage of the Google Calendar API.

    Creates a Google Calendar API service object and outputs a list of the next
    10 events on the user's calendar.
    """
    credentials = get_credentials()
    http = credentials.authorize(httplib2.Http()) #5
    service = discovery.build('calendar', 'v3', http=http) #6

    now = datetime.datetime.utcnow().isoformat() + 'Z' # 'Z' indicates UTC time
    print('Getting the upcoming 10 events')
    eventsResult = service.events().list( 
        calendarId='primary', timeMin=now, maxResults=10, singleEvents=True,
        orderBy='startTime').execute() #7
    events = eventsResult.get('items', [])

    if not events:
        print('No upcoming events found.')
    for event in events:
        start = event['start'].get('dateTime', event['start'].get('date'))
        print(start, event['summary'])


if __name__ == '__main__':
    main()
```
說明:
1. 設定[OAuth 2.0 Scope for Google APIs](https://developers.google.com/identity/protocols/googlescopes).這個Scope控制第三方應用具有什麼樣的權限可以使用使用者的資源. Calendar APIs, v3的Scope有兩種: `https://www.googleapis.com/auth/calendar`(可以管理calendars), 及`https://www.googleapis.com/auth/calendar.readonly`(僅可以讀取calendars)
1. 使用oauth2client.file.Storage物件, 將Access Token儲存到檔案中, 以便後續的使用
1. `oauth2client.client.flow_from_clientsecrets()`: 根據`client_secrets.json`檔案內容, 來建立Flow物件(驗證流程)
1. `tools.run_flow()`: 這是取得Access Token的magic所在. 這個函數會根據Flow物件, 來執行驗證步驟. 例如(1)開啟使用者預設的browser, 連結至Google authorization server. (2)要求使用者授權讓python程式存取使用者的資源. 若授權成功, 會回傳Access Token.(3)將Access Token儲存到storage參數中. 
1. `credentials.authorize(httplib2.Http())`: 使用Access Token物件, 授權一個Http物件.
1. `discovery.build('calendar', 'v3', http=http)`: 使用這個被授權的Http物件, 建立一個使用某一項Google雲端服務的Service物件
1. `service.events().list().execute()`: 呼叫Google Calendar APIs來取得事件的列表

補充: 以下是使用redirect_uri=`urn:ietf:wg:oauth:2.0:oob`的[範例](https://developers.google.com/api-client-library/python/auth/installed-app). 第三方應用須仰賴使用者將Access Token複製, 然後輸入到第三方程式中
```python
import json
import webbrowser

import httplib2

from apiclient import discovery
from oauth2client import client


if __name__ == '__main__':
  flow = client.flow_from_clientsecrets(
    'client_secrets.json',
    scope='https://www.googleapis.com/auth/drive.metadata.readonly',
    redirect_uri='urn:ietf:wg:oauth:2.0:oob')

  auth_uri = flow.step1_get_authorize_url()
  webbrowser.open(auth_uri)

  auth_code = raw_input('Enter the auth code: ')

  credentials = flow.step2_exchange(auth_code)
  http_auth = credentials.authorize(httplib2.Http())

  drive_service = discovery.build('drive', 'v2', http_auth)
  files = drive_service.files().list().execute()
  for f in files['items']:
    print f['title']
```

### appJar範例
```
from appJar import gui
import httplib2
import os, sys, traceback

from apiclient import discovery
from oauth2client import client
from oauth2client import tools
from oauth2client.file import Storage

import datetime

app = None 
def main():
	global app
	app = gui()
	app.addLabel("l0","Welcome to MyCalendar app",0,0,3)
	app.addNamedButton("連結Google Calendar","connect", connect,1,0,3)
	app.startLabelFrame("近期事件") #1
	row = app.getRow()
	app.addLabel("l1","無任何事件", row, 0, 3)
	app.stopLabelFrame() #2
	#
	app.startLabelFrame("新增事件")
	row = app.getRow()
	app.addLabelEntry("Summary", row, 0, 3)
	row = app.getRow()
	app.addLabelEntry("Description", row, 0, 3)
	row = app.getRow()
	app.addLabelEntry("Location", row, 0, 3)
	row = app.getRow()
	app.addLabelEntry("Start Date(yyyy-mm-dd)", row, 0, 3)
	row = app.getRow()
	app.addLabelEntry("Start Time(hh:mm:ss)", row, 0, 3)
	row = app.getRow()
	app.addLabelEntry("End Time(hh:mm:ss)", row, 0, 3)
	row = app.getRow()
	app.addNamedButton("新增到Google Calendar","insert", insert,row,0,3)
	app.stopLabelFrame()

	app.go()

def connect(btn): 
	service = get_calendar_service()
	app.openLabelFrame("近期事件") #3
	if app.getLabel("l1"): app.removeLabel("l1")
	if service is not None:
		events = get_events(service)
		#display in a Label frame
		if events:
			row = app.getRow()
			app.addLabel("l%d%d" % (row,0),"開始時間", row, 0)
			app.addLabel("l%d%d" % (row,1),"主題", row, 1)
			app.addLabel("l%d%d" % (row,2),"地點", row, 2)
			for index, event in enumerate(events):
				start = event['start'].get('dateTime', event['start'].get('date'))
				start = datetime.datetime.strptime(start[:19], "%Y-%m-%dT%H:%M:%S").strftime("%y-%m-%d(%a) %H:%M")
				summary = event.get('summary',"無")
				location = event.get('location',"無")
				row = app.getRow()
				app.addLabel("l%d%d" % (row,0), start, row, 0)
				app.addLabel("l%d%d" % (row,1), summary, row, 1)
				app.addLabel("l%d%d" % (row,2), location, row, 2)
		else:
			row = app.getRow()
			app.addLabel("l4","無任何事件", row, 0, 3)
	else:
		row = app.getRow()
		app.addLabel("l5","驗證錯誤", row, 0, 3)
	app.stopLabelFrame()
	
def insert(btn): 
	service = get_calendar_service()
	event = {
		'summary': app.getEntry("Summary"),
		'location': app.getEntry("Location"),
		'description': app.getEntry("Description"),
		'start': {
			'dateTime': '%sT%s+08:00' % (app.getEntry("Start Date(yyyy-mm-dd)"),app.getEntry("Start Time(hh:mm:ss)")),
			'timeZone': 'Asia/Taipei',
		}, 
		'end': {
			'dateTime': '%sT%s+08:00' % (app.getEntry("Start Date(yyyy-mm-dd)"),app.getEntry("End Time(hh:mm:ss)")),
			'timeZone': 'Asia/Taipei',
		}, 
	}
	insert_event(service, event)

def get_calendar_service():
	try:
		import argparse
		flags = argparse.ArgumentParser(parents=[tools.argparser]).parse_args()
	except ImportError:
		flags = None

	# If modifying these scopes, delete your previously saved credentials
	# at ~/.credentials/calendar-python-quickstart.json
	SCOPES = 'https://www.googleapis.com/auth/calendar'
	CLIENT_SECRET_FILE = 'client_secrets.json'
	APPLICATION_NAME = 'MyCalendar'

	"""Gets valid user credentials from storage.

	If nothing has been stored, or if the stored credentials are invalid,
	the OAuth2 flow is completed to obtain the new credentials.

	Returns:
		Credentials, the obtained credential.
	"""
	home_dir = os.path.expanduser('~')
	credential_dir = os.path.join(home_dir, '.credentials')
	if not os.path.exists(credential_dir):
		os.makedirs(credential_dir)
	credential_path = os.path.join(credential_dir,
								   'MyCalendar.json')

	store = Storage(credential_path)
	credentials = store.get()
	if not credentials or credentials.invalid:
		flow = client.flow_from_clientsecrets(CLIENT_SECRET_FILE, SCOPES)
		flow.user_agent = APPLICATION_NAME
		if flags:
			credentials = tools.run_flow(flow, store, flags)
		else: # Needed only for compatibility with Python 2.6
			credentials = tools.run(flow, store)
		print('Storing credentials to ' + credential_path)
	#
	http = credentials.authorize(httplib2.Http())
	service = discovery.build('calendar', 'v3', http=http)
	return service

def get_events(service, num=15):
	now = datetime.datetime.utcnow().isoformat() + 'Z' # 'Z' indicates UTC time
	#print('Getting the upcoming 10 events')
	
	eventsResult = service.events().list(
		calendarId='primary', timeMin=now, maxResults=num, singleEvents=True,
		orderBy='startTime').execute()
	events = eventsResult.get('items', [])

	if not events:
		#print('No upcoming events found.')
		app.infoBox("ib1","目前行事曆中沒有新的事件!")
	return events

def insert_event(service, event):
	try:
		event = service.events().insert(calendarId='primary', body=event).execute()
		print('Event created: %s' % (event.get('htmlLink')))
		if event:
			app.infoBox("ib2","新增事件成功!")
		else:
			app.errorBox("ib2","新增事件失敗!")
	except:
		traceback.print_exc(file=sys.stdout)
		app.errorBox("ib2","新增事件失敗!")
if __name__ == '__main__':
	main()
```
:sweat_smile:
