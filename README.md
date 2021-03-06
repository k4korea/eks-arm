# eks-arm

eks arm site
https://github.com/aws/containers-roadmap/tree/master/preview-programs/eks-arm-preview


다음 지침에 따라 Amazon EKS를 사용하여 Kubernetes 클러스터를 생성하고 EC2 ARM 노드에서 서비스를 시작하십시오.

참고 :이 안내서에서는 새 EKS 클러스터를 만들어야합니다. 문제를 피하기 위해 모든 단계를 완료했는지 확인하십시오.

1 단계. EKS 명령 줄 도구 인 eksctl 설치
클러스터를 만들기 위해 EKS 명령 줄 도구 인 eksctl 을 사용 합니다.

최신 버전의 Homebrew가 설치되어 있는지 확인하십시오 . Homebrew가없는 경우 다음 명령을 사용하여 설치할 수 있습니다./usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
Weaveworks Homebrew 탭을 설치하십시오. brew tap weaveworks/tap
ekstctl을 설치하십시오. brew install weaveworks/tap/eksctl
성공적으로 설치되었는지 테스트하십시오. eksctl --help
2 단계. kubectl 및 AWS IAM 인증 자 설치
위의 Homebrew 지침을 사용하여 macOS에 eksctl을 설치 한 경우 kubectl과 aws-iam-authenticator가 이미 시스템에 설치되어 있습니다. 그렇지 않으면 Amazon EKS 시작 안내서 필수 구성 요소를 참조 할 수 있습니다 .

3 단계. 작업자 노드없이 VPC, IAM 역할 및 Amazon EKS 클러스터 생성
다음 eksctl 명령을 사용하여 작업자 노드를 프로비저닝하지 않고 EKS 클러스터를 만들고 사용하려는 Kubernetes 버전을 선택하십시오.

eksctl create cluster \
--name arm-preview \
--version << Choose 1.13 or 1.14 >> \
--region us-west-2 \
--without-nodegroup
eksctl을 사용하여 EKS 클러스터를 시작하면 CloudFormation 스택이 생성됩니다. 이 스택의 시작 프로세스는 일반적으로 10-15 분이 걸립니다. EKS 콘솔 에서 진행 상황을 모니터링 할 수 있습니다 .

시작 프로세스가 완료되면 CloudFormation 스택을 검토하여 VPC ID뿐만 아니라 Control Plane 보안 그룹의 ID를 기록하려고합니다. CloudFormation 콘솔로 이동하십시오 . 라는 스택이 표시됩니다 eksctl-<cluster name>-cluster. 이 스택을 선택하고 오른쪽 패널에서에 대한 탭을 클릭하십시오 Outputs. SecurityGroup및 의 항목 값을 기록하십시오 VPC.

를 사용하여 클러스터가 실행 중인지 테스트하십시오 kubectl get svc. 다음과 같은 정보를 반환해야합니다.

NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   ww.xx.yy.zz      <none>        443/TCP   20m
EKS 클러스터에서 ARM 노드 만 지원하려면 일부 Kubernetes 구성 요소를 업데이트해야합니다. 아래 단계에 따라 CoreDNS, Kube-Proxy를 업데이트하고 AWS ARM64 CNI 플러그인을 설치하십시오.

4 단계. CoreDNS에 사용 된 이미지 ID 업데이트
업데이트 된 버전을 설치하는 데 사용중인 Kubernetes 버전에 따라 아래 명령 중 하나를 실행하십시오 CoreDNS.

쿠 버네 티스 1.13

kubectl apply -f https://raw.githubusercontent.com/aws/containers-roadmap/master/preview-programs/eks-arm-preview/dns-arm-1.13.yaml
쿠 버네 티스 1.14

kubectl apply -f https://raw.githubusercontent.com/aws/containers-roadmap/master/preview-programs/eks-arm-preview/dns-arm-1.14.yaml
5 단계. kube-proxy에 사용 된 이미지 ID 업데이트
업데이트 된 버전을 설치하는 데 사용하는 Kubernetes 버전에 따라 아래 명령을 실행하십시오 kube-proxy.

쿠 버네 티스 1.13

kubectl apply -f https://raw.githubusercontent.com/aws/containers-roadmap/master/preview-programs/eks-arm-preview/kube-proxy-arm-1.13.yaml
쿠 버네 티스 1.14

