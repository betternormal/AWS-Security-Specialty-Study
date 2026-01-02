# Domain 1. Threat Detection and Incident Response(위협 탐지 및 사고 대응)

  이 도메인은 AWS 환경에서 발생하는 보안 위협을 얼마나 잘 탐지(Detect)하고, 조사(Investigate)하며, 신속하게 대응 및 복구(Respond & Remediate)하여 피해를
  최소화하는지에 중점을 둡니다.

  1. 탐지 (Detection): 위협 식별 및 경보

  비정상 활동이나 잠재적 위협을 식별하는 단계입니다. "무엇인가 잘못되었다"는 것을 인지하는 것이 목표입니다.

  핵심 AWS 서비스 및 개념:

   * Amazon GuardDuty:
       * 핵심 기능: 지능형 위협 탐지 서비스. VPC Flow Logs, DNS Logs, CloudTrail 로그를 머신러닝과 통합 위협 인텔리전스를 통해 분석하여 위협을 탐지합니다.
       * 주요 특징:
           * 크립토커런시(암호화폐) 채굴 탐지에 최적화되어 있습니다.
           * 탐지된 위협(Finding)은 Amazon EventBridge로 전송하여 알림(예: SNS)을 보내거나 Lambda를 통한 자동화된 대응을 트리거할 수 있습니다.

   * AWS Security Hub:
       * 핵심 기능: 여러 AWS 보안 서비스(GuardDuty, Inspector, Macie 등)와 써드파티(3rd Party) 솔루션의 보안 결과를 중앙에서 집계하는 대시보드입니다.
       * 주요 특징:
           * 크로스 리전(Cross-Region) 집계를 지원하여 여러 리전의 보안 상태를 한 곳에서 볼 수 있습니다.
           * CIS 벤치마크, PCI-DSS 같은 보안 표준 준수 여부를 자동으로 검사합니다.
           * 결과는 90일 동안 보관되며, 90일이 지나면 삭제됩니다.

   * Amazon Inspector:
       * 핵심 기능: EC2 인스턴스, 컨테이너 이미지(ECR)의 소프트웨어 취약점(CVE) 및 네트워크 노출을 자동으로 평가합니다.

   * AWS Config:
       * 핵심 기능: AWS 리소스 구성 변경을 기록하고, 'Config Rule'을 통해 정책 위반 여부를 지속적으로 평가합니다. (예: "S3 버킷이 공개적으로 열려있는가?")

   * AWS CloudTrail & Amazon CloudWatch:
       * 핵심 기능: CloudTrail은 계정 내 모든 API 활동("누가, 무엇을 했는가")을 기록하고, CloudWatch는 이 로그를 기반으로 특정 이벤트 발생 시 경보를
         울리거나 대응 조치를 트리거합니다.

  ---

  2. 조사 (Investigation): 위협 분석 및 근본 원인 파악

  경보가 발생하면, 실제 위협인지, 영향 범위는 어디까지인지, 어떻게 발생했는지 분석하는 단계입니다.

  핵심 AWS 서비스 및 개념:

   * Amazon Detective:
       * 핵심 기능: 보안 결과(Finding)의 근본 원인(Root Cause)을 쉽고 빠르게 조사하도록 돕습니다.
       * 주요 특징:
           * 로그 데이터를 자동으로 수집하고, 머신러닝과 그래프 이론(Graph Theory)을 사용하여 리소스, 계정, IP 주소 간의 관계를 시각화하여 보여줍니다.
             이를 통해 수동적인 로그 분석 시간을 크게 줄일 수 있습니다.

   * VPC Flow Logs & Amazon Athena:
       * 핵심 기능: VPC 네트워크 트래픽 로그를 S3에 저장하고, Athena SQL 쿼리를 통해 특정 IP와의 통신 기록 등 상세한 트래픽을 분석합니다.

  ---

  3. 대응 및 복구 (Response & Remediation): 위협 완화 및 시스템 복원

  위협을 제거하고, 피해를 복구하며, 재발을 방지하는 조치를 취하는 단계입니다. 자동화된 대응이 핵심입니다.

  핵심 AWS 서비스 및 개념:

   * 자동화된 대응 (EventBridge + Lambda):
       * 동작 방식: EventBridge가 보안 이벤트(예: GuardDuty 결과)를 감지하면 Lambda 함수를 트리거하여 네트워크 격리, 스냅샷 생성, IAM 역할 변경 등의 대응
         조치를 자동으로 수행합니다.

   * EC2 인스턴스 접근 문제 해결 시나리오:
       * 키페어를 잃어버렸을 경우 대응 방법:
           1. (신규 키 등록) EC2 User Data: 인스턴스를 중지한 후, User Data 스크립트에 새 키페어 정보를 추가하여 시작 시 적용되게 합니다.
           2. (자동화) AWS Systems Manager (SSM): SSM Agent가 설치되어 있다면, AWSSupport-ResetAccess 문서를 사용하거나 SSM CLI를 통해 원격으로 키페어
              재설정 과정을 자동화할 수 있습니다.
           3. (임시 웹 콘솔) EC2 Instance Connect: 60초짜리 임시 키페어를 인스턴스에 푸시하여 웹 기반 SSH로 안전하게 접속한 후, 키페어 재설정 등 필요한
              작업을 수행합니다.
           4. (직접 연결) EC2 Serial Console: 인스턴스에 직접 시리얼 연결을 하여 부팅 문제, 네트워크 설정 오류, 루트 계정 암호 재설정 등 OS 레벨의 이슈를
              해결합니다.
           5. (볼륨 교체) EBS Volume Swap: 기존 인스턴스의 루트 EBS 볼륨을 분리하여 다른 정상 인스턴스에 붙인 후, 키 정보를 수정하고 다시 원래 인스턴스에
              붙여서 부팅합니다.

       * 기타 복구 도구:
           * EC2 Rescue Tool: 인스턴스 연결 문제, OS 부팅 이슈, OS 로그 분석 등 일반적인 문제를 진단하고 해결(Troubleshoot)하는 데 사용되는 도구입니다.

  ---

  4. IAM 보안 및 접근 분석 (IAM Security & Access Analysis)

  IAM은 보안의 근간이며, 접근 제어와 관련된 도구들을 잘 이해해야 합니다.

   * IAM Credential Report: 계정 레벨(Account Level)의 보고서로, 계정의 모든 IAM 사용자에 대한 정보(암호 최종 사용일, 액세스 키 최종 사용일 등)를 CSV
     파일로 제공합니다.
   * IAM Access Advisor: 사용자 레벨(User Level)에서 특정 IAM 사용자 또는 역할이 마지막으로 각 서비스에 접근한 정보를 보여줍니다. 최소 권한 원칙을
     적용하는 데 사용됩니다.
   * IAM Access Analyzer:
       * 주요 기능: S3 버킷, IAM 역할 등 어떤 리소스가 외부 엔티티(다른 AWS 계정, IAM 사용자 등)와 공유되어 있는지 식별합니다.
       * 정책 생성: CloudTrail 로그를 분석하여 사용자의 실제 활동에 기반한 세분화된 IAM 정책을 자동으로 생성해주는 기능도 있습니다.

  ---

  5. AWS 정책 및 규정 (AWS Policies & Regulations)

  AWS 사용 시 반드시 준수해야 할 정책과 절차입니다.

   * 모의 해킹 (Penetration Testing):
       * AWS의 허가 없이 수행할 수 있는 서비스는 EC2, NAT Gateway, ELB, RDS, Aurora, CloudFront, API Gateway, Lambda 등 7~8개로 제한됩니다. 이외 서비스에
         대한 테스트는 AWS에 사전 허가를 받아야 합니다.
   * DDoS 공격: AWS Shield는 DDoS 방어 서비스이며, 심화된 공격에 대응하기 위해 AWS DDoS 대응 파트너사와 협력할 수 있습니다.
   * AWS AUP (Acceptable Use Policy): AWS 리소스를 사용하여 불법적이거나 악의적인 활동을 하는 것을 금지하는 정책입니다. 위반 시 계정이 정지되거나 리소스가
     삭제될 수 있습니다.