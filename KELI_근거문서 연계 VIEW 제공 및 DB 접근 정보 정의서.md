## 근거문서 연계 VIEW 제공 및 DB 접근 정보 정의서  

## 1. 데이터베이스 접속 정보

ERP에서 그룹웨어 DB의 근거문서 VIEW를 조회하기 위한 접속 정보입니다.

| 항목 | 값 |
|------|------|
| DB 종류 | MariaDB |
| 그룹웨어계정 | gwuser |
| 권한 | SELECT, FUNCTION 실행 |

## 2. 제공 VIEW 목록
| VIEW 명 | 설명 |
|--------|------|
| **VW_AT_DOCBOX_8010** | 기록물 등록대장 목록 조회용 VIEW |
| **VW_AT_DOCBOX_3010** | 개인 사용자 공람완료 목록 조회용 VIEW |

## 3. VW_AT_DOCBOX_8010 – 기록물등록대장 목록 조회용 VIEW

| 컬럼명 | Data Type | 설명 |
|--------|-----------|--------|
| APPRID | varchar(20) | 문서ID |
| TITLE | varchar(400) | 문서 제목 |
| SECURITYLEVEL | tinyint(4) | 보안등급 |
| LASTSIGNERNAME | varchar(80) | 최종결재자 |
| DRAFTDATE | varchar(19) | 기안일(YYYY-MM-DDTHH:MM:SS) |
| DRAFTERID | varchar(20) | 기안자ID |
| DRAFTERNAME | varchar(64) | 기안자명 |
| ORGDRAFTERID | varchar(20) | 원기안자ID |
| ORGDRAFTERNAME | varchar(200) | 원기안자명 |
| DRAFTDEPTID | varchar(20) | 기안부서ID |
| DRAFTDEPTNAME | varchar(200) | 기안부서명 |
| ORGDRAFTDEPTID | varchar(20) | 원기안부서ID |
| ORGDRAFTDEPTNAME | varchar(200) | 원기안부서명 |
| ORGAPPRID | varchar(20) | 원문서ID |
| SAVEDEPTID | varchar(20) | 보관부서ID |
| APPROVALTYPE | int(11) | 결재유형 코드 |
| APPROVALTYPESTR | varchar(10) | 결재유형명(일반/보고/수신) |
| REGISTDATE | varchar(19) | 등록일(YYYY-MM-DDTHH:MM:SS) |
| OWNERID | varchar(20) | 기준부서ID(DEPT_ID) |
| OWNER_DEPT_CD | varchar(20) | 정규화된 기준부서코드(BOX_F=1) |

<div style="page-break-after: always;"></div>

## 4. 문서함 기준부서 정규화 FUNCTION

### 4.1 함수명  
`FN_GET_BOX_DEPT_CODE`

### 4.2 기능 목적  
입력 부서(DEPT_CODE)에 대해:

- 본인 부서  
- 상위 부서(DEPT_TREE 기반)  

중 **문서함 보유(BOX_F=1)** 부서코드를 반환합니다.

### 4.3 사용 예시

```sql
SELECT FN_GET_BOX_DEPT_CODE('부서코드') AS BOX_DEPT;
```

## 5. VIEW 조회 방법

**VW_AT_DOCBOX_8010 쿼리내 부서코드는 반드시 FN_GET_BOX_DEPT_CODE 함수의 결과값을 입력합니다.**


### 5.1 부서 기준 조회  
```sql
SELECT *  
FROM VW_AT_DOCBOX_8010  
WHERE OWNER_DEPT_CD = '부서코드'  
ORDER BY REGISTDATE DESC;
```

### 5.2 기안일·등록일 등 날짜 범위 조회  
```sql
SELECT *  
FROM VW_AT_DOCBOX_8010  
WHERE REGISTDATE BETWEEN '2026-01-01T00:00:00' AND '2026-12-31T23:59:59'  
ORDER BY REGISTDATE DESC;
```

### 5.3 기타 검색 조건  
뷰 컬럼 명세를 참고하여 TITLE, DRAFTERNAME 등 조건으로 조회 가능합니다.

<div style="page-break-after: always;"></div>

## 6. VW_AT_DOCBOX_3010 – 개인 사용자 공람완료 조회 VIEW

### 6.1 제공 목적
- ERP 화면에서 사용자별 공람완료 문서 목록 조회
- 조회 조건: 사용자 사번(EMP_CD)

### 6.2 컬럼 명세
| 컬럼명 | Data Type | 설명 |
|--------|-----------|--------|
| APPRID | varchar(20) | 문서ID |
| TITLE | varchar(400) | 문서 제목 |
| SECURITYLEVEL | tinyint(4) | 보안등급 |
| LASTSIGNERNAME | varchar(80) | 최종결재자 |
| DRAFTDATE | varchar(19) | 기안일(YYYY-MM-DDTHH:MM:SS) |
| DRAFTERID | varchar(20) | 기안자ID |
| DRAFTERNAME | varchar(64) | 기안자명 |
| ORGDRAFTERID | varchar(20) | 원기안자ID |
| ORGDRAFTERNAME | varchar(200) | 원기안자명 |
| DRAFTDEPTID | varchar(20) | 기안부서ID |
| DRAFTDEPTNAME | varchar(200) | 기안부서명 |
| ORGDRAFTDEPTID | varchar(20) | 원기안부서ID |
| ORGDRAFTDEPTNAME | varchar(200) | 원기안부서명 |
| ORGAPPRID | varchar(20) | 원문서ID |
| SAVEDEPTID | varchar(20) | 보관부서ID |
| APPROVALTYPE | int(11) | 결재유형 코드 |
| APPROVALTYPESTR | varchar(10) | 결재유형명(일반/보고/수신) |
| REGISTDATE | varchar(19) | 등록일(YYYY-MM-DDTHH:MM:SS) |
| OWNERID | varchar(20) | 기준부서ID(DEPT_ID) |
| EMP_CD | varchar(20) | 사원번호 |

### 6.3. VW_AT_DOCBOX_3010 조회 방법
```sql
SELECT *
FROM VW_AT_DOCBOX_3010
WHERE EMP_CD = '사원번호'
ORDER BY REGISTDATE DESC;
```
<div style="page-break-after: always;"></div>

## 7. 연계 처리 흐름 (ERP 기준)

1. ERP에서 “근거문서 선택” 버튼 클릭  
2. ERP → 그룹웨어 VIEW 조회  
  - 근거문서: **VW_AT_DOCBOX_8010**
  - 공람완료: **VW_AT_DOCBOX_3010**
3. ERP 화면에 목록 표시  
4. 사용자가 문서 선택  
5. ERP → 그룹웨어 문서보기 API 호출  
6. 그룹웨어가 원문을 렌더링하여 ERP 화면에서 열람 가능

## 8. 종합정리
| 구분   | VIEW 명            | 조회 기준                       | 대상            |
| ---- | ----------------- | --------------------------- | ------------- |
| 근거문서 | VW_AT_DOCBOX_8010 | 부서(FN_GET_BOX_DEPT_CODE 결과) | 기록물등록대장(8010) |
| 공람완료 | VW_AT_DOCBOX_3010 | 사용자 사번(EMP_CD)              | 개인 공람완료(3010) |




​                                                                                                 Copyright Aintop System Co.,Ltd. All Rights Reserved. 
