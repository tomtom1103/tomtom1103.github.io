---
layout: post
title: Sentiment analysis with Reddit API on /r/stocks - Pt.1
tags:
categories: pythonic
thumbnail: /assets/img/posts/pythonic/redditstockanalysis1/1.png
---

## Contents

- What is Reddit?
- The Reddit API using praw
- Ticker Parsing

## What is Reddit?

레딧은 미국의 social news aggregation 플랫폼이다. 직접 사용해본 사람이 아니면 레딧의 설명만 들으면 고개를 갸우뚱 할텐데, 한국과 비교하자면 디씨인사이드와 비슷하게 주제별 커뮤니티가 나뉜다 (dcinside : gallery = reddit : subreddit). ~~하지만 차원이 다르다~~

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/pythonic/redditstockanalysis1/1.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

현재 레딧은 전세계에서 19번째로 많이 방문되는 웹사이트고, 매일 5억명이 접속하여 정보를 공유하는 플랫폼이다. 인간의 뇌로 생각할 수 있는 주제가 있으면 그에 대한 Subreddit 가 존재한다는 말이 있다.

> 농담이 아니다. 맥주잔을 들고 있는 13세기 수도승들의 그림만 모으는 [subreddit](https://www.reddit.com/r/monkslookingatbeer/) 도 존재한다.

그렇기 때문에 레딧은 **Data Goldmine** 이라고도 할 수 있다. 매일같이 그 어떤 주제를 막론하고 포스팅이 올라오기 때문에, 특정 주제에 대해 사람들의 생각이나 감정을 분석해보고 싶으면 해당 subreddit 을 접속하면 된다.

[/r/stocks](https://reddit.com/r/stocks) 는 미국 주식을 다루는 subreddit 이다. 약 3.6억명의 멤버가 있고, 평균 동시접속자는 1만명으로 굉장히 활성화 되어있는 커뮤니티다. 매일같이 미국 증시의 시황분석과 의견에 대한 포스팅이 올라오기 때문에, 감성분석을 위한 데이터베이스로 최적화되어있다.

레딧 스크레이핑은 전세계 개발자들의 공통 관심사 중 하나이기 때문에, 다양한 방법이 존재한다.

1. Manual Scraping
2. Third-Party APIs
3. Reddit API

먼저 manual scraping 은 논외다. 레딧의 [robots.txt](https://www.reddit.com/robots.txt) 를 확인해 보면 /post 가 Disallow 이기 때문에 Selenium 같은 browser automator 를 사용하는 방법은 윤리적이지 않다.

```html
Disallow: /post Disallow: /submit Disallow: /goto Disallow: /*after= Disallow: /*before= Disallow: /domain/*t= Disallow: /login Disallow:
/remove_email/t2_* Disallow: /r/*/user/
```

> robots.txt 중 일부.

Octoparse 같은 Third-Party API 를 사용해도 되지만, 무료가 아니다. 그렇기 때문에 Reddit 에서 공식으로 지원하는 Reddit API 가 가장 좋다. 이 Reddit API 를 발급받아서 /r/stocks 의 감성분석을 진행해 보았다.

## The Reddit API using praw

레딧의 [공식 API 문서](https://github.com/reddit-archive/reddit/wiki/oauth2) 를 참고하면 쉽게 API 를 발급 받을 수 있다. 보통 다른 사용자들은 이 API 를 개인 subreddit 을 관리하는 용도로 사용하기 때문에 request 에 cap 이 정해져 있지만, 단순 scraping 만으로 request 가 막히는 상황은 지금까지 없었다.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/pythonic/redditstockanalysis1/2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

> API 의 access 에 대해 조사하던 중, 이런 포스팅을 발견했다. 레딧 측에서 API 를 무료로 제공하고 사용자들의 cap 을 딱히 모니터하지 않는 것 같다.

API 를 발급 받고 python으로 레딧 API 를 사용 가능하게 하는 라이브러리인 [praw](https://praw.readthedocs.io/en/latest/index.html) 를 pip 으로 설치하면 된다.

```python
reddit = praw.Reddit(
    client_id=client_id,
    client_secret=client_secret,
    user_agent=user_agent,
    username=username,
    password=password
)
```

그다음 위와 같이 praw.Reddit 이란 인스턴스를 정의해줘야 한다. 해당 인자들의 상세설명은 [공식 API 문서](https://github.com/reddit-archive/reddit/wiki/oauth2) 를 참고하면 되고, 개인 API key 이기 때문에 config.py 에 정의 후 import 를 해주면 된다.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/pythonic/redditstockanalysis1/3.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

> 이때 .gitignore 에 config.py 를 깜빡하고 추가 안하고 커밋해버리면 gitguardian 한테 심장떨어지는 메일을 받을 수 도 있으니 조심하자.

```python
def parse_submission(submission):
    return {
        'title': submission.title,
        'created_utc': submission.created_utc,
        'selftext': submission.selftext,
        'score': submission.score,
        'id': submission.id
    }


def main():
    subreddit = reddit.subreddit('Stocks')
    brr = pd.DataFrame()
    print('Parsing Reddit..')
    for submission in tqdm(subreddit.new(limit=None)):
        brr = brr.append(parse_submission(submission), ignore_index=True)
    brr.to_pickle(f'{reportpath}/{today}-goes-brr.pkl')
```

praw.Reddit 인스턴스를 설정해주었으면 /r/stocks 의 글을 긁어오는 두개의 함수를 설정해 주었다. parse_submission 함수는 하나의 포스팅의 metadata를 가져오는 함수다. 하나의 포스팅에는 굉장히 많은 metadata를 담고 있는데, reddit 의 개발자들이 주단위로 바꾸기 때문에 [docs](https://praw.readthedocs.io/en/latest/code_overview/models/submission.html?highlight=created_utc#praw.models.Submission) 를 참고해서 무엇이 있는지 확인해주면 좋다. 하지만 감성분석을 위해 primary key 에 해당되는 title, created_utc, selftext, score, id 만으로도 충분하다.

main 함수는 /r/stocks 에 있는 포스팅을 데이터프레임으로 저장해주는 함수다. parse_submission 에 지정한 인자들이 데이터프레임의 column 들이 되고, 설정한 path 에 .pkl 로 저장해준다.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/pythonic/redditstockanalysis1/4.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

저장된 .pkl 을 불러와보면 이런 데이터프레임을 확인할 수 있다. 사용자들이 기업의 이름을 그대로 쓰지 않고 기업의 티커를 쓴다는 것은 /r/stocks 의 특징 중 하나다.

> What's the future for AMZN?\n\nI am very much overweighted AMZN (schoolboy error) and invested in at around 2700.\n\nIt's traded sideways for 18 months but I was happy as it was typically above 3000, and I felt I'd managed to buy in at a price that would sustain.\n\nNow it's down 25% over a year, and 12% ish over 5 days. So I'm now just $150 from my break even point. That's 5%... Or just one more Friday day of trading!\n\nOh, how I wish I was smart and exited in December.\n\nObviously, we don't have crystal balls. And a lot of bears about. But I'd be interested to hear your thoughts on the future of AMZN... The immediate future, likely recovery, and longer term.\n\nThank you!\n\n(And please don't hate on this newbie too much. I've been in/out of stocks a few times and always dreamed of getting in with a decent buffer. COVID allowed me to do that. But it's about to kick me out again.... Or at least take me to places I may not be able to afford to dip to!)

위 포스트를 보면 아마존이란 기업을 Amazon 이라 쓰지 않고 티커인 AMZN 이라고 부른다. 그렇기 때문에 미국 증시에 상장되어 있는 티커 리스트를 확보한다면 해당 포스트가 어떤 기업에 대해 이야기 하고 있는지 쉽게 연관지을 수 있다.

## Ticker Parsing

미국 증시는 크게 3개의 index 로 분류 할 수 있다.

1. S&P 500 - 미국의 시가총액 500위 기업들을 모아둔 index.
2. NASDAQ - 뉴욕의 거래소이자 NASDAQ 거래소를 추종하는 index. 대부분 Tech, Financial, Consumer 기업들로 이루어져 있다.
3. DOW - 뉴욕증권거래소(NYSE)와 NASDAQ 의 거래소의 거래량 30위 기업들을 모아둔 index. 정확히는 DJIA (Dow Jones Industrial Average) 이며, Wall Street Journal 을 퍼블리싱 하는 Dow Jones and Company 기업과는 다른 의미다.

> 미국 증시와 S&P, NASDAQ, DOW 는 위 설명보다 훨씬 복잡하다. 더욱 자세한 의미와 계산방식은 [해당 Investopedia](https://www.investopedia.com/ask/answers/difference-between-dow-and-nasdaq/) 를 참고하자.

각 index 의 공식 웹에서 티커 리스트를 다운받는 방법이 있지만, pythonic 하게 (~~라이브러리 꼼수를 써서~~) 쉽게 가져올 수 있다.

```bash
!pip install yahoo_fin
```

```python
from yahoo_fin import stock_info as si

sp500 = pd.DataFrame(si.tickers_sp500())
nasdaq = pd.DataFrame(si.tickers_nasdaq())
dow = pd.DataFrame(si.tickers_dow())
sym1 = set(symbol for symbol in sp500[0].values.tolist())
sym2 = set(symbol for symbol in nasdaq[0].values.tolist())
sym3 = set(symbol for symbol in dow[0].values.tolist())

```

야후에서 지원하는 yahoo_fin 라이브러리에서 stock_info 라는 함수를 불러오면 쉽게 원하는 인덱스의 티커를 불러 올 수 있다. 해당 티커들을 셋으로 묶은 뒤 parsing 하였다.

```python
def tickers(*args):
    symbols = set.union(*args)

    my_list = ['W', 'R', 'P', 'Q']  # warrants, rights, first preferred issue, bankruptcy
    del_set = set()
    sav_set = set()

    for symbol in tqdm(symbols):
        if len(symbol) > 4 and symbol[-1] in my_list:
            del_set.add(symbol)
        else:
            sav_set.add(symbol)

    sav_set = list(sav_set)
    tickers = pd.DataFrame(sav_set)
    tickers['tickers'] = tickers[0]
    del tickers[0]

    if (tickers['tickers'].str.len() < 1).sum():
        tickers['tickers'].replace('', np.nan, inplace=True)
        tickers.dropna(subset=['tickers'], inplace=True)
    else:
        pass

    tickers['length'] = tickers.tickers.str.len()
    tickers.to_pickle('utils/tickers_big3.pkl')
    print('done')
```

tickers 라는 함수를 정의하여 원하는 인덱스를 데이터프레임으로 저장하는 함수를 만들었다. 위에서 set 으로 저장한 티커들을 set.union 으로 묶어 준 뒤, 티커의 길이와 문자에 대한 argument 를 지정하였다. 먼저 my_list 는 티커명 뒤에 붙어있는 추가코드를 판별하기 위한 리스트다. W, R, P, Q 라는 문자열이 티커 뒤에 붙어있으면 모종의 이유로 거래가 정지된 티커를 의미한다. 만약 티커의 문자열 길이가 5 이상이고 티커의 끝에 my_list 의 인자가 있으면 해당 티커를 set 에서 지우고, 나머지를 sav_set 이란 set 에 저장한 뒤 데이터프레임으로 변환하고 .pkl 로 저장을 해준다.

tickers 함수의 인자를 \*args 로 받는 이유는 분석을 하다보니 특정 인덱스에서만 등장하는 티커에 대해 분석을 하고 싶을때가 생기기 때문이다. 만약 NASDAQ 에 있는 티커에 대해서만 분석을 하고싶다면, 인자로 sym3 만 리턴해주면 되는 원리다.
