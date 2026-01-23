# Domain 5: Data Protection (데이터 보호)


이 도메인의 핵심은 데이터의 암호화(Encryption), 키 관리(Key Management), 그리고 **중요 데이터 식별(Data Discovery)** 입니다.

---

### **1. AWS KMS (Key Management Service)**

KMS는 하드웨어 및 소프트웨어 기반의 키 관리 솔루션으로, 단순히 키를 저장하는 것을 넘어 권한 제어와 로테이션을 담당합니다.

- **암호화 유형**:
    - **Symmetric (대칭키)**: **AES-256**을 사용하여 데이터를 암호화합니다.
    - **Asymmetric (비대칭키)**: 암호화/복호화 외에도 **디지털 서명(Signing) 및 검증(Verify)** 에 유용합니다.
- **주요 기술적 특징**:
    - **Envelope Encryption (봉투 암호화)**: 4KB 이상의 데이터를 암호화할 때 사용합니다. 데이터 키(DK)로 데이터를 암호화하고, 그 데이터 키를 KMS의 루트 키(CMK)로 암호화하여 데이터와 함께 보관합니다.
    - **Grants**: 특정 사용자나 서비스에게 **임시 권한(Temporary Grant)** 을 부여할 때 사용합니다.
- **Multi-Region Keys**:
    - 동일한 키 ID와 키 자재를 여러 리전에서 공유합니다. (A리전 암호화 → B리전 복호화 가능)
    - 하지만 글로벌 관리가 아니며 리전 간 독립성이 낮아지므로 **일반적인 권한 사항(Best Practice)은 아닙니다.**
    - 삭제 시 **Replica 키를 먼저 삭제**한 후 Primary 키를 삭제해야 합니다.
- **Key Rotation (키 로테이션)**:
    - **AWS Managed Key**: **1년마다 자동 자동교체**됩니다. 이때 **Key ID는 유지**되지만 내부의 **Binding Key**만 변경되어 기존 데이터를 그대로 복호화할 수 있습니다.
    - **Customer Managed Key (수동)**: **Key Alias**를 활용합니다. 새 키를 생성한 후 기존 Alias가 새 키를 가리키도록 업데이트하며, 기존 데이터 복호화를 위해 예전 키도 보관해야 합니다.
- **EFS 암호화**: 기존의 암호화되지 않은 EFS는 직접 암호화로 전환할 수 없습니다. **새로운 암호화된 EFS**를 생성한 후, **AWS DataSync**를 사용해 데이터를 마이그레이션해야 합니다.

### **2. S3 데이터 보호 및 관리**

S3는 저장 방식과 전송 방식 모두에서 강력한 암호화 옵션을 제공합니다.

- **Server-Side Encryption (SSE) 상세**:
    - **SSE-S3**: 키를 AWS가 소유하고 관리합니다. **AES-256**이 기본(Default)입니다. (손쓸 수 없는 기본 암호화)
    - **SSE-KMS**: KMS를 사용하여 키를 관리합니다. **CloudTrail**에 키 사용 기록이 남으며, **KMS Quota(할당량)** 제한이 있어 대규모 요청 시 성능 한계가 존재할 수 있습니다. (통제 가능한 엔터프라이즈 암호화)
    - **SSE-C**: 고객이 제공하는 키를 사용합니다. 보안을 위해 반드시 **HTTPS** 연결을 사용해야 합니다.
- **기타 암호화 방식**:
    - **Client-side Encryption**: S3로 보내기 전 클라이언트 측에서 직접 암호화하여 전송합니다.
    - **In-Transit (전송 중 암호화)**: **SSL/TLS**를 사용하며, aws:SecureTransport 정책으로 설정가능
        
- **S3 관리 도구**:
    - **S3 Batch Operations**: 대량의 객체에 대해 암호화 적용이나 태그 변경 등 **Bulk 작업**을 수행합니다.
    - **S3 Lifecycle**: 일정 기간이 지나면 데이터를 저렴한 스토리지 클래스(Tier)로 이동시키거나 삭제하여 비용을 절감합니다.

