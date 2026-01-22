# Domain 4: Identity and Access Management

단순히 IAM 사용법을 넘어서, **"누가(Authentication)"**, **"어떤 권한으로(Authorization)"**, **"어떻게 접근하는가(Access Control)"** 를 설계하고 트러블슈팅하는 능력을 평가합니다.

---

### **1. 인증 (Authentication) - "누구인가?"**

사용자나 시스템의 신원을 확인하는 과정입니다.

- **AWS IAM vs AWS IAM Identity Center (구 SSO)**
    - **IAM**: 단일 계정 내의 사용자/역할 관리에 적합. 장기 자격 증명(User Access Key) 사용은 지양하고**임시 자격 증명(Role)**사용을 권장합니다. IAM의 role은 specific한 topic에 대해 일시적인 권한을 줄때 사용된다
    - **IAM Identity Center**: 싱글사인온을 가능하게 해주는 서비스, 멀티 계정 환경의 표준. 온프레미스 AD나 외부 IdP(Okta 등)와 연동하여**SAML 2.0**기반의 단일 로그인(SSO)을 구현합니다.
        - Microsoft Active Directory: 윈도우 서버에서 DB 유저, 프린터, 파일, 시큐리티 그룹 등을 관리하는곳
        - AWS Directory Service: AWS가 Microsoft Active Directory를 관리한다
- **Federation (연합 인증)**
    - **SAML 2.0**: 기업 내부 ID 저장소(AD 등)를 사용하여 AWS 콘솔/CLI에 접근할 때 사용.
    - **Web Identity Federation (OIDC)**: 모바일 앱 등에서 Google, Facebook, Amazon 로그인을 통해 AWS 리소스에 접근할 때 사용 (주로**Amazon Cognito**와 함께 사용).
- **Amazon Cognito**
    - **User Pools**: "회원가입/로그인" 기능 제공 (SaaS 앱의 사용자 디렉터리, id/pw방식 ~ SNS login까지). JWT 토큰 발행.
    - **Identity Pools**: User Pool이나 소셜 로그인으로 인증된 사용자에게 **AWS 임시 자격 증명(IAM Role)**을 부여하여 AWS 리소스(S3, DynamoDB 등)에 직접 접근하게 함.

### **2. 권한 부여 (Authorization) - "무엇을 할 수 있는가?"**

신원이 확인된 대상에게 리소스 접근 권한을 제어합니다.

- **IAM Policy의 구조와 로직 (가장 중요)**
    - `Principal`, `Action`, `Resource`, `Condition` 요소를 완벽히 이해해야 합니다.
    - **명시적 거부(Explicit Deny) > 명시적 허용(Explicit Allow) > 묵시적 거부(Implicit Deny)** 순서로 우선순위가 적용됩니다.
- **정책의 종류 (Policy Types)**
    - **Identity-based Policy**: IAM User/Group/Role에 연결 (이 사용자가 무엇을 할 수 있나?)
    - **Resource-based Policy**: S3 Bucket, SNS Topic, KMS Key 등에 직접 연결 (이 리소스에 누가 접근할 수 있나?).  **Cross-account 접근** 시 필수적입니다.
        - Resource based policy는 영구적 설정이고, 리소스에대한 정책이다.
    - **Permissions Boundary**: IAM User/Role이 가질 수 있는 **최대 권한**을 제한. (관리자 권한을 위임할 때, 위임받은 관리자가 자신보다 더 높은 권한을 생성하지 못하도록 막음).
- **Service Control Policies (SCPs)**
    - **AWS Organizations**에서 사용. 멤버 계정 전체의 **최대 권한**을 제어합니다.
    - SCP는 권한을 부여하는 것이 아니라, **"제한(Deny)"** 하거나 **"허용 가능한 범위를 지정(Allow)"** 하는 가드레일 역할을 합니다. (예: 특정 리전 사용 금지, Root 유저 사용 금지).
    - **중요**: SCP에서 Allow하고 IAM에서도 Allow해야 접근 가능합니다 (교집합).

### **3. 액세스 제어 전략과 보안 원칙**

- **최소 권한의 원칙 (Least Privilege)**
    - 항상 업무 수행에 필요한 최소한의 권한만 부여해야 합니다. Access Advisor나 Last Accessed Data를 통해 불필요한 권한을 식별하고 제거하는 것이 시험에 자주 나옵니다.
        
- **RBAC vs ABAC**
    - **RBAC (Role-Based)**: 역할(Role)에 따라 권한 부여. 역할이 많아지면 관리가 복잡해짐. (Time granted permission)
    - **ABAC (Attribute-Based)**: 태그(Tag)를 기반으로 권한 제어. 새로운 리소스가 생겨도 태그만 맞으면 자동으로 권한이 부여되므로 확장성이 좋습니다.
        

### **4. 트러블슈팅 및 도구**

- **AWS STS (Security Token Service)**
    - 임시 access key to resources(한시간 유효)
    - `AssumeRole`: 임시 보안 자격 증명을 받아오는 핵심 API. Cross-account 접근이나 EC2 인스턴스 프로파일 등에서 내부적으로 사용됩니다.
    - `GetSessionToken`: MFA 인증이 필요한 API 호출 시 MFA 토큰을 포함하여 세션을 획득할 때 사용.
- **IAM Access Analyzer**
    - 외부(다른 계정, 인터넷 등)에서 내 리소스에 접근할 수 있는 정책이 있는지 분석하여 보안 위험을 알려줍니다.
- **정책 시뮬레이터 (IAM Policy Simulator)**
    - 복잡한 정책들이 겹쳐 있을 때, 특정 작업이 허용되는지 거부되는지 테스트하는 도구입니다.

### **💡 시험 팁 (Scenario 위주)**

1. **Cross-Account Access**: 항상 **IAM Role**을 사용해야 합니다. (Access Key 공유 절대 금지)
2. **S3 버킷 퍼블릭 접근 차단**: 잘못된 정책으로 데이터가 노출되는 것을 막기 위해 S3 Block Public Access 설정과 IAM 정책을 어떻게 조합하는지 묻습니다.
3. **KMS Key Policy**: IAM 정책에서 허용했더라도, **KMS Key Policy**에서 허용하지 않으면 암호화/복호화가 불가능합니다. (이중 잠금 장치 개념)
4. **NotAction & Deny 조합**: "이것만 빼고 다 거부" 혹은 "특정 조건에서만 허용" 같은 까다로운 로직을 해석할 수 있어야 합니다.
