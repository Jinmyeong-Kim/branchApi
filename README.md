# BranchApi
### Spring boot를 활용하여 특정 데이터를 조회하는 프로그램입니다.


## 1. 개발 환경
- Languages : Java(ver 1.8)
- Framework and libraries : Spring boot(2.2.4), Gradle(3.x), Mybatis, Oracle(AWS), Lombok
- Tools : Spring STS(4.4.5), Sql developer(19.2.1)
- OS : Windows 10



## 2. 실행방법 및 빌드
1) github 주소나 google drive에서 zip파일을 받아주세요.
2) 받으신 zip파일을 STS에서 import 해주세요.(eclipse에서 import 시 안 되실 수 있습니다.)
* 개발툴(STS) 다운로드 : 
  https://spring.io/tools 에서 운영체제에 맞는 설치파일을 받아주세요.
* import 방법 :

  Package Explorer에서 우클릭 후
  
  import -> General(Existing Projects into Workspace) -> Select archive file
  하셔서
  다운받으신 zip 파일을 선택하신 후 finish를 눌러주세요.
* ***import 후 에러가 날 시 :***

  원인 : src/test/java에 있는 테스트 파일들에 선언된 @RunWith 부분이 문제의 원인입니다.
  
  ***해결방법 : Project 우클릭 후 
            Build Path-> Add Libraries -> JUnit -> JUnit4 -> Finish를 해주세요.***
3) Project 우클릭 후 Run as -> Spring boot App을 클릭해서 build 해주세요.
4) 결과를 확인하실 수 있는 URL은 다음과 같습니다.

   (편의상 localhost라 했으나 사용하고 계시는 pc의 IP를 적으셔도 됩니다. 포트번호 8000이 겹친다면 application.yml에서 숫자만 바꿔주세요.)
* 연결 확인용 URL : http://localhost:8000/
* 문제1 URL : http://localhost:8000/test1
* 문제2 URL : http://localhost:8000/test2
* 문제3 URL : http://localhost:8000/test3
* 문제4 URL : http://localhost:8000/test4?brName=지점명


## 3. 문제 해결방법
### <환경설정 관련 문제 해결방법>
### 3-1. DB 구축
AWS를 이용하여 Oracle로 구축하였습니다.
- EC2 VPC 보안그룹 생성 및 인바운드 설정
- 외부에서도 접근 가능하게 하기 위한 AWS RDS public 접속 설정
- RDS 서브넷 그룹 설정 및 보안규칙 설정
- AWS의 DB 인스턴스 안에 SQL Developer를 사용한 테이블 및 인덱스 생성

### 3-2. Mybatis 설정 및 연동
- build.gradle : mybatis와 ojdbc 관련 dependency 설정
- MybatisConfig.java : SqlSessionTemplate 설정
- application.yml : src/main/resources/application.yml에서 AWS의 RDS와 맞게 class-name, username 등 설정

  (application.yml에서 #DB-Oracle 부분 참조)

### 3-3. 출력 값에 맞는 key값 설정 위해 resultType으로 CamelHashMap 추가
acct_no를 acctNo로 출력하기 위해 xml에서 restulType으로 CamelHashMap을 받게 추가
(CamelHashMap.java 참조)

### 3-4. 각 문제별 해결 방법
문제 1~4들은 파라미터로 받을 값이 없거나 지점명만 받는 단순 조회 기능들이기에 method는 모두 Get으로 설정하였습니다.
#### Q1 : 2018년, 2019년 각 연도별 합계 금액이 가장 많은 고객을 추출
- 관련 controller :　MainController에서 "test1"로 매핑된 부분입니다.
- 관련 service :　memberService.selMaxAmtMemberByYear()
- 프로세스 :　selMaxAmtMemberByYear 쿼리를 이용하여 data를 추출합니다.
- 데이터 추출 방법 :　

  쿼리에서 연도별, 지점별로 합계 금액이 높은 순으로 랭킹을 매긴 후 랭킹이 1인 데이터를 연도별로 정렬하여 가져왔습니다.
- 데이터 출력 형식을 위한 방법 : 

  - LinkedList를 사용 : 2018년 데이터가 2019년보다 앞으로 오게 하였습니다.
  
  - resultType으로 ***CamelHashMap*** 사용 : 요구된 출력값의 키로 출력이 되게 하였습니다.(key값이 acctNo, acctNm과 같은 형식으로 설정) 
  
  - ***CamelHashMap***는 출력값에 맞게 출력하기 위해 타 문제에서도 동일하게 사용하였으므로 다른 부분에서는 설명하지 않겠습니다.

#### Q2 : 2018년 또는 2019년에 거래가 없는 고객을 추출
- 관련 controller :　MainController에서 "test2"로 매핑된 부분입니다.
- 관련 service :　memberService.selNoExchangeMember()
- 프로세스 : selNoExchangeMember 쿼리를 이용하여 raw data 산출 후 연도별로 분리작업하여 데이터를 산출합니다.
- 데이터 추출 방법 :
  1) 쿼리에서 전체 회원의 각 연도 거래수를 0으로 가설정 한 데이터
    
    고객 | 18년도 거래수 | 19년도 거래수   
    ---------|--------------|-------------   
    A|0|0|   
    B|0|0|   
    C|0|0|   
    
    를   
    2년간 1건 이상의 거래가 있는 회원들의 실제 거래수를 나타낸 데이터
    
    고객 | 18년도 거래수 | 19년도 거래수   
    ---------|--------------|-------------   
    A|3|0|   
    B|0|4|   
    
    와    
    union all로 합친 후 계좌번호가 같은 데이터끼리 거래수를 SUM하여 전체 회원이 2년간 실거래한 데이터를 뽑습니다.
     
    고객 | 18년도 거래수 | 19년도 거래수   
    ---------|--------------|-------------   
    A|3|0|   
    B|0|4|   
    C|0|0|   
    
    
  2) 전체 거래 데이터 - 2년간 모두 거래한 데이터 = ***2018년 또는 2019년에 미거래 데이터***이므로   
  
     1)의 데이터에서 2018년과 2019년 모두 거래한 데이터를 제거해줍니다.   
     
     GROUP BY 계좌번호 **HAVING SUM(18년도 거래수)*SUM(19년도 거래수) = 0***   
     
     (2년간 모두 거래한 고객은 18년도 거래수 * 19년도 거래수가 0 이상의 상수일 것이기 때문입니다.)
     
  3) 2018년과 2019년 모두 미거래한 데이터 C의 row를 2줄로 나타내고 한 해에만 미거래인 데이터는 한 줄로 나타내기 위해 ***CONNECT BY***를 사용하여 조인합니다.
     
    
    연도 | 고객 | 계좌   
    -----|-----|-------   
    2018|A|1|   
    2019|B|2|   
    2018|C|3|   
    2019|C|3|   
