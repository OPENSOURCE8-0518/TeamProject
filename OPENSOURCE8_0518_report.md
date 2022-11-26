\# 시스템 설계서

## 캠퍼스 단위 개인화 커뮤니티

- 문서 갱신일 : 2022/11/21

---

# 목차

1. 개요
   - 서비스의 목적
   - 서비스 설명
   - 서비스 구조
2. 유사 서비스 분석
3. 사용된 오픈소스 소프트웨어
   - Elastic Search
   - Scrapy
   - LTR
   - IGListKit
   - Bert
   - KoNLPy
   - Django
4. DFD(Data-Flow-Diagram)

### Elastic Search

#### 설명

Apache Lucene을 기반으로 만들어진 오픈소스 소프트웨이며, 데이터를 검색, 저장, 분석하는데 유용한 도구이다.  
특히, 데이터 검색이 관계형 데이터 베이스에 비해 굉장히 빠른데 그 이유는 역색인을 통해 데이터를 저장하고 검색하기 때문이다.  
그렇기 때문에, 데이터 베이스에 저장된 데이터의 양과 관계없이 검색 속도는 일정하다.  
Elastic Search는 정형 데이터와 비정형 데이터를 모두 받아 들일 수 있으며, Kibana를 통해 검색 결과를 시각화할 수 있다.  
거기에 더해 검색 결과를 그냥 내보내는 것이 아니라 관련도가 높은 데이터를 우선적으로 보여준다.  
따라서 Elastic Search를 단순한 검색 엔진이 아닌 초고속 데이터 베이스 이면서 빅데이터 플랫폼으로 받아들이는 것이 더 올바르다.

#### 입출력데이터의 형식

기본적으로 Elastic Search는 모든 형태의 데이터에 대하여 쿼리를 할 수 있다.  
(쿼리란 데이터 베이스의 데이터를 변화시키지 않고 속성 데이터를 조사하는 것을 의미한다.)  
하지만, 이 플랫폼에서는 텍스트를 위주로 입력과 출력이 이루어질 예정이다.

#### 특징

1. Scale out  
   규모가 수평적으로 늘어날 수 있다.

2. 고가용성  
   데이터의 안정성을 높여준다.

3. Schemaless  
   Json 문서를 통해 데이터를 검색하므로 스키마가 필요 없다.

4. 역색인
   본문의 검색어를 먼저 추출한 뒤 검색어에 해당하는 문서(document)를 찾는다.

5. 실시간성  
   데이터 분석이 실시간으로 이루어지며 사용자가 원하는 결과를 바로바로 보여줄 수 있다.

#### 관계형 데이터 베이스를 계속 쓰는 이유

Elastic Search는 관계형 데이터 베이스가 아니기 때문에 완전한 대체는 할 수 없다.  
Elastic Search가 더 나은 영역이 있고, 관계형 데이터 베이스가 더 나은 영역이 있다.  
데이터가 빈번하게 업데이트 되는 상황이라면 관계형 데이터 베이스를 사용하는 것이 더 좋을 것이고,  
데이터의 삭제없이 로그성으로 남겨야 하는 상황이라면 Elastic Search가 더 유용하다.

#### 작동방식

Elastic Search는 단독으로 사용되기도 하고, ELK(Elastic Search/Logstash/Kibana) 스택으로 쓰이기도 한다.

##### Logstash

다양한 소스(DB, CSV 파일등)의 로그 또는 트랜잭션 데이터를 수집, 집계, 파싱하여 Elastic Search에 전달

##### Elastic Search

Logstash로 부터 받은 데이터를 검색 및 집계를 하여 필요한 정보를 획득

##### Kibana

Elastic Search의 빠른 검색을 통해 데이터를 시각화 및 모니터링

#### 시스템 구조와 데이터 구조

1. 클러스터

독립된 Elastic Search 시스템 환경으로, 1개 이상의 노드로 구성되며 각각의 클러스터는 내부적으로 데이터의 교환이 이루어지지 않는다.
하나의 클러스터를 다수의 서버가 구성할 수도 있고, 다수의 클러스터를 하나의 서버가 구성할 수도 있다.

2. 노드

실행중인 Elastic Search 시스템의 프로세스이다. 검색을 할 때 쿼리를 요청 받은 노드를 Coordinate 노드라고 부르며, Coordinate 노드는 각각의 샤드(다른 노드에 있는 샤드들 포함)들에게 검색 명령을 내린다. 샤드별로 검색이 분산 실행되고, 검색 결과를 다시 Coordinate 노드에게 전달한다. 검색 결과를 받은 Coordinate 노드는 자신이 받은 결과를 취합하여 사용자에게 응답한다.

3. 도큐먼트

Elastic Search에 저장되는 단일 데이터 단위.
사용자는 어떤 도큐먼트가 어떤 샤드를 가지는 지 알 수 없고 알 필요도 없다.

4. 샤드

색인&검색을 진행하는 작업 단위이며, Primary 샤드와 Replica 샤드로 나뉘어진다. 샤드는 클러스트내의 다른 노드에 분산되어 저장됨으로써 데이터의 신뢰성을 가진다.

5. 인덱스

도큐먼트의 논리적 집합으로, 1개 이상의 샤드로 구성된다.
인덱스 별로 Primary 샤드와 Replica 샤드의 세트 수를 설정한다.
Primary 샤드의 수는 처음에 설정하고 나면 수정이 불가능하지만 Replica 샤드의 수는 수정이 가능하다.

