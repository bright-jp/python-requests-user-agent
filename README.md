# Python RequestsでUser Agentを設定・変更する方法

[![Bright Data Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/)

本ガイドでは、安全かつ成功するWebスクレイピングのために、Python RequestsでUser-Agentヘッダーを設定してローテーションする方法を解説します。

- [User Agent Headerを常に設定すべき理由](#why-you-should-always-set-the-user-agent-header)
- [RequestsのデフォルトのPython user agentとは？](#what-is-the-default-requests-python-user-agent)
- [Python RequestsのUser Agentを変更する方法](#how-to-change-the-python-requests-user-agent)
- [RequestsでUser Agentローテーションを実装する](#implement-user-agent-rotation-in-requests)

## User Agent Headerを常に設定すべき理由

User-Agent HTTPヘッダーは、リクエストを行うクライアントソフトウェアを識別します。通常、ブラウザの種類、OS、アーキテクチャなどの情報が含まれます。

以下はChromeのuser agentの例です。

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36
```

この文字列には、以下が含まれています。

* `Mozilla/5.0`: Mozilla互換性を示す歴史的なプレフィックス
* `Windows NT 10.0; Win64; x64`: OS、プラットフォーム、アーキテクチャ
* `AppleWebKit/537.36`: ブラウザエンジン
* `KHTML, like Gecko`: 互換性を示す指標
* `Chrome/125.0.0.0`: ブラウザ名とバージョン
* `Safari/537.36`: Safari互換性

user agentは、Webサイトがリクエストが正当なブラウザから来ているのか、あるいは自動化ソフトウェアの可能性があるのかを判断するのに役立ちます。アンチスクレイピングシステムは、このヘッダーを見てボットを特定してブロックすることが多く、ボットは通常デフォルトまたは一貫性のないuser agent文字列を使用します。

## RequestsのデフォルトのPython user agentとは？

Python Requestsライブラリは、以下の形式でデフォルトのUser-Agentヘッダーを設定します。

`python-requests/X.Y.Z`

ここでX.Y.ZはインストールされているRequestsのバージョンです。テストリクエストを送ることで確認できます。

```python
import requests

# make an HTTP GET request to the specified URL
response = requests.get('https://httpbin.io/user-agent')

# parse the API response as JSON and print it
print(response.json())
```

出力は次のようになります。

```
{'user-agent': 'python-requests/2.32.3'}
```

これは、ブラウザではなく自動化ツールからのリクエストであることを明確に示すため、Webサイトがあなたのスクレイピング試行を検知してブロックしやすくなります。

## Python RequestsのUser Agentを変更する方法

### カスタムUser Agentを設定する

headers辞書に渡してUser-Agentをカスタマイズします。

```python
import requests

# custom user agent header

headers = {

  'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36'

}

# make an HTTP GET request to the specified URL

# setting custom headers

response = requests.get('https://httpbin.io/user-agent', headers=headers)

# parse the API response as JSON and print it

print(response.json())
```

セッション内のすべてのリクエストに対してuser agentを設定するには、以下のようにします。

```python
import requests

# initialize an HTTP session

session = requests.Session()

# set a custom header in the session

session.headers['User-Agent'] = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36'

# perform a GET request within the HTTP session

response = session.get('https://httpbin.io/user-agent')

# print the data returned by the API

print(response.json())

# other requests with a custom user agent within the session ...
```

これにより、先ほどと同じ出力が得られます。

### User Agentを解除する

推奨はしませんが、場合によってはUser-Agentヘッダーを削除する必要があるかもしれません。`None`に設定しても期待どおりには動作しません。

```python
import requests

# custom user agent header

headers = {

  'user-agent': None

}

# make an HTTP GET request to the specified URL

# setting custom headers

response = requests.get('https://httpbin.io/user-agent', headers=headers)

# parse the API response as JSON and print it

print(response.json())
```

この方法では、urllib3のデフォルトuser agentになってしまいます。

```
{'user-agent': 'python-urllib3/2.2.1'}
```

ヘッダーを完全に削除するには、`urllib3.util.SKIP_HEADER`を使用します。

```python
import requests

import urllib3

# exclude the default user agent value

headers = { 

  'user-agent': urllib3.util.SKIP_HEADER 

}

# prepare the HTTP request to make

req = requests.Request('GET', 'https://httpbin.io/headers')

prepared_request = req.prepare()

# set the custom headers with no user agent

prepared_request.headers = headers

# create a requests session and perform

# the request

session = requests.Session()

response = session.send(prepared_request)

# print the returned data

print(response.json())
```

上記のPythonコードを実行すると、次の結果が返されます。

```
{'headers': {'Accept-Encoding': ['identity'], 'Host': ['httpbin.io']}}
```

## RequestsでUser Agentローテーションを実装する

複数のリクエストで単一のuser agentを使用すると、それでもアンチボットシステムが作動する可能性があります。user agentローテーションにより、リクエストが異なるブラウザから来ているように見せられます。

実装方法は以下のとおりです。

### Step 1: User Agentリストを作成する

```python
user_agents = [

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36",

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0",

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

    # other user agents...

]
```

### Step 2: ランダムなUser Agentを選択する

[`random.choice()`](https://docs.python.org/3/library/random.html#random.choice)を使用して、配列からuser agent文字列をランダムに抽出します。

```python
random_user_agent = random.choice(user_agents)
```

上記の行には、次のimportが必要です。

```python
import random
```

### Step 3: ランダムUser Agentを使用する

ランダムuser agentでヘッダー辞書を定義し、`requests`のリクエストで使用します。

```python
headers = {

  'user-agent': random_user_agent

}

response = requests.get('https://httpbin.io/user-agent', headers=headers)

print(response.json())
```

これらの手順には、次のimportが必要です。

```python
import requests
```

### Step 4: 完全な実装

以下がスクリプト全体のコードです。

```python
import random

import requests

# list of user agents

user_agents = [

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36",

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0",

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

    # other user agents...

]

# pick a random user agent from the list

random_user_agent = random.choice(user_agents)

# set the random user agent

headers = {

  'user-agent': random_user_agent

}

# perform a GET request to the specified URL

# and print the response data

response = requests.get('https://httpbin.io/user-agent', headers=headers)

print(response.json())
```

このスクリプトを複数回実行して、異なるuser agentが動作していることを確認してください。

## Conclusion

Python Requestsで成功するWebスクレイピングを行うには、適切なUser-Agentヘッダーの設定が不可欠です。user agentをカスタマイズしてローテーションすることで、自動化リクエストをより自然に見せ、基本的なアンチボット検知を回避できます。

ただし、高度なアンチスクレイピングシステムは、別の手段で自動化を検知する可能性があります。より堅牢なスクレイピングのためには、user agentローテーションに加えてプロキシローテーションの併用を検討するか、これらの複雑さを代わりに処理する専用の[Web Scraper API](https://brightdata.jp/products/web-scraper)の利用をご検討ください。