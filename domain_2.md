# Domain 2. Security Logging and Monitoring

이 도메인은 단순히 "로그를 남긴다"는 것을 넘어, 로그를 안전하게 수집, 보호, 분석하고, 위협에 대해 자동화된 대응(Remediation)을 할 수 있는지를 묻습니다.

### **1. Monitoring & Metrics (감시 및 지표)**

### **Amazon CloudWatch**

- **기본 역할**: AWS 리소스 및 애플리케이션의 모니터링 서비스.
- **Custom Metrics & Agent**:
    - 기본적으로 CPU, Disk I/O, Network 등은 하이퍼바이저 레벨에서 수집됩니다.
    - **Memory Usage, Process, Disk Swap** 등 OS 내부 정보는 **CloudWatch Agent**를 설치해야만 수집할 수 있습니다.
- **Log Streaming Architecture**:
    - 로그 데이터를 실시간 처리 및 장기 보관하는 표준 파이프라인:
    - CloudWatch Logs ->  ->  ->  ->  (최종 저장)
        -> Subscription Filter
        -> Kinesis Data Streams
        -> Kinesis Data Firehose
        -> S3
        
- **EventBridge (구 CloudWatch Events)**:
    - 상태 변경(Event)을 감지하여 자동화된 작업(Target)을 트리거합니다.
    - 예: "EC2 인스턴스 상태가 변경되면 Lambda 실행", "GuardDuty 위협 발견 시 SNS 알림".

---

### **2. Auditing & Compliance (감사 및 규정 준수)**

### **AWS CloudTrail**

- **역할**: "누가, 언제, 어디서, 무엇을 했는가"에 대한 API 호출 기록.
- **저장 및 보관**:
    - 기본적으로 **90일간**의 관리 이벤트는 콘솔에서 무료로 조회 가능.
    - **장기 보관(Audit)**: 그 이후 데이터는 반드시 **S3**로 내보내야 합니다. 아주 오래된 로그는 S3 Lifecycle 정책을 통해 **Glacier**로 이동하여 비용 효율적으로 저장합니다.
- **CloudTrail Insights**:
    - 정상적인 패턴을 벗어난 비정상적인 활동(Write API 급증 등)을 머신러닝으로 탐지하여 알림을 줍니다.
- **무결성 및 보안**:
    - 로그 파일 무결성 검증(SHA-256 해시).
    - S3 버킷에 **KMS 암호화, IAM 정책 제한, MFA Delete** 설정을 권장합니다.

### **Amazon Macie (데이터 보안)**

- **역할**: S3 버킷 내의 데이터를 머신러닝으로 스캔하여 **PII(개인 식별 정보)**와 같은 민감한 데이터를 발견(Discover)하고 분류합니다.
- **자동 대응**: 민감 데이터 발견 시 EventBridge로 이벤트를 전송하여 관리자에게 알림(Notify)을 보낼 수 있습니다.

### **AWS Config**

- 리소스의 설정 변경 기록 및 규정 준수 여부(Compliance)를 평가합니다.

---

### **3. Network Security Monitoring (네트워크 보안 모니터링)**

### **VPC Flow Logs**

- **역할**: VPC 인터페이스를 오가는 IP 트래픽 정보 수집.
- **활용**:
    - **SG(Security Group) 및 NACL 설정 문제 디버깅**: 트래픽이 ACCEPT 되었는지 REJECT 되었는지 확인 가능.
        
    - *주의: 트래픽의 내용은 볼 수 없으며, DHCP, DNS(VPC DNS), Windows 라이선스 인증 등 일부 트래픽은 로깅되지 않음.*

### **VPC Traffic Mirroring**

- **역할**: 실제 패킷의 **내용(Payload)**을 검사해야 할 때 사용 (Deep Packet Inspection).
- **동작**: 트래픽을 복제하여 별도의 보안 어플라이언스(IDS/IPS) 인스턴스로 전송.

### **Route 53 Logs**

- **Standard Logs**: CloudWatch로 전송됨.
- **Resolver Query Logging**: VPC 내에서 발생하는 DNS 쿼리 로그. S3, CloudWatch Logs, Kinesis Data Firehose로 전달하여 분석 가능.

