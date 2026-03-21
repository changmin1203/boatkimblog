---
{"dg-publish":true,"permalink":"/sas/"}
---


# SAS ondemand
 - https://welcome.oda.sas.com 에서 가입 후 사용
 - Launch - > SAS studio로 이동
	 - 온라인 서버에서 구동되는 SAS

# SAS studio 화면 구성
![스크린샷 2026-03-21 오후 8.56.05.png](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-21%20%EC%98%A4%ED%9B%84%208.56.05.png)
- 탐색기: 서버 파일 및 폴더, 라이브러리 
	- odaws(ondemand web server) apse(Asia Pacific Southeast 1) 해당 서버 내에서 실행 중임 의미
	- SAS 는 외부에서 데이터를 불러온 후 "SAS data set"에 저장 
		- SAS studio에서는 해당 서버에 올려야 data set으로 저장 가능함. 
		- 데이터 불러올 때 LIBNAME에 서버 경로 작성해야함.(파일 우클릭 후 속성 선택)
- 확장편집기: 코드, 로그, 결과 확인 가능

# DATA STEP
 - SAS data를 구성하는 단계
## 직접 입력하기
```
data 파일명;
		/* input 변수명, 이때 숫자 변수는 $표시*/
	input # class ID name $ gender $ dept age $reg;
	cards; /* cards 명령어로 직접 변수 입력 가능*/
	1 1 kim M 통계 20 서울
	1 2 choi F 전산 20 서울
	1 3 강희영 F 전산 20 서울
	;
	run;
```
## 외부 데이터 가져오기
 1. 데이터를 SAS에 업로드
![Pasted image 20260321225804.png](/img/user/5.%20Attatchments/Pasted%20image%2020260321225804.png)
	 - 서버 파일 및 폴더 메뉴에서 우클릭으로 파일업로드 선택 / 또는 위에 업로드 버튼을 누른다
		 - 업로드 시 로컬 -> SAS 서버 내로 파일 전송
	 - 새로만들기 누르면 SAS 프로그램, 데이터 가져 오기 가능 
		 - 데이터 가져오기 -> 서버 내 데이터를 해당 폴더 내로 가져와서 읽어줌.
2. 업로드 된 데이터를 가져오기(일시 데이터 )
![Pasted image 20260321230805.png](/img/user/5.%20Attatchments/Pasted%20image%2020260321230805.png)
 - 업로드 된 데이터 우클릭 - 데이터 가져오기 -> 아래쪽에 자동으로 코드 생성 -> RUN -> work.이름 으로 저장됨
	 - 코드로 하는 법(PROC IMPORT)
```
FILENAME REFFILE '/home/u64253833/SAS 튜토리얼/ex1_1.csv';

PROC IMPORT DATAFILE=REFFILE
	DBMS=CSV
	OUT=WORK.IMPORT1;
	GETNAMES=YES;
RUN;
```

3. 업로드된 일시데이터 -> 영구 sas7bdat 으로 저장하기 
![스크린샷 2026-03-21 오후 11.25.01.png](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-21%20%EC%98%A4%ED%9B%84%2011.25.01.png)