부재연동(Absence Sync) 인터페이스 연계정의서
1. 개요

본 문서는 외부 시스템에서 그룹웨어 전자결재 시스템으로 부재 정보(등록/삭제) 를 연계하기 위한 규격을 정의한다.
연계 요청 파라미터, 응답 규격, 부재사유 코드, 암호화 처리 방식(AES 기반)을 포함한다.

2. 연계 방식 개요

Protocol: HTTP GET

Method: GET

Encoding: UTF-8

Response Format: JSON

암호화: AES 기반 커스텀 암호화 (별첨: CryptoAESUtil.java 사용)

3. 요청(Request)
3.1 요청 URL
http://{GW_IP}:{GW_PORT}/ATWork/at_abs/abs_sync.jsp
  ?key=absent_sync_
  &emp_cd={사번코드 또는 암호화된 사번코드}
  &sa_emp_cd={대리결재자사번}
  &re_emp_cd={대리접수자사번}
  &se_emp_cd={대리발송처리자사번}
  &not_sanc_cd={부재사유코드}
  &f={IN|DEL}
  &st_dt={부재시작일(YYYYMMDDHHMI)}
  &ed_dt={부재종료일(YYYYMMDDHHMI)}
  &encryptkey={암호화된 키}

3.2 요청 파라미터 정의
파라미터명	설명	비고
key	헤더값, 고정값 absent_sync_abs	필수
emp_cd	사원코드 또는 AES 암호화된 사원코드	암호화 사용 시 encryptkey 필수
st_dt	부재 시작일 (YYYYMMDDHHMI)	필수
ed_dt	부재 종료일 (YYYYMMDDHHMI)	필수
sa_emp_cd	대리결재자 사번	선택
re_emp_cd	대리접수자 사번	선택
se_emp_cd	대리발송처리자 사번	선택
not_sanc_cd	부재사유 코드	코드표 참조
f	IN=등록, DEL=삭제	필수
encryptkey	AES로 암호화된 키 값	암호화 사용 시 필수
3.3 부재사유 코드표
코드	의미
01	교육
02	출장
03	외출
04	휴가
05	조퇴
06	연가
07	병가
08	청가
09	특별휴가
10	결근
11	지참
12	부재
13	휴직
14	퇴직
15	자동응답
기타 문자	자동응답
공백 또는 NULL	휴가
4. 암호화 처리 규칙 (AES)
4.1 암호화 개요

부재연동 시 개인정보 보호를 위해 사원번호(emp_cd)를 AES 방식으로 암호화하여 전달할 수 있다.

암호화는 두 단계로 이루어진다.

(1) encryptkey 생성

원본키: 임의 값 또는 현재 timestamp

인증코드(AUTH_CODE) 기반으로 AES 암호화

결과값을 요청 파라미터 encryptkey 로 전달

(2) emp_cd 암호화

복호화키: 위에서 사용한 원본키

사번(emp_cd)을 암호화하여 emp_cd 파라미터로 전달

4.2 CryptoAESUtil.java

AES 기반 암·복호화 유틸리티

commons-codec-1.10.jar 사용

해당 소스는 본 연계 정의서의 첨부 파일로 별도 제공

5. 응답(Response)
5.1 JSON 구조

성공:

{"result": "Success"}


실패:

{"result": "오류메세지내용"}

5.2 오류 메시지 목록

시간의 형식이 맞지 않습니다.

삭제할 부재 정보가 없습니다.

시작일이 종료일보다 나중에 올 수 없습니다.

부재자와 대리결재자는 같을 수 없습니다.

부재자와 대리접수담당자는 같을 수 없습니다.

부재자와 대리발송담당자는 같을 수 없습니다.

사용자 정보가 없습니다.

해당 유저에는 접수담당자 권한이 없습니다.

해당 유저에는 발송담당자 권한이 없습니다.

부재설정되어 있는 사용자는 결재대리자로 설정할 수 없습니다.

부재설정되어 있는 사용자는 대리접수담당자로 설정할 수 없습니다.

부재설정되어 있는 사용자는 대리발송담당자로 설정할 수 없습니다.

6. 암호화 적용 예시
6.1 암호화 Key 생성 예시
String key = String.valueOf(System.currentTimeMillis());  // 원본 key
String encryptKey = CryptoAESUtil.encryptAES(key, CryptoAESUtil.AUTH_CODE);   

6.2 사번 암호화 예시
String empCode = "test001";
String encryptEmpCd = CryptoAESUtil.encryptAES(empCode, key);

7. 예시 요청 URL

암호화 적용 시:

http://GW_IP:GW_PORT/ATWork/at_abs/abs_sync.jsp
?key=absent_sync_
&emp_cd=8fa31fbec9d2a91ab3...
&encryptkey=a93bc19d0ef3a8e1f3d...
&sa_emp_cd=000123
&not_sanc_cd=04
&f=IN
&st_dt=202602050900
&ed_dt=202602051800

8. 첨부 파일 목록

CryptoAESUtil.java (AES 암호화·복호화 유틸리티 클래스)
commons-codec-1.10.jar (암호화 라이브러리)