인덱스를 만드는 코드

<pre>
<code>
PUT books // books 라는 인덱스를 만듦
{
   "settings":{
      "index":{
         "number_of_shards": 5, //Primary 샤드의 개수를 5개로 설정
         "number_of_replicas": 1 //Replica 샤드의 개수를 1세트로 설정
      }
   }
}
</code>
</pre>

#### 참고

Datastream - 새로운 Elasticsearch 데이터 구조 이해하기 : https://youtu.be/JqKDIg8fgd8

[Elasticsearch] 기본 개념잡기 : https://victorydntmd.tistory.com/308

[ Elasticsearch ] 엘라스틱 서치 가볍게 살펴보는 개념 :) : https://youtu.be/MWItWo67F14

데이터 분석 플랫폼의 새로운 트렌드 "엘라스틱서치"(엘라스틱서치코리아 김관호 상무) : https://youtu.be/nOFB3jTnHEk

깃허브 주소 : https://github.com/elastic/elasticsearch
Elastic License 2.0 : https://www.elastic.co/kr/licensing/elastic-license/faq

#### 라이센스
Elastic License 2.0 (ELv2)

Elastic License 2.0은 Elasticsearch와 Kibana의 모든 무료 및 유료 기능의 소스 코드와 Elastic의 배포에 적용됩니다. 
- 제품을 다른 사람에게 관리형 서비스(Managed Service)로 제공할 수 없습니다. 
- 라이선스 키 기능을 우회하거나 라이선스 키로 보호되는 기능을 제거하거나 숨길 수 없습니다.
- 라이선스, 저작권 또는 기타 통지(notice)를 제거하거나 숨길 수 없습니다.


---

### Scrapy

#### 설명

Scrapy는 웹 사이트를 돌아다니면서 구조화된 데이터를 추출하기 위해 파이썬으로 작성한 오픈소스 프레임워크이다.

추출한 데이터를 데이터 마이닝, 정보 처리, 이력 기록 등 일련의 유용한 애플리케이션에 활용할 수 있다.
스크래피는 가볍고, 빠르고 확장성이 좋으며 비동기 네트워킹 라이브러리(synchronous networking library)인 Twisted를 기반으로 하기 때문에 매우 우수한 성능을 발휘한다.
XPath, CSS 표현식으로 HTML 소스에서 데이터 추출이 가능하고 현재 스크래핑허브(Scrapinghub), 플래스(Flax) 고스크레이프(GoScrape) 등 많은 기업들이 상용 지원을 제공하고 있다.

#### (Crawling)크롤링 이란

웹에는 수억개의 웹페이지가 있으며, 대부분의 페이지들은 수많은 정보를 가지고 있다.
최근 빅데이터가 대두되면서 이전에 작성되었던 페이지에서 유의미한 정보를 도출하기 위한 여러 가지 방법들이 논의되고 있는데 이를 Scraping 혹은 Crawling이라고 한다.

#### 입출력데이터의 형식

Scrapy는 웹 페이지에 있는 텍스트나 이미지 등 다양한 형태의 데이터들을 크롤링 할 수 있고 이 플랫폼에서는 주로 텍스트 형식의 데이터를 입출력한다.

#### 라이선스 정보
BSD License

#### 특징

1. Python 기반의 프레임워크로 쉬운 설정이 가능하다.

2. 단순한 스크랩 과정덕분에 크롤링 후, 바로 데이터 처리가 가능하다

3. Xpath, CSS표현식으로 HTML소스에서 데이터 추출이 가능하다.

4. webdriver를 사용하지 않는다

5. 다른 크롤링 오픈소스에 비해 가볍고 빠르다.

#### 구조

Scrapy의 아키텍처 구조에는 수집 주기를 설정하는 Scheduler가 존재하고 수집할 항목을 정의하는 Item과 수집 데이터의 저장 형식을 정의하는 Pipeline이 출력을 담당하는 형태를 가지고 있다.
그리고 Spiders를 통해 웹페이지의 정보를 수집한다.

1. Scheduler

스케쥴러는 수집 주기, 프록시 설정, 멀티 에이전트 설정 기능을 가지고 있어 Scrapy 엔진의 수집에 관련된 정책 사항을 설정하는 역할을 담당한다.

2. Item Pipeline

아이템 파이프라인은 수집하려는 데이터의 입출력을 담당한다. 수집하려는 항목을 아이템으로 정의하고 수집한 데이터의 형태를 파일 혹은 DBMS로 직접 입력이 가능하도록 설정할 수도 있다.

3. Spiders

수집하는 데이터를 크롤링하는 역할을 한다. 스파이더는 스케쥴러로부터 프로젝트에서 크롤링하는 정책에 따라 설정값을 요청하여 다운로더로부터 받은 크롤링 데이터를 아이템의 형태로 아이템 파이프라인에 전송한다.

4. Downloader

http, ftp 프로토콜을 해석하여 웹에 있는 데이터를 다운로드하는 역할을 담당한다.

#### 설치 및 실행 방법

##### Scrapy 설치

pip가 설치되어 있는 쉘에서 아래와 같은 명령어를 입력하여 Scrapy를 설치한다.
$ pip install scrapy

##### 프로젝트 생성 및 프로젝트 구조