kubectl apply -f https://raw.githubusercontent.com/aws/containers-roadmap/master/preview-programs/eks-arm-preview/kube-proxy-arm-1.14.yaml
6 단계. ARM CNI 플러그인 배포
아래 명령을 실행하여 AWS ARM64 CNI 플러그인을 설치하십시오 (이 명령은 1.13 및 1.14 모두에서 작동 함).

kubectl apply -f https://raw.githubusercontent.com/aws/containers-roadmap/master/preview-programs/eks-arm-preview/aws-k8s-cni-arm64.yaml
7 단계. Amazon EKS ARM 작업자 노드 시작 및 구성
(선택 사항) M6g 인스턴스 를 사용 하려는 경우 M6g 미리보기에 액세스 할 수 있는지 확인하십시오. 액세스를 요청하려면 https://pages.awscloud.com/m6gpreview.html 을 방문 하십시오 .
https://console.aws.amazon.com/cloudformation 에서 AWS CloudFormation 콘솔을 엽니 다 . EKS 클러스터를 생성 한 AWS 리전에 있는지 확인하십시오.
스택 생성을 선택합니다 .
대한 템플릿을 선택 , 선택 아마존 S3 템플릿 URL을 지정합니다 .
다음 URL을 텍스트 영역에 붙여 넣고 다음을 선택하십시오 . https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-11-15/amazon-eks-arm-nodegroup.yaml
온 것은, 세부 사항 페이지를 지정 이에 따라 다음과 같은 매개 변수를 작성, 선택 다음을 .
스택 이름 : AWS CloudFormation 스택의 스택 이름을 선택하십시오. 예를 들어 -worker-nodes라고 부를 수 있습니다.
KubernetesVersion EKS 클러스터를 시작할 때 선택한 Kubernetes 버전을 선택하십시오. 중요이 버전은 1 단계 : Amazon EKS 클러스터 생성에서 사용한 버전과 일치해야합니다. 그렇지 않으면 작업자 노드가 클러스터에 참여할 수 없습니다.
ClusterName : Amazon EKS 클러스터를 생성 할 때 사용한 이름을 입력하십시오. 중요이 이름은 1 단계 : Amazon EKS 클러스터 생성에서 사용한 이름과 정확히 일치해야합니다. 그렇지 않으면 작업자 노드가 클러스터에 참여할 수 없습니다.
ClusterControlPlaneSecurityGroup : 보안 그룹 드롭 다운 목록이 표시됩니다. Amazon EKS 클러스터 VPC 생성 단계에서 캡처 한 AWS CloudFormation 출력에서 ​​값을 선택하십시오. (예 : eksctl- <클러스터 이름> -cluster-ControlPlaneSecurityGroup-XXXXXXXXXXXXX)
NodeGroupName : 노드 그룹의 이름을 입력하십시오. 이 이름은 나중에 작업자 노드에 대해 생성 된 Auto Scaling 노드 그룹을 식별하는 데 사용할 수 있습니다.
NodeAutoScalingGroupMinSize : 작업자 노드 Auto Scaling 그룹이 확장 할 수있는 최소 노드 수를 입력하십시오.
NodeAutoScalingGroupDesiredCapacity : 스택이 생성 될 때 확장 할 원하는 노드 수를 입력하십시오.
NodeAutoScalingGroupMaxSize : 작업자 노드 Auto Scaling 그룹이 확장 할 수있는 최대 노드 수를 입력하십시오.
NodeInstanceType : 작업자 노드에 대한 ARM 인스턴스 유형 중 하나를 선택합니다 (예 :)a1.large .
NodeVolumeSize : 노드 볼륨 크기를 입력하십시오. 기본값은 20입니다.
KeyName : RDP가 시작된 후 작업자 노드로 관리자 암호를 해독하는 데 사용할 수있는 Amazon EC2 키 페어의 이름을 입력합니다. Amazon EC2 키 페어가 아직없는 경우 AWS Management Console에서 키 페어를 생성 할 수 있습니다. 자세한 내용은 Linux 인스턴스 용 Amazon EC2 사용 설명서의 Amazon EC2 키 페어를 참조하십시오.
참고 : 여기에 키 페어를 제공하지 않으면 AWS CloudFormation 스택 생성이 실패합니다.