### **3. CloudHSM**

- **역할**: FIPS 140-2 Level 3 인증을 받은 **암호화 전용 하드웨어만 제공**합니다.
- **책임 모델**: AWS는 하드웨어 가용성만 책임지며, 그 안에서의 **키 생성, 관리, 애플리케이션 연동 등 나머지는 모두 사용자가 직접 관리**하고 책임져야 합니다.

### **4. 로드 밸런서(ELB) & ACM (HTTPS/TLS)**

네트워크 전송 중 보안(Encryption in Transit)을 위한 핵심 구성 요소입니다.

- **Load Balancer 유형별 보안 특징**:
    - **Application (ALB)**: 7계층(HTTP/HTTPS), gRPC 지원, 경로 기반 라우팅, 고유 URL 제공.
    - **Network (NLB)**: 4계층(TCP/UDP), 고성능, **Static IP(고정 IP)** 제공.
    - **Gateway (GWLB)**: 3계층, GENEVE 패킷 처리, 트래픽을 방화벽이나 침입 탐지 시스템(IDS/IPS)으로 라우팅.
- **SSL/TLS 및 SNI**:
    - 현재는 **TLS**가 최신 표준이지만 관습적으로 SSL이라 부르기도 합니다.
    - 로드 밸런서는 **X.509(SSL/TLS 서버 인증서)** 를 사용하여 암호화를 수행합니다.
    - **SNI (Server Name Indication)**: 하나의 로드 밸런서에 여러 개의 다른 SSL 인증서를 로드하여 여러 도메인을 처리하는 기술입니다. (**ALB, NLB는 지원**, 이전 세대인 CLB는 지원 안 함)
- **ACM (AWS Certificate Manager)**:
    - 인증서를 프로비저닝, 관리, 갱신합니다. **Private CA** 기능도 제공합니다.
    - **Regional 서비스**: 특정 리전에서 발급받은 인증서는 해당 리전의 서비스(ALB 등)에서만 사용 가능합니다. (Global 서비스 아님)

### **5. AWS Backup & Secrets Manager**

- **AWS Backup**:
    - 완전 관리형 자동 백업 서비스로, **Cross-Region 백업**을 지원합니다.
    - **Backup Plans**를 통해 정책(주기, 보존 기간 등)을 작성하며 백업본은 S3에 저장됩니다.
    - **Vault Lock**: WORM(한번 기록하면 변경 불가) 속성을 부여하여 백업 데이터 삭제나 수정을 방지하는 강력한 보호 조치입니다.
- **Secrets Manager**:
    - 데이터베이스 자격 증명 등을 안전하게 저장하며, 정의된 **X일마다 자동으로 로테이션**하는 기능을 제공합니다.

### **6. AWS Nitro Enclaves**

- **용도**: EC2 인스턴스 내에서 생성할 수 있는 완벽히 **격리된(Isolated) 실행 환경**입니다.
- **특징**: PII(개인식별정보)나 아주 민감한 금융 데이터 등을 처리할 때 사용하며, 인스턴스의 사용자나 루트 권한조차 Enclave 내부 데이터에 접근할 수 없습니다.

### **💡 Exam Point**

1. **KMS Key Policy 문제**: "IAM 권한만 줬는데 왜 외부에선 키 사용이 안 될까?" → Key Policy 확인이 정답인 경우가 많습니다.
2. **데이터 가용성 vs 보호**: 삭제 방지가 목적이라면 S3 Versioning, Multi-Factor Authentication (MFA) Delete, Object Lock을 먼저 떠올리세요.
3. **Cross-Region 복제**: 암호화된 S3 버킷을 다른 리전으로 복제할 때, KMS 키는 리전 종속적이므로 **대상 리전의 KMS 키**로 재암호화가 필요함을 유의하세요.