scrapy라는 명령어를 통해 프로젝트를 생성하거나, 작성된 프로젝트를 실행할 수 있다.

$ scrapy startproject project명

##### item.py

크롤링해오는 데이터를 class로 받아오는 파일로 가지고 올 element를 작성한다.

item.py 작성 예시
```
import scrapy
from scrapy.item import ITem, Field
class APT2UItem(scrapy.Item):

depname=scrapy.Field() //단과대 element
trackname=scrapy.Field() //트랙명 element
trackcontent=scrapy.Field() //트랙의 설명 element

```
##### spider.py

데이터 수집 절차에 대한 수행 코드를 정의하는 파일로 해당 웹페이지에서 추출하고자 하는 정보들의 위치와 정보의 구조를 파악한다.
그리고 정규표현식을 이용하여 확보한 전체 웹페이지 정보 중 필요 데이터만 추출한다.

##### pipeline.py

파일, DB, 이메일 발송 등 수집된 데이터 처리 방식을 정의한다. 한글 처리를 위해 저장할 파일에 대한 utf-8 설정이 필요하다.

##### settings.py

프로젝트 모듈간 연결 및 기본 설정을 정의하는 파일이다.

#### DB 와 Scrapy 연결 방법

파일로 저장하는 방법과의 차이점은 Scrapy의 프로젝트 명을 APT2U에서 APT2U_DB로 변경한다는 점과 pipelines.py의 파일이 다르다는 점이다.
DB로 저장하기 위해서 크롤링의 결과를 저장할 Table을 생성해야 한다.

Scrapy 프로젝트 명 변경 ex) scrapy crawl APT2U_DB

"hs" Table 생성 스크립트 예시
```
CREATE TABLE `hs` (
`depname` varchar(200) DEFAULT NULL,
`trackname` varchar(200) DEFAULT NULL,
`trackcontent` varchar(500) DEFAULT NULL,
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 참고

Scrapy github : https://github.com/scrapy/scrapy

[Scrapy] 웹사이트 크롤링해서 DB 저장 하기 : https://uslifelog.tistory.com/54

(파이썬) Scrapy를 이용한 웹 데이터 크롤러(Crawler) 만들기 : https://blog.naver.com/PostView.nhn?blogId=rjs5730&logNo=221280231854&categoryNo=14&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=search

[티스토리 게시글 추천 시스템 만들기] #5 Scrapy로 스크랩하기 : https://weejw.tistory.com/536

How to Connect Scrapy to your PostgreSQL database : https://nicolas-bourriez.medium.com/how-to-connect-scrapy-to-your-postgresql-database-6d27230ec6f8

대용량 데이터를 수집하기 위한 생산성있는 웹 크롤러의 구조 : https://exmemory.tistory.com/81

#### 라이센스
- BSD (Berkeley Software Distribution) 

**BSD 소프트웨어는 소스 코드 공개의 의무가 없으며 상업적 이용에도 제한을 받지 않는다. 따라서 BSD를 이용하면 상용 프로그램 개발의 저작권 논쟁에서 굉장히 자유로워진다.**


---

### LTR

#### 설명 

Leaning To Rank, LTR은 머신러닝으로 계산된 스코어를 기반으로 정렬하는 것을 의미한다. 색인된 인덱스 파일에서 유저 쿼리가 들어와 Similarity를 계산할 때 머신러닝 알고리즘으로 학습된 데이터를 기반으로 스코어를 계산하여 랭킹모델에 사용한다는 것이다. 즉 머신러닝을 스코어를 만드는 데 쓰는 알고리즘이다.  LTR은 이러한 Ranking System을 mac에 적용하여 Query와 Item의 연관성이 점수를 예측한다.

주로 기존 검색 엔진들은 score 즉 정렬에 필요한 Similarity 알고리즘을 주로 TF-IDF 혹은 BM25 알고리즘에 의지하였다. 일반적으로 업채들도 검색엔진을 사용할 때 보통 이 알고리즘들에 기반하고 자신의 비즈니스 로직에 의한 가중치 값을 더 추가하여 서비스에 사용하였을 것이다. 

##### Ex)

구글 같은 웹페이지에서 검색 시 나오는 결과들을 연관성이 높은 순서로 정렬하는 것과 User의 특성에 따라 가장 User에게 알맞을 것 같은 Item을 추천 점수가 높은 순서대로 정렬하는 것. 



#### LTR을 위한 모델

LTR의 framework는 n개의 Query에 대해서 각 Item에 대한 m개의 feature가 있고 n개 Relevance score y (ex. user의 클릭수, 평점 등)이 있다. 이 학습 데이터로 모델 h를 만들어 테스트 데이터를 입력했을 때 relevance score를 예측한다. LTR에서 중요한 것은 '어떤 손실함수(Loss Function)를 활용해 모델을 학습하는가'인데,  Loss Function은 이러하다.



####  Loss Funtion

##### Point-wise : score를 머신러닝으로 생성

한 개의 입력 데이터에 대해 예측된 y값과 ground truth y값에 대한 차이만 계산하는 방법.



##### Pair-wise : 2개씩 비교하면 Order를 분류

두개의 item을 비교해 어느 Item이 Query와 가장 유사한지 판단하는 방법이다. Point-wise 방법을 사용하기 위해서는 테스트 데이터에 대한 ground truth 값이 모두 절대적이어야 하는데 현실에서는 그러한 데이터를 찾기가 어렵다. 이에 대한 해결책으로 Pair-wise 방법은 두 Item 사이의 상대적인 Relevancy를 학습한다. 



#####  List-wise : 전체 리스트를 한번에 분류

해당 방법은 Pair를 넘어서  Item list에 대한 모든 Relevancy를 계산한다. Ranking metric을 최대화하는 방법이기에 가장 좋은 성능을 기대할 수 있다.  



#### model의 생성

1. 모델의 Feature정의 

   검색 결과 및 대상의 따라 다르게 적용할 수 있다. 검색 대상이 공고와 같은 데이터라면 데이터의 단어의 빈도수,  본문 내용, 직종분류 등 만약 검색 대상이 사진이라면 픽셀 등으로 일반적인 머신러닝처럼 구성 가능하다. 

2.  Log  데이터를 활용 

    로그 데이터를 활용하여 Query에 대한 클릭 이벤트 같은 데이터로 모델 구성, 이후 검색 시 같은 쿼리에 대한 클릭에 대하여 모델링 적용.



#### Search Example

```  
POST tmdb/_search
{
  "query": {
      "sltr": {
              "params": {
                  "keywords": "rambo"
              },
              "model": "my_model", <- 모델 지정
          }
  }
}
```

해당 Query는 model을 돌리면서 전체에 대한 계속적인 비교작업으로 효율이 떨어짐



#### Rescore Top N

```
POST tmdb/_search
{
  "query": {
      "match": {
          "_all": "rambo"
      }
  },        
  "rescore": {   # 결과값 도출 이후 상위 1000개의 대해서만 재정렬 진행
      "window_size": 1000,
      "query": {
          "rescore_query": {
              "sltr": {
                  "params": {
                      "keywords": "rambo"
                  },
                  "model": "my_model",
              }
          }
      }
  }
}
```


#### 참조 

https://github.com/o19s/elasticsearch-learning-to-rank

https://elasticsearch-learning-to-rank.readthedocs.io/en/latest/logging-features.html

http://4four.us/article/2009/10/what-is-learning-to-rank

https://elasticsearch-learning-to-rank.readthedocs.io/en/latest/logging-features.html

####라이센스
Apache License 2.0
**아파치 라이센스의 경우 원저작물을 어떠한 방식으로든 사용할 수 있다.
2차 저작물(파생결과물/Derivative Works)에 대해서는 소스를 공개하지 않아도 되지만, 이 경우에는 수정된 부분을 표시하여야 한다.**


---

### IGListKit

#### 설명

IGListKit은 빠르고 유연한 목록을 작성하기 위한 데이터 중심 UICollectionView 프레임 워크이다.
IGListKit은 UICollectionView(여러 데이터를 관리하고 커스텀 할 수 있는 레이아웃을 사용해서 사용자에게 보여줄 수 있는 객체)에 표시할 개체의 배열을 제공해준다.

#### 특징

디핑: 가장 긴 공통 하위 시퀸스(_longest common subsequence_) 기술을 사용하여 선형 시간에서 컬렉션 간의 최소 차이를 찾는다. 이때의 시간복잡도는 O(n)이다. 데이터 배열 사이에서 모든 삽입, 삭제, 업데이트 및 이동을 찾는다. 따라서 사용자들이 좋아요를 누른 수를 실시간으로 업데이트 해줄 수 있다. 다음은 이를 구현하기 위한 코드 중 일부이다.

```
extension User: ListDiffable {
  func diffIdentifier() -> NSObjectProtocol {
    return primaryKey
  }

