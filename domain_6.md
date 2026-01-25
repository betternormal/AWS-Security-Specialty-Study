# Domain 6: Management and Security Governance
멀티 계정 환경에서의 중앙 집중식 관리 및 거버넌스 자동화가 핵심입니다

### **1. AWS Organizations & 계정 관리**

- **멀티 계정 전략:** 글로벌 서비스로서 여러 AWS 계정을 중앙 집중적으로 관리합니다.
- **통합 빌링 (Consolidated Billing):** 여러 계정의 결제를 하나로 묶어 관리하며, **통합 사용량에 따른 대량 구매 할인(Volume Discounts)** 혜택을 받을 수 있습니다.
- **Organizational Units (OU):** 개발, 운영, 보안 등 목적에 따라 계정을 그룹화하여 계층적 거버넌스를 구현합니다.
- **SCP (Service Control Policies):** 계정의 최대 권한 경계를 설정하며, **루트 사용자에게도 적용**됩니다.

### **2. AWS Control Tower (거버넌스 자동화)**

- **자동 설정:** 베스트 프랙티스에 기반한 가드레일을 멀티 계정 환경에 자동으로 적용하여 보안 거버넌스를 수립합니다.
- **랜딩 존 (Landing Zone):** AWS 보안 기준에 부합하는 환경을 클릭 몇 번으로 구성해줍니다.

### **3. AWS Config (변경 감지 및 준수 평가)**

- **실시간 모니터링:** 리소스 설정이 컴플라이언스를 준수하는지 추적합니다.
- **탐지형 가드레일:** 설정 변경이 규정에 어긋나면(Non-compliant) 알람을 보내지만, **변경 자체를 직접 차단하지는 못합니다.**
- **자동 수정 (Remediation):**
    - **SSM Automation:** Config 규칙 위반 시 시스템 매니저를 트리거하여 자동으로 보정 작업을 실행합니다.
    - **Event-Driven:** EventBridge + Lambda/SNS를 통해 커스텀 보정 로직을 실행하거나 관리자에게 알림을 보냅니다.
- **Aggregator:** 여러 계정과 리전의 설정 데이터를 한곳에 모아 대시보드 형태로 제공합니다.

### **4. CloudFormation (IaC 보안 및 거버넌스)**

- **IaC (Infrastructure as Code):** 템플릿을 통해 일관된 보안 인프라를 배포합니다.
- **Stack Policy:** 스택 업데이트 중 **특정 리소스가 실수로 수정되거나 삭제되는 것을 방지**하도록 정의합니다.
- **Dynamic Reference:** SSM Parameter Store나 Secrets Manager의 값을 템플릿에서 보안상 안전하게 참조합니다.
- **Drift Detection:** 템플릿으로 생성된 인프라가 수동 변경으로 인해 원래 설정과 달라졌는지 감지합니다.
- **CloudFormation Guard:** 템플릿이 조직의 보안 정책에 부합하는지 배포 전 검증하는 **오픈소스 CLI 툴**입니다.

### **5. 감사 및 규정 준수 (Audit & Compliance)**

- **AWS Audit Manager:** GDPR, HIPAA, PCI-DSS 등 규제 표준에 맞게 증거를 자동으로 수집하여 리스크와 규정 준수 여부를 평가하고 보고서를 생성합니다.
- **AWS Artifact:** AWS의 각종 보안 준수 인증 보고서를 제공합니다.

### **6. 리소스 공유 및 카탈로그 관리**

- **AWS Resource Access Manager (RAM):** 서브넷, 라이선스 등을 다른 계정과 공유하여 리소스 중복을 줄입니다. (특히 **Private IP 대역** 활용 등 효율적 네트워크 구성에 중요)
- **AWS Service Catalog:** 관리자가 승인한 CloudFormation 템플릿을 포트폴리오로 구성하여, 사용자(엔지니어)들이 규제에 맞는 리소스만 생성할 수 있게 합니다.

### **7. 비용 및 아키텍처 거버넌스**

- **Cost Explorer:** 과거 비용 확인 및 미래 지불 비용을 예측합니다.
- **Cost Anomaly Detection:** 머신러닝을 사용하여 **비정상적인 비용 지출을 감지**하고 보고/알림을 제공합니다.
- **Trusted Advisor:** 보안, 비용 최적화, 성능 등 5가지 카테고리에 대해 계정을 분석하고 개선을 추천합니다.
- **Well-Architected Tool:** 6가지 원칙(Pillars)에 따라 아키텍처를 리뷰하고 위험 요소를 식별합니다.

---

**💡 시험 핵심 키워드 매칭:**

- "변경을 막고 싶다" → **SCP** (Preventive)
- "변경을 감지하고 고치고 싶다" → **Config + SSM/Lambda** (Detective + Remediation)
- "여러 계정의 로그/설정을 합치고 싶다" → **Aggregator / Delegated Admin**
- "CFN 배포 시 특정 리소스 보호" → **Stack Policy**
- "비정상적인 지출 감지" → **Cost Anomaly Detection**
- "특정 리전 이외의 서비스 사용을 아예 금지하고 싶다" → AWS Organizations의 **SCP**에서 , NotAction, NotIpAddress 등을 활용한 Deny 정책 설정.
- "루트 사용자의 액세스 차단 또는 특정 작업 제한" → **SCP**를 통해 Root 계정의 권한을 제한하는 방법.
- "S3 버킷이 생성되자마자 특정 설정을 강제하고 싶다" → **CloudFormation StackSets** 또는 **Control Tower**의 Guardrails 활용.