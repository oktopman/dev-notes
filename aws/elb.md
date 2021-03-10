# Elastic Load Balancer

## ELB 란
ELB(Elastic Load Balancer)는 들어오는 어플리케이션 트래픽을 EC2 인스턴스, 컨테이너, IP주소, Lambda 함수와 같은 여러 대상에 자동으로 분산시킨다.  
단일 가용영역 또는 여러 가용 영역에서 다양한 어플리케이션 부하를 처리할 수 있다.  
세가지 로드밸런서를 제공하는데 고가용성, 자동 확장/축소, 강력한 보안을 갖추고 있다.  

## 왜 사용할까
AWS 클라우드상에 있는 것들은 탄력적이기 때문에 필연적으로 이러한 부분에 대응 할 수 있어야 한다.  
여러개의 EC2 인스턴스를 사용할때 유저가 어떤 인스턴스로 접근할지 IP 처리도 다 해줘야 하고 트래픽이 줄어듬에 따라 사용하는 인스턴스를 줄이게 된다면, 기존에 요청하는 IP 도 끊어줘야 하기때문에 관리부분에서 불편한 점이 많다.  
ELB 를 사용한다면 유저는 ELB로만 요청을 하면 된다.  
부하를 분산시켜주기도하고, 끊어진 인스턴스는 ELB가 알아서 요청을 보내지 않는다.  

## ELB 특징
### IP의 주소가 지속적으로 바뀜
분산 처리를 해야하기때문에 IP의 주소가 지속적으로 바뀐다. 그렇기 때문에 도메인기반으로 사용해야 한다. aws elb 를 생성하면 도메인이 하나 자동으로 부여된다.  

### Health Check
직접 트래픽을 발생시켜 인스턴스가 살아있는지를 체크한다. 인스턴스가 떠있는지 확인을 주기적으로 해서 끊어진 인스턴스로는 유저의 요청을 보내지 않는다.

### 3가지 종류
Application Load Balancer, Network Load Balancer, Clasic Load Balancer 가 존재한다.  
타겟그룹이란 EC2 인스턴스를 오토스케일링 할 수 있는 단위로 사용된다.  
`Clasic Load Balancer` 의 단점은 서버의 기본 주소가 바뀌면 로드밸런서를 새로 생성해야 하며, 하나의 주소에 하나의 타겟그룹으로만 보내게 된다.  
이러한 문제점은 서버의 구성이 비대해지고 마이크로 아키텍쳐를 구성하기 어렵다.  
예를 들어 회원, 정산 인스턴스가 따로 존재한다면 2개의 로드밸런서가 필요하게 된다.  

반면 `Application Load Balancer(ALB)` 는 path, 나 port 등에 따라 다른 타겟그룹으로 매핑할 수 있다.  
또한 타겟을 EC2 인스턴스, 람다, IP로도 연결이 가능하며 특정한 요청에 대해서는 서버없이 직접 응답메세지를 작성할 수 있기 때문에 마이크로 아키텍쳐 구성에 좋다.  
동일한 port 라도 path 등 에 따라 다르게 분기할 수도 있다.  
![image](https://user-images.githubusercontent.com/55048593/110647362-1ab86a80-81fb-11eb-8f53-1e9919668dc0.png)  
출처 : https://medium.com/harrythegreat/aws-%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%8B%B1-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-9fd0955f859e  

EC2 -> 로드밸런싱 -> 대상그룹 -> Create target group 을 누르면 타겟그룹을 생성할 수 있다.  
직관적인 설정부분은 제외하고 간단한 설정에 대해 설명한다.  
![image](https://user-images.githubusercontent.com/55048593/110651567-e5158080-81fe-11eb-9f07-c0d950e34e8b.png)
타겟 타입은 인스턴스, ip, lambda 를 설정할 수 있다. 타겟그룹에 포함할 인스턴스가 포함된 VPC 도 선택해야 한다.  
Next 를 누르면 타겟으로 지정할 인스턴스들을 볼 수 있다.  

![image](https://user-images.githubusercontent.com/55048593/110652256-8997c280-81ff-11eb-8177-01a35ee8fc28.png)
인스턴스들을 선택하고 라우팅하기 위한 포트도 설정 한 후 `Include as pending below` 를 누르면 타겟으로 지정이 되고 Create target group 을 통해 타겟그룹을 생성한다.  

로드밸런서를 생성하는 부분을 생략하겠다.  
로드밸런서를 생성후 선택하면 해당 로드밸런서에 리스너를 추가할 수 있다.  로드밸런서는 리스너 규칙을 사용하여 요청을 대상으로 라우팅 한다.  
리스너 추가 버튼을 누른후 클라이언트에서 로드밸런서로 요청할때 수신할 프로토콜과 포트를 선택하고 위에서 생성한 타겟그룹 까지 지정한 후 리스너를 추가한다.  
https 프로토콜을 사용하려면 보안정책과 SSL 인증서가 필요하다.  

![image](https://user-images.githubusercontent.com/55048593/110653282-73d6cd00-8200-11eb-9bc6-d1d11dd7e563.png)
추가한 리스너들에 대해 설명하자면, 클라이언트가 보낸 80 포트에 대해서 `HTTPS://#{host}:443/#{path}?#{query}` 로 리다이렉션 하는 리스너를 기본적으로 만들었다.  
그 아래 리스너들은 443이나 8443 으로 요청을 받게 되면 위에서 생성한 타겟그룹인 `stg-slcs`, `stg-cfs` 로 라우팅 한다.  
타겟그룹을 선택해보면 현재 속해있는 VPC, 로드밸런서, 프로토콜, 포트, 라우팅하고있는 인스턴스들을 볼 수 있다.  
인스턴스를 더 추가하고 싶다면 화면에 있는 Targets 탭안에 있는 `Register targets` 를 눌러서 추가할 수 있다.  

## Sticky Session
ELB 의 옵션으로 사용되며 로그인 세션을 유지하기 위해 사용한다.  
인스턴스가 이중화 되어있을때 로그인을 A 인스턴스를 통해 하였다면, ELB 는 그것을 쿠키에 저장해놓고 다음 요청을 받을때 마다 A 인스턴스로만 요청한다.  
Sticky Session 을 적용하고 싶다면, 타겟그룹을 선택한 후 Group details 탭의 Attibutes 를 편집하여 초,분,시간,일 단위로 설정 가능하다.  

## 참고
https://medium.com/harrythegreat/aws-%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%8B%B1-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-9fd0955f859e  
https://www.youtube.com/watch?v=rMx0MHlgpMY