VpcId : Amazon EKS 클러스터 VPC 생성 단계에서 캡처 한 AWS CloudFormation 출력에서 ​​값을 선택합니다. (예 : eksctl- <클러스터 이름> -cluster / VPC)

서브넷 : Amazon EKS 클러스터 VPC 생성에서 생성 한 서브넷을 선택합니다.

NodeImageAMI113 : 1.13 AMI 이미지 ID의 Amazon EC2 Systems Manager 파라미터. 이 매개 변수를 변경하지 않아야합니다. KubernetesVersion으로 1.14를 선택한 경우이 값은 무시됩니다.

NodeImageAMI114 : 1.14 AMI 이미지 ID의 Amazon EC2 Systems Manager 파라미터. 이 매개 변수를 변경하지 않아야합니다. KubernetesVersion으로 1.13을 선택한 경우이 값은 무시됩니다.

온 옵션 페이지에서, 당신은 당신의 스택 자원에 태그를 선택할 수 있습니다. 다음을 선택하십시오 .
온 검토 페이지에서 정보를 검토, 스택 IAM 자원을 만든 다음 선택할 수 있음을 인정 만듭니다 .
단계 8. ARM64 인스턴스 역할 ARN을 기록하십시오.
ARM 작업자 노드 스택 생성이 완료되면 콘솔에서 해당 스택을 선택하고 출력 탭을 선택 하십시오.
작성된 노드 그룹 의 NodeInstanceRole 값을 기록하십시오 . 10 단계에서 Amazon EKS 작업자 노드를 구성 할 때 이것이 필요합니다.
9 단계. 작업자 노드가 클러스터에 참여할 수 있도록 AWS 인증 자 구성 맵 구성
구성 맵 다운로드 wget https://raw.githubusercontent.com/aws/containers-roadmap/master/preview-programs/eks-arm-preview/aws-auth-cm-arm64.yaml

자주 사용하는 텍스트 편집기로 파일을 엽니 다. 바꾸기 <arm64 노드의 인스턴스 역할 (안 인스턴스 프로파일)의 ARN을 (9 단계 참조)> 와 조각을 NodeInstanceRole의 10 단계 위에서 기록한 값 및 파일 저장.

중요 사항 :이 파일에서 다른 행을 수정하지 마십시오.

apiVersion: v1
kind: ConfigMap
metadata:  
  name: aws-auth  
  namespace: kube-system
data:  
  mapRoles: |  
    - rolearn: <ARN of instance role (not instance profile) of arm64 nodes (see step 10)>
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
구성을 적용하십시오. 이 명령은 완료하는 데 몇 분이 걸릴 수 있습니다.kubectl apply -f aws-auth-cm-arm64.yaml
참고 : 오류가 발생 "aws-iam-authenticator": executable file not found in PATH하면 머신의 kubectl 이 Amazon EKS에 대해 올바르게 구성되지 않은 것입니다. 자세한 정보는 aws-iam-authenticator 설치를 참조하십시오 .

다른 권한 부여 또는 자원 유형 오류가 수신되면 권한이 없거나 액세스가 거부 됨 (kubectl)을 참조하십시오 .

노드의 상태를보고 준비 상태가 될 때까지 기다리십시오 .kubectl get nodes --watch
10 단계. 앱 시작
메트릭 서버를 시작하여 포드를 예약 할 수 있는지 테스트하십시오.

kubectl apply -f https://raw.githubusercontent.com/aws/containers-roadmap/master/preview-programs/eks-arm-preview/cni-metrics-helper-arm64.yaml
산출:
clusterrole.rbac.authorization.k8s.io/cni-metrics-helper created
serviceaccount/cni-metrics-helper created
clusterrolebinding.rbac.authorization.k8s.io/cni-metrics-helper created
deployment.extensions/cni-metrics-helper created
예약 된 포드를 확인하십시오.

kubectl -n kube-system get pods -o wide
다음 단계
새 EKS 클러스터에서 자체 컨테이너를 실행하십시오.

GitHub 문제 에 대한 의견이나 질문을 남겨주십시오 .

이것은 진화하는 프로젝트입니다. 새로운 기능을 출시함에 따라이 저장소와 로드맵 문제가 업데이트 될 것입니다