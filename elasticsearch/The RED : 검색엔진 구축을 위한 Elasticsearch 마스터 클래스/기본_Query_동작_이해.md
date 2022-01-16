# Query
검색을 하기 위한 다양한 검색 질의 API

# 비용이 비싼 API
- 선현적 증가  
script queries : 문서가 많으면 많을수록 해당 쿼리의 비용은 비싸진다. 샤드를 어떻게 나눠서 사용할건지 샤드전략이 중요

- 초기 비용이 높은
fuzzy query  
regexp qurey  
prefix query  
wildcard qurey  
range query  

- 문서당 비용이 높은
script_score qureies

# Query and filter context
Elasticsearch 에서는 매칭된 검색 결과를 관련성 점수를 기반으로 정렬해서 넘겨줌  
관련성점수 : _score 라는 메타 데이터 필드에 반환되는 양의 floating point number. _score가 높을수록 관련성이 높다.

## Query context
문서가 질의 절과 얼마나 잘 일치하는지 확인. query context를 사용할 경우 _score 메타 데이터 필드에 관련성점수가 계산 되어 나옴

## Filter context
문서가 질의 절과 일치하는 지를 확인. 관련성은 보지않음.  
질의 결과에 대한 cache 를 하기 때문에 검색 성능을 높임

# Elasticsearch 에서 제공하는 다양한 형식의 Query API
- Full text query
- Term level query
- Script 와 Scoring query  
검색엔진에서 사용되는 대부분의 scoring query 는 비용적으로 성능적으로 리소스를 많이 사용하는 query 임.  
이런 유형의 query 사용을 방지 하고자 한다면 search.allow_expensive_queries 를 false로 설정(default = true)

## Boolean query
- must
이 절에 작성된 질의는 문서 매칭이 되어야 하며 score 에 반영됨
sql 질의로 비교하면 equals와 and 연산이 가능함

- filter
이 절은 작성된 질의는 문서 매칭이 되어야 하며 Score 반영 되지 않음
filter context 로 실행되며, must 절과 함께 사용될 경우 must 절의 scoreing 에 영향을 주지않고 filter 질의가 실행됨

- should
이 절은 작성된 질의는 문서 매칭이 될 수 있다.  
SQL 질의로 비교하면 in, or 연산 가능

- must not
이 절은 작성된 지의에 대한 매칭 문서가 없어야 하며 scoring 은 무시되고 filter context로 실행.  
sql 질의로 비교해보면 not equals 와 not 연산 가능

## match query
full text query 유형으로 질의에 사용하는 Text를 Matching 하기 전에 형태소 분석을 한 후 매칭함.  

- fuzziness
q=test f=test

