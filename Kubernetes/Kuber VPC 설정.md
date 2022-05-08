

vpc-0a06a20446cb6dcd7
https://www.44bits.io/ko/post/understanding_aws_vpc

아키텍처
AWS 계정 생성
IAM 유저생성-IAM그룹, IAM 유저, 로그인,MFA 설정
네트워크구성-VPC, Subnet생성, route table, internet gateway 설정
NAT서버 구축
 

### VPC 생성
## 참고 url
https://www.44bits.io/ko/post/understanding_aws_vpc


Cloudformation으로 생성
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started-console.html
생성template: https://amazon-eks.s3-us-west-2



## VPC 대역대 및 Subnet 설계 
 - vpc 대역대 XXX.XXX.XXX.XXX/24
    public subnet 2개  
      XXX.XXX.XXX.0/27 
      XXX.XXX.XXX.32/27
    private subnet 4개
      XXX.XXX.XXX.64/27
      XXX.XXX.XXX.96/27
      XXX.XXX.XXX.96/27


====================================================

### EKS Cluster 생성
## 참고 url
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-cluster.html


====================================================

### 관리형 노드 그룹
## 참고 url
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/managed-node-groups.html








  igw-6735cc01 