---

### **4. Vulnerability Management (취약점 관리)**

### **Amazon Inspector**

- **역할**: 자동화된 보안 평가 서비스 (Automated Security Assessment).
- **대상**: **EC2 인스턴스, ECR(컨테이너 이미지), Lambda 함수**.
- **기능**:
    - 네트워크 노출(Network Reachability) 및 소프트웨어 취약점(CVE) 스캔.
    - 스캔 완료 후 위험도에 따른 **점수(Score)**가 포함된 리포트 생성. SSM Agent가 필요할 수 있음.

---

### **5. Operations & Automation (운영 및 자동화 - Systems Manager)**

- *AWS Systems Manager (SSM)**는 이 도메인의 숨겨진 핵심입니다. 에이전트(**SSM Agent**)가 설치된 EC2를 관리합니다.
- **Automation**:
    - **Runbook**: 미리 정의된 절차(문서)를 통해 반복적인 작업(예: 인스턴스 재시작, AMI 생성)을 자동 실행합니다.
- **Run Command**:
    - SSH 접속 없이 원격으로 스크립트 실행 가능.
- **Session Manager**:
    - **권장 사항**: Bastion Host나 SSH 키 관리 없이 브라우저/CLI를 통해 안전하게 쉘 접속.
    - **로깅**: 모든 명령어 입력 및 출력 기록을 S3나 CloudWatch Logs로 전송하여 감사(Audit) 가능.
- **Parameter Store**:
    - DB 접속 정보, 암호 등 **Configuration과 Secrets**를 계층형으로 저장.
    - **TTL(Time-to-Live)** 설정으로 만료 정책 가능. KMS 암호화 지원.
- **Patch Manager & Maintenance Window**:
    - **Maintenance Window**: 유지보수(패치 등)를 수행할 특정 시간(Schedule)을 정의.
    - 지정된 시간에 자동으로 보안 패치 적용.
- **Inventory**:
    - EC2 내부에 설치된 소프트웨어 목록 수집.
- **Tags & Resource Groups**:
    - **Resource Grouping**: Tag(Key-Value)를 기반으로 리소스를 논리적 그룹으로 묶음.
    - SSM 작업은 이 **그룹 단위**로 수행할 수 있어 대규모 관리에 효율적. (예: "Web-Server" 태그가 있는 모든 인스턴스에 패치 적용)

---

### **6. Analysis & Search (분석 및 검색)**

### **Amazon Athena**

- **역할**: S3에 저장된 로그 파일(CloudTrail, VPC Flow Logs, ALB Logs 등)을 **표준 SQL**로 즉시 쿼리 및 분석.
- **시각화**: 분석 결과는 **Amazon QuickSight**와 연동하여 시각적 대시보드 및 리포트 생성 가능.

### **Amazon OpenSearch Service (구 ElasticSearch)**

- **역할**: 대용량 로그의 실시간 검색, 분석, 시각화(Kibana).
- **특징**:
    - 데이터베이스(DynamoDB 등)의 데이터에 대한 전문 검색(Full-text search), 부분 일치(Partial Match) 검색 기능 제공.
    - SQL 쿼리를 지원하지 않는 NoSQL 데이터에 대해 SQL 플러그인을 사용하여 SQL 질의 가능.

---

### **요약: 보안 로깅/모니터링 흐름도**

1. **수집 (Ingest/Capture)**: CloudTrail, VPC Flow Logs, GuardDuty, Inspector, SSM Agent, CloudWatch Agent.
2. **저장 (Store)**: CloudWatch Logs, S3 (장기 보관, 암호화, WORM), Parameter Store.
3. **처리 및 필터링 (Process)**: EventBridge (이벤트 감지), CloudWatch Metric Filters.
4. **분석 (Analyze)**: Athena (S3 SQL 쿼리), OpenSearch (검색), QuickSight (시각화), CloudTrail Insights.
5. **대응 (Remediate)**: Lambda (자동화 로직), SSM Automation (Runbook 실행), SNS (알림).