- 데이터 출력 형식을 위한 방법 :  
  - 연도별로 분리 : 쿼리에서 뽑은 데이터를 연도별로 분리 후 분리한 데이터들을 하나의 LinkedList에 연도순으로 리스트에 담습니다.
  - LinkedList를 사용 : 연도별로 나눈 데이터를 합칠 때 2018년 미거래 고객 데이터가 2019년 미거래 고객 데이터보다 먼저 오게 먼저 담았습니다.

#### Q3 : 연도별 관리점별 거래금액 합계를 구하고 합계금액이 큰 순서로 출력
##### 4번 문제에서 분당점과 판교점이 통합되었다 하였지만 3번 문제를 풀 시에는 판교점은 순수 판교점의 총 거래금액을, 분당점은 순수 분당점의 총 거래금액을 산출하였습니다.
- 관련 controller :　MainController에서 "test3"로 매핑된 부분입니다.
- 관련 service :　branchService.selBrnchAmtByYearDescAmt()      
- 프로세스 :　selBrnchAmtByYearDescAmt 쿼리를 이용하여 raw data 산출 후 연도별로 분리작업을 통해 데이터를 산출합니다.
- 데이터 추출 방법 :   
쿼리에서 연도별 관리점별 거래금액 합계와 조회기간의 연도들이 표시된 데이터를 뽑습니다.
    
    연도   |   지점명   |   지점코드   |   거래금액   |   연도리스트   
    -------|-----------|-------------|--------------|-------------   
    2018|가|A|1000|2018,2019   
    2018|나|B|2700|2018,2019   
    2019|가|A|3000|2018,2019   
    2019|나|B|1200|2018,2019   
  - 연도리스트 :　SELECT LISTAGG(year, ',') WITHIN GROUP(ORDER BY YEAR)
- 데이터 출력 형식을 위한 방법 :  
  - 연도별로 분리 : 쿼리에서 뽑은 데이터를 연도와 연도리스트 항목을 제거 후 연도별로 분리했습니다.
  - LinkedList를 사용 : 연도별로 나눈 데이터를 합칠 때 2018년 데이터가 먼저 오게 먼저 담았습니다.

#### Q4 : 지점명을 입력하면 해당지점의 거래금액 합계를 출력
- 관련 controller :　MainController에서 "test4"로 매핑된 부분입니다.
- 관련 service :　branchService.selBrnchTTLBrCd(brName), branchService.selBrnchAmt(brnchCdInfo)
- 프로세스 : 해당 지점의 총 거래금액을 뽑기에 앞서 **지점 테이블과 지점 이동 테이블을 조합하여** 해당 지점의 정보(지점코드, 타 지점 관리 코드)를 가져옵니다.(아래의 ***4. Table정보***에서 TB_BRANCH_MOVE 참조)
    1) 해당 지점 정보가 없을 시(폐점이거나 지점명이 잘못됨) : 에러 메세지 및 404 상태 전달
    2) 해당 지점 정보가 있을 시 : 
        - 타 지점 관리 코드가 있을 시 : 해당 지점의 매출과 관리하게 된 지점의 매출이 합산되어 나타남 BR_CODE IN ('A', 'B', ‘C’, ‘D’)
        - 타 지점 관리 코드가 없을 시 : 해당 지점지점의 매출만 나타남 BR_CODE IN ('A')
