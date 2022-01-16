# Lucene ?
오픈소스기반의 검색 라이브러리. 검색엔진이 갖춰야 하는 기본기능인 색인, 검색, 형태소 분석을 제공.  
elasticsearch 는 lucene 기반의 검색엔진.  

Lucene 기본 개념
- Index(인덱스)
- Document(문서)
- Field(필드)
- Term(용어)

 ## 색인
 IndexWriter -> Create Segment File -> Document Writing -> Commit  

 ## 검색
 IndexWriter 로 색인후, IndexSearch 로 검색하는 과정  
 IndexReader -> IndexSearcher -> Query -> Sort -> Reduce -> Merge  


 ## Segment
 세그먼트란 엘라스틱서치에서 문서의 빠른 검색을 위해 설계된 자료구조이다.  

## 형태소분석
입력 받은 문자열에서 검색 가능한 정보 구조로 분석 및 분해하는 과정  
구성요소
- Analyzer
- CharFilter
- Tokenizer
- TokenFilter

Input Text -> Character Filter(불필요한 문자 제거) -> Filtered Text -> Tokenizer -> Make Token -> Token Filter -> Filtered Token -> Output Token