  func isEqual(toDiffableObject object: Any?) -> Bool {
    if let object = object as? User {
      return name == object.name
    }
    return false
  }
}
```

#### 구조

1. 섹션 컨트롤러 만들기

새로운 섹션을 만들기 위해 IGListSectionController를 참조한다. 'cellForItemAtIndex:``sizeForItemAtIndex:'를 이용하여 재정의 해준다.

2. UI 만들기

하나 이상의 섹션 컨트롤러를 생성후 IGListAdapter를 생성해준다.

3. 데이터 소스 연결

UI를 만들어 준 후 IGListAdapter의 데이터 소스 일부와 데이터를 반환해준다.

4. 최상위 Post 모델 설계

```
final class Post: ListDiffable {
  // 1
  let username: String
  let timestamp: String
  let imageURL: URL
  let likes: Int
  let comments: [Comment]
  // 2
  init(username: String, timestamp: String, imageURL: URL, likes: Int, comments: [Comment]) {
    self.username = username
    self.timestamp = timestamp
    self.imageURL = imageURL
    self.likes = likes
    self.comments = comments
  }
}
```

ListDiffable 프로토콜을 준수하는 Post모델이다. 여기서 주목할 점은 ListDiffable 프로토콜을 준수한다는 점이다.

```
func diffIdentifier() -> NSObjectProtocol {
  return as NSObjectProtocol
}
func isEqual(toDiffableObject object: ListDiffable?) -> Bool {
  return Boolean
}
```

이것이 피드에 관심도 별로 글을 정렬하기 위해(상단으로) 필수적인 것이다. LTR에서 정보를 받아와 해당 프로토콜을 통해 사용자의 관심도에 따라 글의 우선순위를 정렬해준다. 글에 대한 추천수를 실시간으로 받을때마다 서버로부터 새로운 데이터를 전달받는다. 이를 충돌없이 구현하기 위한 기능이다.

#### 참고

diffing : https://instagram-engineering.com/open-sourcing-iglistkit-3d66f1e4e9aa

IGListKit구조 : https://leejigun.github.io/IGListKit

####라이센스
MIT License
**소스코드 공개 필요 X, 동일한 라이센스 적용 X, 상업적 이용 O**

---

### BERT

#### 설명

구글에서 2018년 발표한 사전 훈련 언어 모델이다. 대량의 말뭉치를 사전 학습한 상태로 제공하기 때문에 사전 훈련 언어 모델이라고 한다. 압도적인 퍼포먼스로 단번에 NLP와 인공지능의 대표주자가 되었다.  

\* NLP (자연어 처리): 인간의 언어인 자연어를 기계에게 이해시키기 위한 방법들을 일컬어 자연어 처리, NLP라고 부르고 있다.

#### BERT의 데이터 학습 방법

1. 기존의 언어모델(LM)은 (글이 왼쪽에서 오른쪽으로 쓰여진다면) 왼쪽에 나온 단어들을 참조하여 다음에 나올 단어를 예측하는 단방향 방식이었다.
   그러나 BERT는 단방향으로만 예측하는 것은 그 문맥을 이해하는데 한계가 있다고 생각하여 예측해야 할 단어의 왼쪽 뿐만 아니라 미래에 나올 오른쪽 단어들에 대해서도 양방향으로 한꺼번에 학습을 하여 더 좋은 성능을 냈다.
   BERT에 입력되는 학습 데이터는 완성된 학습 데이터이므로 그 뒤에 단어를 예측하는 것이 아니라 중간에 어떤 단어를 가리고 (mask) 그것을 예측하기 위해 양방향으로 전체 문장을 분석하는 학습을 진행했다.

   ![image](https://user-images.githubusercontent.com/117567297/203068166-42e1c9a1-5c4d-4668-9094-cee863a01d5b.png)

   이렇게 BERT는 문장의 마지막 단어 예측이 아닌 중간의 단어를 예측하는 구조로 되어 있으며, 이를 Masked Langage Model(MLM)으로 부른다. 이것은 데이터에 대한 라벨링이 필요 없이 웹 상에 존재하는 각종 문장들을 긁어와서 학습 재료로 사용하여 저럼한 데이터 가공 비용으로 방대한 학습량을 달성할 수 있었다. 그 결과 자연어 처리 분야에 큰 성능 향상을 가져왔다.

   

2. BERT는 두 문장을 입력할 수 있는 구조로 만들어졌다. 두 문장을 분석하여 연속된 문장인지 아닌지 학습하는 장치가 있다. (NSP: Next Sentence Predicion)

   ex.1) BERT는 고성능의 획기적인 언어 모델이다. 모든 테스트에서 최상위 성적을 내었다.
   NSP 결과 : 1 (이어진 문장)

   ***

   ex.2) BERT는 고성능의 획기적인 언어 모델이다. 난 아무래도 천재인가봐.

   NSP 결과 : 0 (이어지지 않는 문장)

   

3. Bert는 문장을 생성하지 않고 문장을 분석하고 이해하는데만 집중하는 모델로 디코더 부분은 무시하고 인코더에만 집중한다. 문장을 정해진 길이의 vector 로 수치화 하는 것이다. vector의 각 요소는 문장의 의미를 인코딩한다.
   즉 BRET는 기술적으로 앞에서 말한 MLM과 다음 문장 예측 방식인 NSP로 학습하게 된다.

   

#### BERT의 구조

![image](https://user-images.githubusercontent.com/117567297/203068103-9ba99009-8807-4917-8a1a-951571bac6ce.png)

- 문장을 토큰화 해서 전체 문장벡터를 만든다. 두 문장이 입력되고 문장 시작은 CLS, 문장 끝은 SEP라는 특수한 토큰을 표시한다.

- 두 문장을 구분하는 Segment Embeddings를 만든다. (첫번째 문장 0, 두번째 문장 1)

- 각 토큰의 위치를 표시하는 Position Embeddings를 만든다.

- 이 세가지가 합산되어 입력으로 전달된다.

- 각 토큰에 해당하는 출력 벡터가 출력된다.

- 이후에 출력값에 적당한 레이어를 추가하여 값을 가공하여 원하는 테스크에 적용하게 된다. (파인튜닝: fine-tuning)

  

#### BERT의 문서 분류 수행 방법

![image](https://user-images.githubusercontent.com/117567297/203068272-678528ec-9835-4a0c-897b-c7feae4de19a.png)

[BERT 모델이 빈칸 맞추기로 학습을 마친 상황] BERT는 트랜스포머의 인코더 블록(레이어)을 여러 개 쌓은 구조이다. 그림에서 확인할 수 있다시피 이 블록(레이어)의 입력과 출력은 단어 시퀀스이며, 블록(레이어) 내에서는 입력 단어(벡터)를 두 개씩 쌍을 지어 서로의 관계를 모두 고려하는 방식으로 계산된다.
단어 단위로 토큰화한 뒤, 앞뒤에 문장 시작과 끝을 알리는 스페셜 토큰(CLS, SEP)를 추가한 뒤 BERT에 입력한다. BERT 모델의 마지막 레이어의 출력에서 CLS에 해당하는 벡터를 추출한다. 트랜스포머 인코더 블록에서는 모든 단어가 서로 영향을 끼치기 때문에 마지막 블록 CLS 벡터는 문장 전체(이 영화 재미없네요)의 의미가 벡터 하나로 응집된 것이라고 할 수 있다. 이렇게 뽑은 CLS 벡터에 작은 모듈을 하나 추가해, 그 출력이 미리 정해놓은 범주(우리 조의 경우 컴퓨터공학부 관련 키워드)가 될 확률이 되도록 한다. 학습 과정에서는 BERT와 그 위에 쌓은 작은 모듈을 포함한 전체 모델의 출력이 정답 레이블과 최대한 같아지도록 모델 전체를 업데이트한다. 이것이 파인튜닝(fine-tuning)이다.

#### BERT의 활용

- 두 문장의 연관성 파악

- 단일 문장의 분류 문제

- 태깅 문제

- 질문과 답 문제 등에 적용 가능(챗봇)

- BERT 모델(레이어) 위에 추가적인 레이어를 올려서 다른 새로운 태스크에도 적용할 수 있다.

  

### Bert-as-a-service

#### 설명

- BERT를 이용해 문장 인코딩 서비스를 하는 bert-as-service라는 오픈소스이다.

- 서빙 속도, 빠른 추론 속도, 적은 메모리 사용과 높은 확장성에 특화된 서비스이다.

- Bert-as-service는 단 두 줄의 코딩으로 문장을 정해진 길이의 vector로 표현해줄 수 있도록 하는 문장 인코더로서 BERT를 사용하고 ZeroMQ를 통해 서비스를 제공한다. 

  \* **ZeroMQ**(ØMQ, 0MQ, ZMQ)는 분산/동시성 애플리케이션에 사용하도록 개발된 고성능 비동기 메시징 라이브러리이다. ZeroMQ는 전용 [메시지 브로커](https://ko.wikipedia.org/wiki/메시지_브로커) 없이 동작이 가능하다. 라이브러리의 API는 [버클리 소켓](https://ko.wikipedia.org/w/index.php?title=버클리_소켓&action=edit&redlink=1)을 모방하도록 설계되었다. (위키백과)

#### Bert-as-service 설치

bert-as-service를 설치하는 가장 좋은 방법은 pip을 이용하는 것이다. server, client 모듈을 각각 나눠 설치할 수 있다.

<code>pip install -U bert-serving-server bert-serving-client</code>



#### 참고

[Github]-BERT 설명문서 : https://github.com/KPFBERT/kpfbert/blob/main/BERT-MediaNavi.pdf

위 문서의 유튜브 : https://www.youtube.com/watch?v=Pj6563CAnKs&t=832s

NLP 기술 : https://wannabenice.tistory.com/33

BERT 자료 : https://ratsgo.github.io/nlpbook/docs/language_model/tutorial/ , https://happy-obok.tistory.com/23

Bert-as-service 자료 : https://blog.naver.com/dmswldla91/222167659381 , https://blog.naver.com/dmswldla91/222167659381

[Github]-bert-as-service / README.md : https://github.com/hansonrobotics/bert-as-service/blob/master/README.md

####라이센스
Apache License 2.0
**아파치 라이센스의 경우 원저작물을 어떠한 방식으로든 사용할 수 있다.
2차 저작물(파생결과물/Derivative Works)에 대해서는 소스를 공개하지 않아도 되지만, 이 경우에는 수정된 부분을 표시하여야 한다.**

---

### KoNLPy

#### 개요
 Natural Language Processing(자연어 처리)는 텍스트에서 의미있는 정보를 분석, 추출하고 이해하는 일련의 기술을 의미한다. 이미 다양한 분야에서 사용되는 기술이며, 텍스트 요약, 대화 시스템(Siri, Bixby 등), 기계 번역 등에 사용되고 있다.
 그 중 KoNLPy는 한국어 텍스트의 정보 처리를 위한 파이썬 패키지이다.

#### 라이선스 정보
 GPL v3 혹은 그 이상.

#### 기능

##### 품사 태깅
 형태소 분석은 형태소를 비롯하여, 어근, 접두사/접미사, 품사(POS, part-of-speech) 등 다양한 언어적 속성의 구조를 파악하는 것이다. 품사 태깅은 이렇게 분석된 형태소의 뜻과 문맥을 고려해 그것에 각각 마크업을 하는 일이다.
ex) 가방에 들어가신다 -> 가방/NNG + 에/JKM + 들어가/VV + 시/EPH + ㄴ다/EFN
 KoNLPy에는 품사 태깅을 위한 여러 옵션이 존재하는데, 이들은 모두 문구를 입력받아 태깅된 형태소를 출력하는 동일한 입출력 구조를 지닌다. (어떤 옵션을 사용하더라도 입출력 데이터는 변하지 않는다는 뜻)
 이 옵션은 다양한 품사 태깅 클래스인데, 각 태거들은 모두 제각기 다른 기준에 따라 형태소를 분석하고 다른 태그를 부착한다. Kkma, Komoran, Hannanum, Okt, Mecab 의 5개 클래스가 존재하며, 이 중 필요한 것을 사용하면 된다.

##### 문장 분석, 추출
 품사 태깅 기능을 통해 분석된 문장에서 원하는 요소만을 추출할 수 있다. 예를 들어, "유일하게 항공기 체계 종합개발 경험을 갖고 있는 KAI는." 이라는 문장에서 명사만 추출하게 되면 '항공기', '체계', '종합', '개발', '경험' 만 추출해낸다.

#### 성능 분석
 각 클래스가 작동하는 방식이 다른 만큼 성능상에도 차이가 존재하는데, 응용할 프로그램의 특성에 맞춰 가장 적절한 것을 고르는 것이 좋다. 
 Kkma의 경우, 문장의 요소 하나하나를 분석하는 능력은 뛰어나지만, 띄어쓰기 알고리즘이 약해 ‘아버지가방에’ 등의 문장을 제대로 분석하지 못하며, 또한 ‘아이폰’을 ‘아이’와 ‘폰’으로 인식하는 등 신조어, 줄임말, 은어 등 사전에 포함되지 않은 단어를 잘 처리하지 못하는 모습을 보인다.
 Mecab의 경우, 뛰어난 띄어쓰기 알고리즘을 보유하고 있어 띄어쓰기가 되지 않은 문장을 가장 잘 분석하며 사전에 포함되지 않은 단어도 어느정도 정확하게 처리해내고, 클래스 로딩과 문서 처리 시간이 다른 클래스에 비해 10배 이상 빠른 모습을 보인다. (로딩 시간 0.0007sec, 실행 시간 0.2838sec. 문서 처리의 경우 10만 문자의 문서.) 그러나 문맥에 따라 분석하는 능력이 떨어져 ‘하늘을 나는 자동차’ 등의 문장에서 ‘나는’을 대명사 ‘나’에 보조사‘는’이 붙은 것으로 해석했다. 또한 window에서 사용이 지원되지 않는다.
 Okt의 경우 다른 부분은 그리 특출나지는 않지만, 그렇다고 아주 떨어지지도 않으며, 기존에 트위터에 사용되었던 패키지이기 때문에, 사전에 존재하지 않는 단어를 처리하는 능력이 아주 뛰어나다. 또한 로딩과 처리 속도 역시 빠른 편으로, 각각 1.5초, 2.5초가 소요되었다.

 목적으로 하는 시스템이 커뮤니티이므로, 커뮤니티의 다양한 신조어와 맞춤법 오류에도 뛰어난 정확성을 보이는 Okt가 가장 적절할 것으로 판단된다.

#### 사용
 KoNLPy는 파이썬 패키지이므로, 파이썬이 설치되어 있어야 한다. 우분투, CentOS, 맥, 윈도우, 도커 OS를 지원하며 파이썬 패키지 설치 명령어 ($ pip ~)를 통해 설치할 수 있다. 이후 사용할 태거 클래스와 모듈을 import하는 것으로 바로 사용할 수 있다.

 ex)
 <code>
 from konlpy.tag import Okt
 okt = Okt() # Open Korean Text(Okt) 클래스 객체 생성

 okt.morphs(u'분석할 문장')
 #-> 문장을 형태소 단위로 분해해 리스트로 반환 

 okt.nouns(u'분석할 문장')
 #-> 문장의 명사만 리스트 형태로 반환
 </code>

##### 데이터 입/출력
 KoNLPy는 JSON 파일과 txt 파일을 받아 사용할 수 있다. (패키지 내부 utils 모듈의 read_json(), read_txt() 매서드를 사용해 해당 확장자의 파일을 사용할 수 있다.) 이 파일에서 내용을 읽어와 사용할 태거 클래스의 매서드를 이용해 원하는 방식으로 분석/추출하면 된다. 분석/추출된 텍스트는 문자열 리스트의 형태로 반환되며 이 중 필요한 것만 골라 반환하도록 할 수 있다.
 반환된 문자열을 다시 파일로 저장해 내보내거나, 딕셔너리의 형태로 가공해 코드 내부에서 바로 이용할 수도 있을 것이다.

#### 참조
https://konlpy.org/ko/latest/# - KoNLPy: 파이썬 한국어 NLP
https://github.com/konlpy/konlpy - 깃허브 KoNLPy 패키지
https://datascienceschool.net/03%20machine%20learning/03.01.02%20KoNLPy%20%ED%95%9C%EA%B5%AD%EC%96%B4%20%EC%B2%98%EB%A6%AC%20%ED%8C%A8%ED%82%A4%EC%A7%80.html - 데이터 사이언스 스쿨 KoNLPy 한국어 처리 패키지
https://github.com/open-korean-text/open-korean-text - 깃허브 Okt 태거 클래스


---

### Django

#### 개요
Django는 파이썬으로 만들어진 무료 오픈소스 웹 애플리케이션 프레임워크(web application framework)이다.
쉽고 빠르게 웹사이트를 개발할 수 있도록 돕고, 고도의 데이터베이스 기반 웹사이트를 작성하는 데 있어서 수고를 더는 것이 장고의 주된 목표이다. 

장고는 콤포넌트의 재사용성(reusability)과 플러그인화 가능성(pluggability), 빠른 개발 등을 강조하고 있다. 또한, "DRY(Don't repeat yourself: 중복배제)" 원리를 따랐다. 설정 파일부터 데이터 모델에까지 파이썬 언어가 구석구석에 쓰였다.

Django는 인스타그램, NASA, 빗버킷, Disqus, 모질라에서 장고를 사용하는 것으로 알려져있으며 아파치 HTTP 서버에 mod wsgi 또는 mod python 모듈을 설치하여 서비스할 수 있다.


#### 특징
장고는 웹 개발에서 번거로운 요소들을 새로 개발할 필요 없이 내장된 기능만을 이용해 빠른 개발을 할 수 있다는 장점이 있다.

- ###### 구성
    장고는 파이썬으로 코딩한 모델을 관계형 데이터베이스로 구축해주는 모델(Model), HTTP 요청을 처리하는 웹 템플릿 시스템인 뷰(View), URL의 라우팅을 처리하는 URL 컨트롤러 (Controller) 로 구성된 MVC 디자인 패턴을 따른다.
    하지만 전통적인 MVC 디자인 패턴에서 이야기하는 컨트롤러의 기능을 프레임워크를 자체에서 하기 때문에 모델(Model), 템플릿(Template), 뷰(View)로 분류해 MTV 프레임워크라고 보기도 한다.

- ###### 모델
    모델은 데이터에 관한 정보를 담는다. 데이터에 대한 접근, 검증, 작동과 데이터 사이의 관계를 정의하는데, 일반적으로 각각의 모델은 데이터베이스에서 테이블에 해당한다.
    장고에서는 모델을 정의할 때 필드의 종류를 지정해줘야 하는데, 이것이 데이터베이스에게 컬럼 타입을 알려주고 HTML 폼으로 표시 될 때의 입력 타입도 내포하는 역할을 한다. 또한 장고의 폼 자동 생성 API를 이용할 때 데이터 검증에 쓰이기도 한다.

- ###### 뷰
    어떤 데이터가 표시될 것인지를 정의한다. 뷰는 HTTP 응답(response)를 반환해야 하며 응답의 종류는 웹 페이지, 리디렉션, 문서 등의 다양한 형태가 가능하다.
    장고에는 자주 사용되는 형태의 뷰를 패턴화하여 추상화 해둔 재사용 가능한 뷰들을 내장해 놓았는데, 이들을 제네릭 뷰(generic view) 라고 하며 원하는 제네릭 뷰를 상속한 클래스 뷰를 생성하여 사용할 수 있다.

- ###### 템플릿
    데이터가 어떻게 표시되는 지를 정의한다. 템플릿은 사용자에게 실제로 보여지는 웹 페이지나 문서를 다룬다.
    흔히 HTML 에 기반해서 템플릿을 만들며, HTML 에 동적인 요소를 추가하기 위해 파이썬의 일부 기능을 쓰게 도와주는 장고 템플릿 태그가 존재한다.

- ###### 내장 애플리케이션
    장고에는 아래와 같은 웹 개발에서 자주 쓰이는 애플리케이션이 'contrib' 패키지에 내장되어있다.
    - 확장가능한 사용자 인증 시스템
    - 동적 관리자 인터페이스
    - RSS 또는 아톰을 위한 피드 생성
    - 사이트맵 생성
    - 사이트 간 요청 위조(CSRF), 사이트 간 스크립팅(XSS), SQL 인젝션과 같은 해킹 시도에 대한     보안 대책

- ###### 외부 패키지
    내장된 요소들 중에 원하는 것이 없으면 장고를 위해 개발된 외부 패키지를 이용할 수도 있다. 장고에서 사용할 수 있는 패키지들을 모아놓은 웹사이트 Django Packages의 경우에는 3600개가 넘는 패키지가 등록되어있는 것을 알 수 있으며, 커뮤니티 멤버들의 꾸준한 기여 덕분에 많은 양질의 패키지들이 개발되고 있다. 
    이는 파이썬의 "건전지 포함"(Batteries Included)의 정신에 입각한 것으로 볼 수도 있다.

#### 참조

###### 위키피디아 - Django (웹 프레임워크) 문서
https://ko.wikipedia.org/wiki/%EC%9E%A5%EA%B3%A0_(%EC%9B%B9_%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC)#cite_note-14

###### 깃허브 Django 주소
https://github.com/django/django

#### 라이센스
- BSD (Berkeley Software Distribution) 

다음과 같은 사항을 요구하며 이를 준수하는 경우 개작/배포에 아무런 제한을 받지 않는다.
저작권자 표기. BSD 라이선스가 걸린 소프트웨어를 이용하는 경우, 해당 저작권자의 이름과 BSD 라이선스의 내용을 같이 배포해야 한다.
보증 부인. BSD 라이선스가 걸린 소프트웨어로 인하여 법률상/도의상 피해가 발생하더라도, 그 책임을 저작권자에게 물을 수 없다.

3조항(3-clause) 이상 라이선스를 사용하는 경우 다음 항목이 추가된다.
저작권자의 이름을 광고에 사용 금지. 별도의 서면 허락 없이 BSD 소프트웨어를 개발한 단체 또는 개인의 이름을 BSD 소프트웨어 또는 그 파생물의 추천 및 판촉 광고에 사용할 수 없다.

4조항(4-clause) 라이선스를 사용하는 경우 다음 항목이 추가된다.
광고에서의 저작권자 표기. BSD 소프트웨어를 사용 또는 포함한 것(파생물 포함)을 광고할 때에는 광고에 저작권자를 명시해야 한다.

**BSD 소프트웨어는 소스 코드 공개의 의무가 없으며 상업적 이용에도 제한을 받지 않는다. 따라서 BSD를 이용하면 상용 프로그램 개발의 저작권 논쟁에서 굉장히 자유로워진다.**

