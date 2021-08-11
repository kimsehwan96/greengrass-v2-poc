# 그린그래스 버전 2 코어 소프트웨어 설치 방법 (필요시 배포까지)

## 사전 조건

1. 자바 런타임이 설치되어있어야 한다. 
   (greengrass v2의 경우 코어 소프트웨어가 자바 베이스)
   ref : https://github.com/aws-greengrass/aws-greengrass-nucleus
- `sudo apt-get install -y default-jdk`

## AWS IoT Greengrass 코어 소프트웨어 설치


- Greengrass nucleus 컴포넌트 (코어 컴포넌트라고 생각하면 된다.)가 설치되어야 한다.

- 디바이스를 AWS IoT thing에 등록하고, 인증서 파일을 다운로드 받는다. 

- Devcie 의 Iot Thing을 그룹에 추가한다. 그룹은 thing의 모임이다.

- IAM 롤을 생성한다. (AWS와 상호작용 하기 위한 적절한 권한)

- AWS IoT Greengrass 커맨드 라인 인터페이스를 설치한다. (greengrass-cli)

### 쉬운 설치 및 배포 방법

1. 타겟 장비(원격 접속, 혹은 로컬)에서 홈 디렉터리로 간다.

` cd ~`

2. Greengrass Core 소프트웨어 다운로드를 위해 아래 명령어 실행 (curl 필요)

`curl -s https://d2s8p88vqu9w66.cloudfront.net/releases/greengrass-nucleus-latest.zip > greengrass-nucleus-latest.zip`

cf : greengrass nucleus 소프트웨어를 다운로드 받는것임.

3. 아래 명령어를 실행해 압축 해제하고, GreengrassCore 라는 디렉터리 명으로 바꾼다.

`unzip greengrass-nucleus-latest.zip -d GreengrassCore && rm greengrass-nucleus-latest.zip`

4. 타겟 장비에 AWS Credential을 주입한다. (실제 프로덕션 환경에서는 적절한 권한을 갖는 IAM 롤 및 인증서를 디바이스에 줄 것)

```shell
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

5. 아래 명령어를 수행한다.

```shell
sudo -E java -Droot="/greengrass/v2" -Dlog.store=FILE \
  -jar ./GreengrassCore/lib/Greengrass.jar \
  --aws-region region \
  --thing-name MyGreengrassCore \
  --thing-group-name MyGreengrassCoreGroup \
  --thing-policy-name GreengrassV2IoTThingPolicy \
  --tes-role-name GreengrassV2TokenExchangeRole \
  --tes-role-alias-name GreengrassCoreTokenExchangeRoleAlias \
  --component-default-user ggc_user:ggc_group \
  --provision true \
  --setup-system-service true \
  --deploy-dev-tools true
```

a. `/greengrass/v2` : IoT Greengrass 코어 소프트웨어가 설치될 루트 디렉터리이다.

b. `greengrassCore` : Greengrass 코어 소프트웨어 설치를 위해 다운로드 및 압축해제를 한 디렉터리 명이다.

c. `region` : 리소스를 찾거나, 없을 경우 생성할 리전명이다.

d. `MyGreengrassCore` : IoT thing의 이름이다. 만약 없으면 인스톨러가 배포(생성)한다.
    - 만약 해당 계정에 해당 thing 네임이 있고, AWS 상에 생성 되어있다면 인스톨러가 해당하는 thing에 대한
인증서를 다운로드 및 설치한다.
    - 만약 해당 계정에 네임이 없다면 생성하고 인증서 또한 생성한다.

e. `MyGreengrassCoreGroup` : thing group의 이름이다. 없으면 생성하고 thing을 자동으로 그룹에 추가한다. 
    - 만약 해당 계정에 이미 생성디어있다면, 배포 설정에 따른 소프트웨어를 설치하고 다운로드 받고 실행한다. (컴포넌트들)

f. `GreengrassV2IoTThingPolicy` : IoT policy의 이름이다. 없으면 생성하고 적절한 권한을 갖는 정책을 추가한다. 

g. `GreengrassV2TokenExchangeRole` : 일시적으로 AWS Credential 을 얻을 수 있는 IAM 롤의 이름이다. 
    - 만약 없다면 인스톨러가 생성하고, `GreengrassV2TokenExchangeRoleAccess`와 같이 `Access`를 추가한 IAM 롤을 생성한다.

h. `GreengrassCoreTokenExchangeRoleAlias` : .

위 명령어를 적절히 성공하면 다음과 같은 출력이 나온다.

```console
Successfully configured Nucleus with provisioned resource details!
Configured Nucleus to deploy aws.greengrass.Cli component
Successfully set up Nucleus as a system service
```

### 컴포넌트 배포 예시

```shell
aws greengrassv2 create-deployment \
  --target-arn "arn:aws:iot:region:account-id:thing/MyGreengrassCore" \
  --components '{
    "aws.greengrass.Cli": {
      "componentVersion": "2.4.0"
    }
  }'
```

위 명령어는 aws cli를 사용하여 deployment를 생성하는 것.

컴포넌트로 greengrass cli를 명시했다.

이는 추후 우리가 `recipe` 을 생성하여 배포하고자 하는 컴포넌트를 정의 할 수 있다. 

빌트인 및 사용자 정의 컴포넌트 모두 사용 가능

![](./install_success.png)