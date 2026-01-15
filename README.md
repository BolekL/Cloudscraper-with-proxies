# CloudScraper를 프록시와 함께 사용하기

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/) 

이 가이드는 CloudScraper의 프록시 통합을 설정하는 방법, IP를 ローテーティングする 방법, 그리고 원활한 스크레이핑을 위해 인증된 프록시를 사용하는 방법을 다룹니다.

- [CloudScraper란 무엇입니까?](#about-cloudscraper)
- [CloudScraper에서 왜 프록시를 사용합니까?](#why-use-proxies-with-cloudscraper)
- [CloudScraper로 프록시 설정하기](#setting-up-a-proxy-with-cloudscraper)
- [프록시 ローテーション 구현하기](#implementing-proxy-rotation)
- [CloudScraper에서 인증된 프록시 사용하기](#using-authenticated-proxies-in-cloudscraper)
- [CloudScraper에 프리미엄 프록시 통합하기](#integrating-premium-proxies-in-cloudscraper)
- [결론](#conclusion)

## About CloudScraper

[CloudScraper](https://github.com/VeNoMouS/cloudscraper)는 Cloudflare의 안티봇 페이지(일반적으로 "I'm Under Attack Mode" 또는 IUAM으로 알려짐)를 우회하도록 설계된 Python 모듈입니다. 내부적으로는 가장 인기 있는 Python HTTP 클라이언트 중 하나인 Requests를 사용하여 구현되어 있습니다.

## Why Use Proxies with CloudScraper?

Cloudflare는 요청를 너무 많이 보내거나 우회하기 어려운 보다 정교한 방어를 트리거하면 사용자의 IP 주소를 차단할 수 있습니다. Cloudflare에서 호스팅되는 웹사이트를 스크레이핑할 때 프록시와 CloudScraper를 함께 사용하면 다음 두 가지 핵심 이점이 있습니다:

- **보안 및 익명성 향상**: 요청를 프록시를 통해 라우팅하면 실제 신원이 숨겨져 탐지 위험이 줄어듭니다.
- **차단 및 중단 방지**: 프록시를 사용하면 IP 주소를 동적으로 ローテーティング할 수 있어 차단 및 속도 제한を 우회하는 데 도움이 됩니다.

## Setting Up a Proxy With CloudScraper

### Step #1: Install CloudScraper

`cloudscraper` pip 패키지를 설치합니다:

```bash
pip install -U cloudscraper
```

`-U` 옵션은 Cloudflare의 안티봇 엔진에 대한 최신 우회책이 포함된 패키지의 최신 버전을 받도록 보장합니다.

### Step #2: Initialize CloudScraper

CloudScraper를 import합니다:

```python
import cloudscraper
```

`create_scraper()` 메서드를 사용하여 CloudScraper 인스턴스를 생성합니다:

```python
scraper = cloudscraper.create_scraper()
```

`scraper` 객체는 `requests` 라이브러리의 `Session` 객체와 유사하게 동작합니다. 특히 Cloudflare의 안티봇 조치를 우회하면서 HTTP 요청를 보낼 수 있게 해줍니다.

### Step #3: Integrate a Proxy

아래와 같이 `proxies` 딕셔너리를 정의하고 `get()` 메서드에 전달합니다:

```python
proxies = {
    "http": "<YOUR_HTTP_PROXY_URL>",
    "https": "<YOUR_HTTPS_PROXY_URL>"
}

# Perform a request through the specified proxy
response = scraper.get("<YOUR_TARGET_URL>", proxies=proxies)
```

`get()` 메서드의 `proxies` 매개변수는 Requests로 전달됩니다. 이를 통해 HTTP 클라이언트는 대상 URL의 프로토콜에 따라 지정된 HTTP 또는 HTTPS 프록시 서버를 통해 요청를 라우팅할 수 있습니다.

### Step #4: Test the CloudScraper Proxy Integration Setup

데모를 위해 HTTPBin 프로젝트의 `/ip` 엔드포인트를 대상으로 하겠습니다. 이 엔드포인트는 호출자의 IP 주소를 반환합니다. 모든 것이 예상대로 동작한다면, 응답에는 프록시 서버의 IP 주소가 표시되어야 합니다.

프로キ시 서버의 URL이 `http://202.159.35.121:443`라고 가정하면, 스크립트 코드는 다음과 같습니다:

```python
import cloudscraper

# Create a CloudScraper instance
scraper = cloudscraper.create_scraper()

# Specify your proxy
proxies = {
    "http": "http://202.159.35.121:443",
    "https": "http://202.159.35.121:443"
}

# Make a request through the proxy
response = scraper.get("https://httpbin.org/ip", proxies=proxies)

# Print the response from the "/ip" endpoint
print(response.text)
```

다음과 같은 응답가 표시되어야 합니다:

```json
{
    "origin": "202.159.35.121"
}
```

응답의 IP는 예상대로 프록시 서버의 IP와 일치합니다.

> **Note**:\
> 무료 프록시 서버는 수명이 짧은 경우가 많습니다. 스크립트를 테스트할 때는 새 IP 주소의 프록시를 확보하는 것이 가장 좋습니다.

## Implementing Proxy Rotation

신뢰할 수 있는 제공업체에서 プロキ시 목록을 가져와 배열에 저장합니다:

```python
proxy_list = [
    {"http": "<YOUR_PROXY_URL_1>", "https": "<YOUR_PROXY_URL_1>"},
    # ...
    {"http": "<YOUR_PROXY_URL_n>", "https": "<YOUR_PROXY_URL_n>"},
]
```

다음으로 `random.choice()` 메서드를 사용하여 목록에서 プロキ시를 무작위로 선택합니다:

```python
import random

random_proxy = random.choice(proxy_list)
```

무작위로 선택된 プロキ시를 `get()` 요청에 설정합니다:

```python
response = scraper.get("<YOUR_TARGET_URL>", proxies=random_proxy)
```

모든 설정이 올바르면 실행할 때마다 リクエ스트는 목록의 다른 プロキ시를 사용하게 됩니다. 전체 코드는 다음과 같습니다:

```python
import cloudscraper
import random

# Create a Cloudscraper instance
scraper = cloudscraper.create_scraper()

# List of proxy URLs (replace with actual proxy URLs)
proxy_list = [
    {"http": "<YOUR_PROXY_URL_1>", "https": "<YOUR_PROXY_URL_1>"},
    # ...
    {"http": "<YOUR_PROXY_URL_n>", "https": "<YOUR_PROXY_URL_n>"},
]

# Randomly select a proxy from the list
random_proxy = random.choice(proxy_list)

# Make a request using the randomly selected proxy
# (replace with the actual target URL)
response = scraper.get("<YOUR_TARGET_URL>", proxies=random_proxy)
```

## Using Authenticated Proxies in CloudScraper

CloudScraper에서 プロキ시를 인증하려면, 필요한 자격 증명을 プロ키시 URL에 직접 포함합니다. 사용자명 및 비밀번호 인증 형식은 다음과 같습니다:

`<PROXY_PROTOCOL>://<YOUR_USERNAME>:<YOUR_PASSWORD>@<PROXY_IP_ADDRESS>:<PROXY_PORT>`
    
이 형식을 사용하면 CloudScraper プロキ시 구성은 다음과 같습니다:

```python
import cloudscraper

# Create a Cloudscraper instance
scraper = cloudscraper.create_scraper()  

# Define your authenticated proxy
proxies = {
   "http": "<PROXY_PROTOCOL>://<YOUR_USERNAME>:<YOUR_PASSWORD>@<PROXY_IP_ADDRESS>:<PROXY_PORT>",
   "https": "<PROXY_PROTOCOL>://<YOUR_USERNAME>:<YOUR_PASSWORD>@<PROXY_IP_ADDRESS>:<PROXY_PORT>"
}

# Perform a request through the specified authenticated proxy
response = scraper.get("<YOUR_TARGET_URL>", proxies=proxies)
```

## Integrating Premium Proxies in CloudScraper

프로덕션 스크레이핑 환경에서 신뢰할 수 있는 결과를 얻으려면 [Bright Data](https://brightdata.co.kr/)와 같은 최상위 제공업체의 プロ키시를 사용하십시오. CloudScraper에 Bright Data의 プロ키시를 통합하려면 다음을 수행합니다:

1. 계정을 생성하거나 로그인합니다.

2. 대시보드로 이동한 다음 표에서 “Residential” 존을 클릭합니다:

![Bright Data's proxies and scraping infrastructure control panel](https://github.com/bright-kr/Cloudscraper-with-proxies/blob/main/image-7.png)

3. 토글을 클릭하여 プロ키시를 활성화합니다:

![Turning on the residential zone](https://github.com/bright-kr/Cloudscraper-with-proxies/blob/main/image-8.png)

이제 다음과 같이 표시되어야 합니다:

![The residential zone turned on](https://github.com/bright-kr/Cloudscraper-with-proxies/blob/main/image-9.png)
> **Note**:\
> Bright Data의 レジデンシャル프록시는 자동으로 ローテーティング됩니다.

4. “Access Details” 섹션에서 プロ키시 호스트, 사용자명, 비밀번호를 복사합니다:

![The access details for your residential proxies zone](https://github.com/bright-kr/Cloudscraper-with-proxies/blob/main/image-10.png)
Bright Data プロ키시 URL은 다음과 같이 보입니다:

```
http://<PROXY_USERNAME>:<PROXY_PASSWORD>@brd.superproxy.io:33335
```

5. 아래와 같이 プロ키시를 Cloudscraper에 통합합니다:

```python
import cloudscraper
# Create a CloudScraper instance
scraper = cloudscraper.create_scraper()
# Define the premium proxy
proxies = {
"http": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST>:<PROXY_PORT>",
"https": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST>:<PROXY_PORT>"
}
# Perform a request using the premium proxy
response = scraper.get("https://httpbin.org/ip", proxies=proxies)
# Print the response to verify the proxy is working
print(response.text)
```

CloudScraper プロ키시 통합이 완료되었습니다. 이제 테스트 및 검증이 필요합니다. プロ키시가 올바르게 작동하는지 확인하려면 호출자의 IP 주소를 반환하는 [https://httpbin.org/ip](https://httpbin.org/ip) 같은 서비스로 테스트할 수 있습니다. 설정이 올바르면 응답에는 로컬 IP가 아니라 プロ키시 서버의 IP 주소가 표시되어야 합니다.


## Putting Everything Together 

```python
import cloudscraper
import random
import time

# Step 1: Define a list of proxies (authenticated and non-authenticated)
# Replace <PROXY_USERNAME>, <PROXY_PASSWORD>, <PROXY_HOST>, and <PROXY_PORT> with actual values
proxy_list = [
    {"http": "http://<PROXY_HOST_1>:<PROXY_PORT_1>", "https": "http://<PROXY_HOST_1>:<PROXY_PORT_1>"},
    {"http": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_2>:<PROXY_PORT_2>", 
     "https": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_2>:<PROXY_PORT_2>"},
    {"http": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_3>:<PROXY_PORT_3>", 
     "https": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_3>:<PROXY_PORT_3>"}
]

# Step 2: Create a CloudScraper instance
scraper = cloudscraper.create_scraper()

# Step 3: Define the target URL
target_url = "https://httpbin.org/ip"  # This endpoint returns the caller's IP address

# Step 4: Implement proxy rotation and make requests
def fetch_with_proxy_rotation(proxy_list, target_url, num_requests=5):
    """
    Fetch the target URL using proxy rotation.
    
    Args:
        proxy_list (list): A list of proxy configurations.
        target_url (str): The URL to scrape.
        num_requests (int): Number of requests to make.
    """
    for i in range(num_requests):
        # Randomly select a proxy from the list
        proxy = random.choice(proxy_list)
        
        try:
            # Make a request using the selected proxy
            print(f"Using proxy: {proxy}")
            response = scraper.get(target_url, proxies=proxy, timeout=10)
            
            # Print the response (IP address of the proxy)
            print(f"Response {i + 1}: {response.text}")
        
        except Exception as e:
            # Handle errors (e.g., connection timeout, proxy failure)
            print(f"Error with proxy {proxy}: {e}")
        
        # Wait a bit before the next request to mimic human behavior
        time.sleep(random.uniform(1, 3))

# Step 5: Run the function
fetch_with_proxy_rotation(proxy_list, target_url, num_requests=5)
```

### Output Example

```python
Using proxy: {'http': 'http://<PROXY_HOST_1>:<PROXY_PORT_1>', 'https': 'http://<PROXY_HOST_1>:<PROXY_PORT_1>'}
Response 1: {
    "origin": "203.0.113.1"
}
Using proxy: {'http': 'http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_2>:<PROXY_PORT_2>', 'https': 'http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_2>:<PROXY_PORT_2>'}
Response 2: {
    "origin": "198.51.100.2"
}
Using proxy: {'http': 'http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_3>:<PROXY_PORT_3>', 'https': 'http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_3>:<PROXY_PORT_3>'}
Response 3: {
    "origin": "192.0.2.3"
}
...
```

## Conclusion

Bright Data는 Fortune 500 기업과 20,000명 이상의 고객에게 서비스를 제공하며, 세계 최고의 プロ키시 서버를 운영합니다. 전 세계 プロ키시 네트워크에는 다음이 포함됩니다:

*   [Datacenter proxies](https://brightdata.co.kr/proxy-types/datacenter-proxies) – 770,000개 이상의 データセンター프록시 IP.
*   [Residential proxies](https://brightdata.co.kr/proxy-types/residential-proxies) – 195개 이상의 국가에서 72M 이상의 レジデンシャル프록시 IP.
*   [ISP proxies](https://brightdata.co.kr/proxy-types/isp-proxies) – 700,000개 이상의 ISP프록시 IP.
*   [Mobile proxies](https://brightdata.co.kr/proxy-types/mobile-proxies) – 7M 이상의 モバイル프록시 IP.

지금 [무료 Bright Data 계정을 생성](https://brightdata.co.kr)하여 プロ키시 서버를 사용해 보십시오.