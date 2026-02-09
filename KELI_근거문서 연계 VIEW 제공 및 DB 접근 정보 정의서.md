# 근거문서 연계 VIEW 제공 및 DB 접근 정보 정의서  
(ERP ↔ 그룹웨어 연계용)

본 문서는 ERP 시스템에서 그룹웨어 결재문서를 근거문서 형태로 조회·사용할 수 있도록 하기 위해,  
그룹웨어 측에서 제공하는 근거문서 조회 VIEW, 컬럼 명세,  
부서코드 정규화 함수(FUNCTION),  
그리고 DB 접속 방식 및 조회 방법을 정의한 문서입니다.

---

# 1. 데이터베이스 접속 정보 (To ERP)

ERP에서 그룹웨어 DB의 근거문서 VIEW를 조회하기 위한 접속 정보입니다.

| 항목 | 값 |
|------|------|
| DB 종류 | MariaDB 10.11 |
| 호스트 | {기관 내부망 접속주소} |
| 포트 | 3306 |
| 스키마 | {gwdb} |
| 계정 | {erp_gw_link} |
| 권한 | SELECT, FUNCTION 실행 |
| 접속 방식 | ERP → 그룹웨어 DB 직접 READ ONLY |

※ 실제 정보는 공문 또는 별도 전달된 보안 문서 기준으로 입력합니다.

---

# 2. 근거문서 연계 VIEW 정의서  
VIEW 명: **VW_AT_DOCBOX_8010**

## 2.1 제공 목적
- ERP에서 그룹웨어 문서 메타정보를 직접 조회  
- 열람제한 문서(VIEWRESTRICTTYPE ≠ 0)는 자동 제외  
- 기록물 등록대장(APPLID=8010) 문서만 조회  
- 문서함 기준부서 정규화를 위한 BOX_F=1 부서코드 제공

---

# 3. VIEW 컬럼 명세 (최신 스키마 기반)

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

---

# 4. 문서함 기준부서 조회  
문서함을 사용하지 않는 부서는 자동으로 상위부서 문서함을 사용해야 합니다.  
이를 위해 그룹웨어는 정규화된 문서함 기준부서코드를 제공하는 FUNCTION을 제공합니다.

## 4.1 함수명  
FN_GET_BOX_DEPT_CODE

## 4.2 기능 목적  
입력 부서(DEPT_CODE)의  
- 본인 부서  
- 상위 부서(DEPT_TREE 기반)  

중 문서함보유(BOX_F=1)으로 설정된 부서코드를 반환합니다.

## 4.3 FUNCTION 사용 예시  
SELECT FN_GET_BOX_DEPT_CODE('사용자부서코드') AS BOX_DEPT;

이 값으로 VIEW의 OWNER_DEPT_CD 와 비교해 조회합니다.

---

# 5. VIEW 조회 방법 안내 (ERP 적용)

ERP는 그룹웨어 VIEW에 대해 SELECT만 수행합니다.
VW_AT_DOCBOX_8010 쿼리내 부서코드는 반드시 FN_GET_BOX_DEPT_CODE 함수의 결과값을 입력합니다.

---

## 5.1 부서 기준 조회  
SELECT *  
FROM VW_AT_DOCBOX_8010  
WHERE OWNER_DEPT_CD = '부서코드'  
ORDER BY REGISTDATE DESC;

---

## 5.2 기안일·등록일 등 날짜 범위 조회  
SELECT *  
FROM VW_AT_DOCBOX_8010  
WHERE REGISTDATE BETWEEN '2024-01-01T00:00:00' AND '2024-12-31T23:59:59'  
ORDER BY REGISTDATE DESC;

---

## 5.3 기타 검색 조건  
뷰 컬럼 명세를 참고하여 TITLE, DRAFTERNAME 등 조건으로 조회 가능합니다.

---

# 6. 연계 처리 흐름 (ERP 기준)

1. ERP에서 “근거문서 선택” 버튼 클릭  
2. ERP → 그룹웨어 VIEW(VW_AT_DOCBOX_8010) 조회  
3. ERP 화면에 목록 표시  
4. 사용자가 문서 선택  
5. ERP → 그룹웨어 문서보기 API 호출  
6. 그룹웨어가 원문을 렌더링하여 ERP 화면에서 열람 가능

---

# 7. 참고 (협의된 조건)

- 열람제한문서(VIEWRESTRICTTYPE ≠ 0)는 VIEW에서 제외  
- 원문 문서는 API로만 제공되며 DB에서 직접 볼 수 없음  
- ERP는 READ ONLY 정책  
