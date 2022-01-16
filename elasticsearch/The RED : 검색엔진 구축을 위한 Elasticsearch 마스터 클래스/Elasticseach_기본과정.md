# Directory 구성
- bin: 실행스크립트
- config : 설정 파일
- data : index
- jdk.app : jdk
- lib : elasticsearch 와 dependency jar
- logs : elasticsearch 실행 로그
- modules : elasticsearch 에서 사용하는 모듈 jar
- plugins : elasticsearch 에서 사용하는 플러그인 jar

path.data, path.logs 를 변경하면 저장되는 path 변경 가능  

# Configuration
 elasticsearch.yml
 - cluster.name : 모든 노드와 클러스터 이름이 공유 되었을 때 연결 가능. 같은 클러스터 이름 사용하지 않도록 주의
 - node.name : 노드의 용도와 목적을 이해하기 위한 사람이 읽을 수 있는 식별자로 작성  

 노드의 역할(12가지)
 master
 - 클러스터의 상태를 변경
 - 클러스터의 상태를 모든 노드에 게시
 - 전역 클러스터 상태를 유지
 - 샤드에 대한 할당과 클러스터 상태 변화 게시

 data
 - 문서의 색인 및 저장
 - 문서의 검색 및 분석
 - 코디네이팅

 data_content
 - 색인과 검색 요청이 많은경우에 해당 역할을 부여

 data_hot  
 data_warm  
 data_cold  
 data_frozen  
 ingest
 - 클러스터 내 적어도 하나 이상의 ingest 역할을 하는 노드 필요  
 ml  
 remote_cluster_client  
 transform  
 voting_only  

역할을 이렇게 많이 나누어 놓아도 설정을 하지않으면 자동으로 되는건 없음.  
elasticsearch 에서 잦은 문제 경험 -> disk full 로 인한 오류.    
es 는 90% 까지만 저장되고 그 이후는 색인 되지않음.  
오류 막는방법 : 저장 공간이 충분한 경로에 색인 데이터와 로그를 기록하고 저장할 수 있도록 경로 설정  
path.data 는 다중 구성이 가능 

```
# 단일구성
path:  
    data: /var/lib/elasticsearch  
    log: /var/log/elasticsearch  


# 다중구성
path:
    data:
        -/mnt/elasticseach_1
        -/mnt/elasticseach_2
        -/mnt/elasticseach_3

# 디스크 노드에서 사용하는 스펙을 동일하게 맞추는게 좋음. 저장공간마다 성능(색인 및 질의)이 다르기때문
```

cluster 구성시 네트워크 상의 노드들을 노출 시키기 위해 network.host 설정 -> 인스턴스에 부여된 IP 주소를 작성  
docker 로 클러스터 구성시 : network.host 와 network.publish_host 설정  
docker 로 싱글노드 구성시 : network.host: "0.0.0.0" 으로 구성  
cluster 환경 구성시 묶어야 하는 노드들을 발견하기 위한 설정  

# Heap 설정
bootstrap.memory_lock: true  
http.port  
http.max_content_length  
content가 100mb를 넘는경우가 있기때문에 압축전송 추천!

# Components
클러스터 > 노드 > 인덱스 > 샤드 > 도큐먼트  

클러스터
- cluster.name 설정이 중요
- 기본값은 elasticseach

노드
- es 인스턴스가 시작될 때 마다 실행
- 노드들의 모음이 cluster
- 단일 노드로도 실행가능
- 설정기본 : node.name과 node.roles
- node 의 이름을 지정하고 역할을 정의해서 용도와 목적에 따라 운영
- master, data, cordinating 노드를 가장 많이 사용

인덱스
- 분산된 shard에 저장된 문서들의 논리적 집합

샤드
- 물리적인 데이터가 저장 되어 있는 단위
- Indexing 요청이 있을때 분산된 노드에 위치한 shard로 문서를 색인
- primary shard : 색인 요청이 들어오면 가장 먼저 생성해서 문저를 저장하는 샤드. 이를 기반으로 데이터를 복제하여 활용
- replica shtard : primary shard 를 기준으로 복제하는 샤드. 검색 성능을 개선하기 위한 용도로 사용