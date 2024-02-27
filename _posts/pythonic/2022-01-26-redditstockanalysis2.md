---
layout: post
title: Sentiment analysis with Reddit API on /r/stocks - Pt.2
tags: 
categories: pythonic
---

## Contents

- Text Cleaning using cleantext
  - Unix time and the Unix Epoch
- Sentiment Analysis using flair
- Ticker parsing with list comprehensions

## Text Cleaning using cleantext

Reddit API 로 /r/stocks 의 daily 포스팅을 성공적으로 가져왔으면, 해당 포스트들을 분석 하기 전 text cleaning 을 진행 해 주어야 한다. Reddit 의 포스트들은 기본적으로 embedded markdown 으로 작성되고 emoji 도 지원하기 때문에, raw text 들을 긁어오면 line break 와 마크다운 형식의 지저분한 텍스트를 마주한다.

하지만 다행히도 /r/stocks 의 사용자들은 레딧 치곤 굉장히 점잖은 편이다. 포스팅을 할 때 사진을 첨부하지 않고, 줄바꿈도 거의 하지 않고 레딧의 특유의 밈 문화도 보이지 않는다 (물론 이 역할은 /r/stocks 의 큰형격인 [/r/wallstreetbets](https://www.reddit.com/r/wallstreetbets/) 가 담당하고 있다).

그렇기 때문에 수동 parsing 이나 NLTK 같은 무거운 라이브러리로 클리닝을 할 필요 없고, 가벼운 clean-text 라이브러리를 사용하였다.

``` bash
!pip install clean-text
```

``` python
from cleantext import clean
import datetime

def cleaner(brr):
    cleantitle = [clean(text, lower=False) for text in brr['title']]
    brr['title'] = cleantitle

    cleanselftext = [clean(text, lower=False) for text in brr['selftext']]
    brr['selftext'] = cleanselftext

    realtime = [datetime.datetime.fromtimestamp(time) for time in brr['created_utc']]
    brr['realtime'] = realtime

    score = [int(score) for score in brr['score']]
    brr['score'] = score
    return brr

```

cleantext 모듈은 clean 이란 강력한 함수를 지원한다. to_ascii, no_emails 등 약 20개의 세부 인자를 제공하며, 굉장히 빠르다는게 큰 장점이다. 가벼운 모듈이기 때문에 딱히 documentation 도 없고, [pypi 페이지](https://pypi.org/project/clean-text/) 에 나와있는 설명만으로도 가동이 가능하다. 이 cleantext 를 기반으로 cleaner 이란 함수를 만들었다. Reddit API 로 가져온 데이터프레임을 인자로 받으며, 각 칼럼에 대해 파싱을 진행한다. title, selftext 같은 str 값들은 전부 clean(text, lower=False) 로 파싱을 해주며, 해당 포스트의 추천수인 score 을 float 에서 int 로 바꿔준다.

#### Unix time and the Unix Epoch

created_utc 라는 칼럼값은 해당 포스트가 생성된 시간이다. 하지만 해당 시간은 우리가 익숙한 datetime 이 아닌 [Unix Epoch time](https://en.wikipedia.org/wiki/Unix_time) 으로 로깅이 된다. 줄여서 Unix time 은 **Unix Epoch** 라는 시점부터 초단위로 시간을 표현하는 값인데, 이때 Unix Epoch 은 1970년 1월 1일 00시 00분을 의미한다. 대부분의 OS 는 시간을 계산할 때 이 Unix time 을 이용하고, 대부분의 언어는 기본 모듈에 Unix time 을 datetime 으로 바꿔주는 함수를 제공한다.

```python
import time
import datetime
print(time.time()) #Current Unix time
print(datetime.datetime.fromtimestamp(time.time())) #Current datetime
```

파이썬의 경우 time모듈의 time 함수가 현재 Unix time 을 리턴해주며, datetime모듈의 datetime 클래스의 fromtimestamp 함수가 Unix time 을 datetime 으로 바꿔서 리턴해준다. 그렇기 때문에 cleaner 함수에 이 부분을 추가하여 모든 포스팅이 작성된 시간을 datetime 으로 바꿔주었다.

> Unix time 관련해서 Y2K 사건과 비슷한 [Y2K38](https://en.wikipedia.org/wiki/Year_2038_problem) 사건이 예정되어 있다. Unix time 을 32bit integer 값으로 저장하는 시스템들은 언젠간 $$2^{32}$$ bit 까지 도달한 Unix time 을 처리하지 못하고 Unix time 을 Unix Epoch의 $$-2^{31}$$ 초 **전**, 즉 1901년 12월 13일 로 롤백 할 것이다. 이 롤백은 2038년 1월 19일로 예정되어 있기 때문에 Y2K38 이란 이름이 붙었고, Y2K 와 준하는 사건이라고 보는 사람들도 있다. 

## Sentiment Analysis using flair

감성분석을 해주는 파이썬 라이브러리는 굉장히 많이 존재한다. 대표적으로 NLTK 가 있는데, NLTK 라이브러리는 모듈 뿐만 아니라 데이터셋, 그리고 튜토리얼 까지 포함되어 있는 라이브러리라 굉장히 무겁다. 그렇기 때문에 NLTK 대신 훨씬 가볍고 감성분석의 새로운 강자로 떠오르고 있는 flair 모듈을 사용하였다.

[flair](https://github.com/flairNLP/flair) 은 sota NLP 모델들을 제공하는 모듈이다. NER, PoS, text embedding 같은 NLP 라이브러리의 기본을 다 갖추고 있고, PyTorch 에 직접 구축되어 있기 때문에 새로운 데이터와 모델생성이 쉽다. 심지어 flair 가 지원하는 대부분의 모델들은 [HuggingFace model hub](https://huggingface.co/models?library=flair&sort=downloads) 에 호스팅 되어있기 때문에 직접 다운로드가 가능하다. 이 프로젝트에선 flair 의 TextClassifier 모델을 사용하여 간단한 Positive/Negative rating 감성분석을 진행하였다.

```bash
!pip install flair
```

```python
import flair
from tqdm import tqdm

sentiment_model = flair.models.TextClassifier.load('en-sentiment')

def flairme(brr):
    probability = []
    sentiment = []
    print('Analyzing Sentiment..')
    for text in tqdm(brr['selftext'].to_list()):
        sentence = flair.data.Sentence(text)
        sentiment_model.predict(sentence)
        probability.append(sentence.labels[0].score)
        sentiment.append(sentence.labels[0].value)
    brr['probability'] = probability
    brr['sentiment'] = sentiment
    return brr
```

위 코드는 brr 이란 데이터프레임을 인자로 받아 감성분석을 진행해주는 함수이다. flair 의 TextClassifier 은 문장을 파싱하여 해당 문장의 감성이 긍정적인지 부정적인지 리턴해주고, 해당 라벨의 score 도 리턴해준다. 하지만 여기서 주의할 점은 TextClassifier 은 그저 str 을 인자로 넣으면 감성을 리턴하진 않고, flair.data.Sentence 라는 함수로 str 을 flair.data.Sentence type 으로 전처리를 해주어야 한다. 해당 타입은 string-like 이며, iterable 하며 해당 string 값의 토큰을 iterate 한다. **즉 flair.data.Sentence 는 NER tagging 을 진행해준다**.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/pythonic/redditstockanalysis2/1.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

위 코드를 보면 이해하기 쉽다. testdata는 Reddit API 로 가져온 약 1000개의 포스팅이 담겨있는 데이터프레임이며, 포스팅 본문은 ['selftext'] 칼럼에 있다. [0] 인덱스로 하나의 포스트의 type 을 확인해 보면 일반적인 str 이지만, flair.data.Sentence(text) 로 파싱하여 sentence 의 type 을 확인 해 보면 flair.data.Sentence 이다. 이 sentence 는 str 와 언뜻 보면 비슷하지만 [0]로 인덱싱을 하면 하나의 문자열을 리턴하지 않고 token 을 리턴하는 것을 알 수 있다.

```python
sentiment_model.predict(sentence)
probability.append(sentence.labels[0].score)
sentiment.append(sentence.labels[0].value)     
```

처음에 선언한 sentiment_model 엔 predict 라는 함수가 있다. 해당 함수가 실질적인 감성분석을 해주는 함수며, 내부적으로 인자로 받은 sentence 에 label 을 추가하여 리턴한다. sentence.labels[0].values 가 해당 문장의 Positive/Negative 태그이며, sentence.labels[0].score 가 해당 문장의 감성태그의 신뢰도 이다. 이 값들을 리스트에 저장하여 기존 인자로 받은 데이터프레임에 새로운 칼럼으로 선언하면 간단한 감성분석이 완료되는 것이다.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/pythonic/redditstockanalysis2/2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

리턴해주는 데이터프레임을 불러오면 해당 포스트에 대한 감성과 신뢰도가 추가된 것을 볼 수 있다.

> 아무리 flair 같은 가벼운 모듈을 사용하였더라도 평균적으로 길이가 1000이 넘는 포스트를 NER tagging 하고 해당 태그를 통해 감성분석을 진행하기 때문에 해당 함수는 결코 빠르다고 할 수는 없다. tqdm 으로 확인 해본 결과 약 1000개의 포스팅을 순회하는데 걸리는 시간은 약 6분이다.
>
> 위 사진의 21행을 주목해보자. 해당 포스트에서 Peloton 에 진입하기 좋은 시점인지 조언을 구하는데, 포스팅에 대한 감성분석은 Negative 이고 신뢰도가 무려 1이다. 공교롭게도 해당 포스트가 작성되고 이틀후, Peloton 은 하루만에 -23% 하락했다. 해당 포스트의 작성자마저 Peloton 에 대해 부정적으로 생각하고 있었지만 진입하기 좋은 시점이냐고 물어본 셈이다.

## Ticker Parsing using list comprehension

포스트에 대한 감성분석을 진행 하면 어떤 포스트가 감성분석으로 positive 인지 negative 인지 알 수 있다. 하지만 앞서 말했듯이 하나의 포스트는 평균적으로 1000자를 넘어가며, 사람들이 어떤 기업에 대해 어떻게 생각하는지 알기 위해 본문을 하나하나 읽을 수는 없다. 그렇기 때문에 이전에 만들어 둔 티커 리스트를 대차대조하며 어떤 포스트에 어떤 티커가 등장하는지 보여주는 함수를 짰다.

```python
def tickercount(brr):
    sentences = [flair.data.Sentence(post) for post in brr['selftext']] #한 포스트를 센텐스화
    sentences = [sentence.tokens for sentence in sentences] #센텐스를 토큰화
    sentence_tokens = [[str(token) for token in sentence] for sentence in sentences]
    # 토큰을 str list 화

    tickers = pd.read_pickle('utils/tickers_big3.pkl')
    tickerlist = list(tickers['tickers'])

    fullcount=[]
    print('Analyzing tickers..')
    for sentence in tqdm(sentence_tokens):
        sentence = str(sentence)
        sentence = clean(sentence, lower=False)
        tickercount = []

        for ticker in tickerlist:
            count = 0
            nocount = 0
            secondcount=[]
            if ticker in sentence:
                count +=1
            else:
                nocount +=1
            if count > 0:
                tickercount.append(f'{ticker} was mentioned {count} times in this post.')

            secondcount.append(tickercount)
        fullcount.append(str(tickercount))
    brr['tickerinfo'] = fullcount
    replacer = brr.tickerinfo.replace('[]', 0)
    brr['tickerinfo'] = replacer
    return brr
```

처음 함수를 짜기 시작했을 때 간단하게 해당 티커가 포스트에 있는지 확인하는 방법을 str.isin() 으로 테스트를 해봤으나, str.isin() 은 정확하지 않기 때문에 flair.data.Sentence 가 기본적으로 제공하는 토큰 파싱을 기반으로 티커가 있는지 확인하였다. 

```python
    sentences = [flair.data.Sentence(post) for post in brr['selftext']] #한 포스트를 센텐스화
    sentences = [sentence.tokens for sentence in sentences] #센텐스를 토큰화
    sentence_tokens = [[str(token) for token in sentence] for sentence in sentences]
```

먼저 데이터프레임의 포스트가 있는 ['selftext'] 칼럼의 모든 포스트를 flair.data.Sentence type 으로 바꾸었다. 해당 데이터타입은 .tokens 라는 인자가 있으며, 해당 문장들을 토큰으로 바꿔준 뒤 모든 토큰을 다시 str 값으로 바꾸었다.

```python
    tickers = pd.read_pickle('utils/tickers_big3.pkl')
    tickerlist = list(tickers['tickers'])
```

티커가 저장되어 있는 데이터프레임을 부르고, 모든 티커를 담고있는 tickerlist 를 정의하였다.

```python
    fullcount=[]
    print('Analyzing tickers..')
    for sentence in tqdm(sentence_tokens):
        sentence = str(sentence)
        sentence = clean(sentence, lower=False)
        tickercount = []

        for ticker in tickerlist:
            count = 0
            nocount = 0
            secondcount=[]
            if ticker in sentence:
                count +=1
            else:
                nocount +=1
            if count > 0:
                tickercount.append(f'{ticker} was mentioned {count} times in this post.')

            secondcount.append(tickercount)
        fullcount.append(str(tickercount))
    brr['tickerinfo'] = fullcount
    replacer = brr.tickerinfo.replace('[]', 0)
    brr['tickerinfo'] = replacer
    return brr
```

함수의 해당 부분이 실제로 포스팅에 어떤 티커가 등장하는지, 그리고 몇번 등장하는지 알려주는 함수다. 처음엔 list comprehension 으로 이중 for 문을 해결하려고 했지만, list comprehension 에서는 기본 for 문 처럼 새로운 변수를 선언하는 것이 불가능 하기 때문에 어쩔 수 없이 for 문으로 구축하였다. 다행히도 해당 함수는 flairme 함수보다 빠르고, 약 1000개의 포스팅을 파싱하는데 약 3분이 걸렸다.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/pythonic/redditstockanalysis2/3.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

최종적으로 리턴되는 데이터프레임은 다음과 같다. 해당 포스트에 티커에 대한 멘션이 없으면 0을, 티커에 대한 멘션이 있으면 해당 티커와 티커가 몇번 멘션되는지 추가가 되어있다.

이 tickercount 함수를 짜면서 아직 해결하지 못한 문제들이 몇가지 있다. 예를 들어 /r/stocks 의 사용자들은 티커명 말고도 ATH (all time high), MDD (maximum draw down), VIX (Volitality index) 같은 약어들을 사용한다. 해당 약어들도 중요한 지표이기 때문에 모든 약어를 정리한 파일이 필요하다. 그리고 지금까지의 포스팅은 /r/stocks 감성분석의 제일 기초적인 뼈대만 마련한 것이지, 나아갈 수 있는 방향은 무궁무진하다. 해당 티커와 감성과 실제 주가의 시계열 분석과 같은 모듈을 계속 추가해 나갈 수 있을 것이다.



