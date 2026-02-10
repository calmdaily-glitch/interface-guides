# 부재연동(Absence Sync) 인터페이스 연계정의서


## 1. 요청(Request)

### 1.1 요청 URL

```
http://{GW_IP}:{GW_PORT}/ATWork/at_abs/abs_sync.jsp
  ?key=absent_sync_
  &emp_cd={AES 암호화된 사번코드}
  &sa_emp_cd={AES 암호화된 대리결재자 사번}
  &not_sanc_cd={부재사유코드}
  &f={IN|DEL}
  &st_dt={부재시작일(YYYYMMDDHHMI)}
  &ed_dt={부재종료일(YYYYMMDDHHMI)}
  &encryptkey={AES 암호화된 키}
```

### 1.2 요청 파라미터 정의

| 파라미터명 | 설명 | 비고 |
|------------|-------|------|
| key | 헤더값, 고정값 `absent_sync_aks` | 필수 |
| emp_cd | **AES 암호화된 사원 사번** | 필수 |
| sa_emp_cd | **AES 암호화된 대직자(대리결재자) 사번** | 선택 |
| st_dt | 부재 시작일 (YYYYMMDDHHMI) | 필수 |
| ed_dt | 부재 종료일 (YYYYMMDDHHMI) | 필수 |
| not_sanc_cd | 부재사유 코드 | 코드표 참조 |
| f | IN=등록, DEL=삭제 | 필수 |
| encryptkey | **AES로 암호화된 키 값** | 필수 |

### 1.3 부재사유 코드표

| 코드 | 의미 | 코드 | 의미 | 코드 | 의미 | 코드 | 의미 |
|------|------|------|------|------|------|------|------|
| 01 | 교육 | 05 | 조퇴 | 09 | 특별휴가 | 13 | 휴직 |   
| 02 | 출장 | 06 | 연가 | 10 | 결근 | 14 | 퇴직 |   
| 03 | 외출 | 07 | 병가 | 11 | 지참 | 15 | 자동응답 |
| 04 | 휴가 | 08 | 청가 | 12 | 부재 | NULL | 휴가 | 
 
 
 
 

<div style="page-break-after: always;"></div>

## 2. 암호화 처리 규칙 (AES)

### 2.1 암호화 개요
본 인터페이스에서는 개인정보 보호를 위해  
사번 관련 파라미터(`emp_cd`, `sa_emp_cd`) 및  
인증용 파라미터(`encryptkey`)를 **AES 방식으로 암호화하여 전달**한다.

### 2.2 암호화 적용 대상 파라미터

| 파라미터명 | 암호화 여부 |
|------------|-------------|
| emp_cd | AES 암호화 |
| sa_emp_cd | AES 암호화 |

### 2.3 암호화 규칙 요약

- **Algorithm:** AES  
- **Key Length:** 128bit  
- **Mode:** ECB  
- **Padding:** PKCS5Padding  

> Java 기준 참고 구현은 `CryptoAESUtil.java` 로 제공됨.

### 2.4 암호화 처리 절차

#### (1) encryptkey 생성
- 원본키: 임의값 또는 현재 timestamp  
- 인증코드(`AUTH_CODE`)를 사용하여 AES 암호화  
- 결과값을 `encryptkey` 파라미터로 전달

#### (2) 사번 파라미터 암호화
- 복호화 키: 위 (1) 단계에서 사용한 **원본키**  
- 대상: `emp_cd`, `sa_emp_cd`
- AES 암호화 후 요청 파라미터로 전달

<div style="page-break-after: always;"></div>

## 3. 응답(Response)

### 3.1 JSON 구조

성공:
```json
{"result": "Success"}
```

실패:
```json
{"result": "오류메세지내용"}
```

### 3.2 오류 메시지 목록

- 시간의 형식이 맞지 않습니다.  
- 삭제할 부재 정보가 없습니다.  
- 시작일이 종료일보다 나중에 올 수 없습니다.  
- 부재자와 대리결재자는 같을 수 없습니다.  
- 사용자 정보가 없습니다.  
- 부재설정 사용자는 결재대리자로 설정할 수 없습니다.

## 6. 첨부 파일 목록
- `CryptoAESUtil.java` (AES 암·복호화 유틸리티 클래스)  
- `commons-codec-1.10.jar` (암호화 라이브러리)