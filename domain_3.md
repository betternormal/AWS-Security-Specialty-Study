# Domain 3. Infrastructure Security
네트워크 보안은 데이터 흐름(Traffic Flow)을 이해하는 것이 핵심입니다. 아래 흐름도를 머릿속에 그려보세요.

1. 사용자가 접속 시 CloudFront/WAF/Shield 통과.
2. VPC 경계에서 NACL 검사.
3. 인스턴스 바로 앞에서 Security Group 검사.
4. 내부 통신 시 VPC Endpoint 활용.

### **1. 네트워크 보안 및 접근 제어 (Network Security & Access Control)**

VPC 내 트래픽 흐름 제어 및 내/외부 접속에 대한 보안입니다.

- **Security Groups (SG) vs NACLs (네트워크 ACL)**
    - **Security Groups (Stateful)**: 인스턴스/ENI 레벨 방화벽.
        - 운영 중 **Rule을 변경(제거)하더라도, 기존에 수립된 연결(Connection)은 타임아웃될 때까지 지속**됩니다. (즉시 차단 안 됨).
    - **NACLs (Stateless)**: 서브넷 레벨 방화벽.
        - 허용/거부 규칙 모두 가능.
        - **Rule 변경 시 즉시 적용**되어 연결을 바로 끊을 수 있습니다.
        - *Ephemeral Ports (임시 포트)*: 클라이언트가 연결을 위해 랜덤으로 사용하는 포트(1024-65535 등)에 대해, 되돌아오는 트래픽(Outbound/Inbound)을 NACL에서 열어줘야 함을 잊지 말아야 합니다.
- **보안 연결 및 격리 (Secure Connectivity)**
    - **Bastion Host (점프 서버)**: Public 서브넷에 위치하여 SSH로 접속 후 Private 인스턴스로 점프하는 용도.
        - *보안 설정*: SG에서 관리자의 **특정 IP로만 22번 포트 제한**. Private 인스턴스의 SG는 Bastion Host의 **Private IP** 또는 **Bastion의 SG ID**를 소스로 허용.
    - **AWS Client VPN**: OpenVPN 기반. 클라이언트가 AWS VPC 내 **Private IP**로 안전하게 접속 가능하게 함.
    - **VPC Peering**: 두 개의 VPC를 AWS 내부 망을 통해 마치 하나의 VPC처럼 연결.
    - **VPC Endpoints**: 인터넷 게이트웨이 없이 Private Network 내에서 AWS 서비스(S3, DynamoDB, CloudWatch, SSM 등)에 접근.
    - **VPC PrivateLink (Interface Endpoints)**: 서비스를 안전하게 노출하는 기술. 특히 SaaS 형태나 **1,000개 이상의 수많은 VPC**에 내 서비스를 노출해야 할 때 확장성 있고 안전한 방법.
- **AWS Network Firewall**: VPC를 위한 관리형 네트워크 방화벽. 3계층(IP/Port)부터 7계층(도메인, IPS/IDS)까지 심층 방어 제공.

### **2. 엣지 및 경계 보안 (Edge & Perimeter Security)**

외부 공격 차단 및 콘텐츠 전송 보안입니다.

- **CloudFront (CDN)**
    - **보안 기능**: DDoS 방어(Shield 결합), 지리적(Geo) 제한(Allow/Block), WAF 통합.
    - **Signed URL**: **IP 주소** 및 **접속 시간** 제한 가능. 유료/프리미엄 콘텐츠를 인가된 사용자에게만 전달할 때 사용.
    - **Field-Level Encryption**: POST 요청 시 민감 정보(신용카드 등)를 엣지 로케이션에서 **비대칭키**로 즉시 암호화하여 오리진까지 전달.
    - **헤더 처리**: Origin에 인증이 필요한 경우  헤더를 Forwarding 하도록 설정해야 함.
        
- **AWS WAF (Web Application Firewall)**
    - **7계층 보호**: SQL Injection, XSS 등 일반적인 웹 공격 방어.
    - **로깅**: Web ACL 로그를 **CloudWatch Logs, S3, Kinesis Firehose**로 전송하여 분석 가능.
- **AWS Shield**: DDoS 방어 서비스 (Standard는 무료/L3-L4, Advanced는 유료/보호 및 24/7 대응).
- **Route 53 DNSSEC**: 도메인 네임 시스템 보안 확장. **비대칭 키** 서명을 사용하여 DNS 스푸핑(Spoofing) 및 중간자 공격을 방지.
- **AWS Firewall Manager**: AWS Organizations와 연동하여 조직 내 **모든 계정**의 방화벽 규칙(WAF, Security Group, Network Firewall 등)을 중앙에서 통합 관리.

### **3. 애플리케이션 보안 및 기타 (Application & Compliance)**

애플리케이션 레벨의 인증 및 규정 준수 서비스입니다.

- **API Gateway**
    - Lambda 등을 외부에서 REST API로 호출할 때의 진입점.
    - **내부 인증**: **IAM Role** 및 Policy 사용 (SigV4 서명).
    - **외부 인증**: **Amazon Cognito** User Pools/Identity Pools 연동.
- **AWS SES (Simple Email Service)**: 대량 이메일(Bulk emails) 전송 서비스. 스팸으로 분류되지 않도록 DKIM/SPF 설정 등의 보안 구성이 중요 (시험에서는 주로 대량 메일 발송 주체로 언급됨).
- **AWS Artifact**: 규정 준수 보고서(ISO, PCI-DSS 등)를 **On-demand**로 다운로드 및 조회할 수 있는 포털. (감사인이 자료 요청 시 사용).