- 데이터 추출 방법 : 
지점명을 이용하여 폐점이 아닌 지점의 지점코드와 통합 관리하는 지점코드 추출 후 

    
    지점코드   |   총 지점 코드(검색지점 + 관리지점)   |
    -------|-----------|   
    A|A,B   
    C|C   
    D|D      
    
    
(실제로 쿼리를 돌리면 한 줄만 나오나 이해를 돕기 위해 여러 줄로 표시했습니다.)  
해당 정보를 이용하여 지점의 총 거래금액을 산출합니다.


## 4. TABLE 정보
- 고객 테이블(TB_ACCOUNT)   
CREATE TABLE TB_ACCOUNT (   
　ACCT_NO VARCHAR(8) NOT NULL, // 계좌번호   
　NAME VARCHAR(50) NOT NULL, // 이름   
　BR_CODE VARCHAR(1) NOT NULL, // 지점 코드   
　REG_ID VARCHAR(20) NOT NULL, // 등록자 아이디   
　REG_DT DATE DEFAULT SYSDATE NOT NULL, // 등록일   
　MOD_ID VARCHAR(20) NOT NULL, // 수정자 아이디   
　MOD_DT DATE DEFAULT SYSDATE NOT NULL, // 수정일   
　PRIMARY KEY(ACCT_NO)   
);   
CREATE INDEX IX_ACCOUNT_001 ON TB_ACCOUNT(REG_ID);   

- 지점 테이블(TB_BRANCH)   
CREATE TABLE TB_BRANCH (   
　BR_CODE VARCHAR(6) NOT NULL, // 지점 코드   
　BR_NAME VARCHAR(50) NOT NULL, // 지점명   
　CLOSE_YN VARCHAR(1) DEFAULT 'N' NOT NULL, // 폐점 여부   
　REG_ID VARCHAR(20) NOT NULL, // 등록자 아이디   
　REG_DT DATE DEFAULT SYSDATE NOT NULL, // 등록일   
　MOD_ID VARCHAR(20) NOT NULL, // 수정자 아이디   
　MOD_DT DATE DEFAULT SYSDATE NOT NULL, // 수정일   
　PRIMARY KEY(BR_CODE)   
);   
CREATE INDEX IX_BRANCH_001 ON TB_BRANCH(BR_NAME);   

- 거래내역 테이블(TB_STOCK_EXCHANGE)   
CREATE TABLE TB_STOCK_EXCHANGE (   
　REG_DT VARCHAR(8) NOT NULL, // 등록일   
　ACCT_NO VARCHAR(8) NOT NULL, // 계좌번호   
　EXCG_NUM NUMBER NOT NULL, // 거래번호   
　REG_ID VARCHAR(20) NOT NULL, // 등록자 아이디   
　AMT NUMBER NOT NULL, // 금액   
　FEE NUMBER NOT NULL, // 수수료   
　CANCLE_YN VARCHAR(1) NOT NULL, // 취소여부   
　MOD_ID VARCHAR(20) NOT NULL, // 수정자 아이디   
　MOD_DT DATE DEFAULT SYSDATE NOT NULL, // 수정일   
　PRIMARY KEY(REG_DT, ACCT_NO, EXCG_NUM)   
);   
CREATE INDEX IX_STK_EXCG_001 ON TB_STOCK_EXCHANGE(REG_DT, CANCLE_YN);   

- 지점 이동 테이블(TB_STOCK_EXCHANGE)   
CREATE TABLE TB_BRANCH_MOVE (   
　PRE_BR_CODE VARCHAR(6) NOT NULL, // 이전 지점 코드   
　NEW_BR_CODE VARCHAR(6) NOT NULL, // 새 지점 코드   
　REG_ID VARCHAR(20) NOT NULL, // 등록자 아이디   
　REG_DT DATE NOT NULL, // 등록일   
　MOD_ID VARCHAR(20) NOT NULL, // 수정자 아이디   
　MOD_DT DATE NOT NULL // 수정일   
);   
CREATE INDEX IX_BRNCH_MOVE_001 ON TB_BRANCH_MOVE(NEW_BR_CODE);   

## 5. 테스트 방법
- JUnit : src/test/java에서 각 서비스별로 기능을 검증
- Postman : content-type 검사(JSON), 입출력 데이터 형식 검사, 상태값 체크
- Browser : content-type 검사(JSON), 상태값 체크, 실제 브라우저에서 호출 시 이상이 없나 테스트
