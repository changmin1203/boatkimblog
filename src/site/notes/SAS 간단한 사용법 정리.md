---
{"dg-publish":true,"dg-home":true,"permalink":"/sas/","tags":["gardenEntry"]}
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
```sas
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
```sas
FILENAME REFFILE '/home/u64253833/SAS 튜토리얼/ex1_1.csv';

PROC IMPORT DATAFILE=REFFILE
	DBMS=CSV
	OUT=WORK.IMPORT1;
	GETNAMES=YES;
RUN;
```

3. 업로드된 일시데이터 -> 영구 sas7bdat 으로 저장하기 
![스크린샷 2026-03-21 오후 11.25.01.png](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-21%20%EC%98%A4%ED%9B%84%2011.25.01.png)

# 예제를 통한 실습

## 데이터 불러오기 및 새 변수 생성
>[!note] 
>1. "HN18_ALL" SAS DATA를 불러와서,
>	- 위에서 설명한 변수만을 선택하고
>	- 여자, 그리고 45세 이상인 대상만을 분석하시오.
>	- 연령을 45-55, 55-60, 60세 이상으로 나누고 AGEGP라는 변수를 생성
>	- 신장과 체중을 이용하여 BMI (kg/m2)를 생성
>	- 일시적 SAS DATA인 TEMP1을 만드시오.


```sas
/*데이터 불러오기 */
LIBNAME hn18 '/home/u64253833/SAS 튜토리얼/';

/*KEEP: 사용 변수만 남기기 */
data TEMP1;
set hn18.hn18_all(keep= ID sex age BS1_1 BD1 HE_wt HE_ht HE_wc N_en N_cho N_fat HE_chol);
if sex =2 and age>=45; /*여성, 45세 이상만 남김*/

/*AGEGP 생성 */

if age>= 60 then AGEGP = 3;
else if 55<=age<60 then AGEGP = 2;
else if 45<=age<55 then AGEGP = 1;
else AGEGP = 'NULL';

/*BMI 생성  */

BMI = HE_wt/(HE_ht/100)**2;

/*정확한 순위 반영을 위해 BS1_1, BD1의 코딩 형식 변경  */
if BS1_1 = 1 then BS1_1=2;
else if BS1_1 = 2 then BS1_1=3;
else if BS1_1 = 3 then BS1_1=1;
else BS1_1 =.;

if BD1 = 9 then BD1=.;
run;
```

 - 사용할 변수만 남기기 = KEEP
 - if 문은 T/F를 반환하지 않고 T만 남긴다.

## 산점도,  상관계수 추정 및 검정
>[!note] 
> 각 독립변수와 종속변수의 관련성을 보기 위해 X-Y 산점도(multi-scatter plot)를 그리고, 상관계수를 추정하고, 검정하시오.

```sas
/*산점도*/
proc sgscatter data = TEMP1;
matrix age HE_wt HE_ht HE_wc N_en N_cho N_fat BMI HE_chol;
run;
```
![스크린샷 2026-03-27 오후 7.08.47.png\|400](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-27%20%EC%98%A4%ED%9B%84%207.08.47.png)
 - 산점도에서 허리둘레-체중, 에너지섭취량-탄수화물섭취량, 체중-신장의 양의 상관관계를 관찰할 수 있다. 

### 정규성 검정
- 피어슨 상관계수의 가정을 위해 qqplot을 통해 정규성을 검정한다.
```sas
/*피어슨vs스피어맨 결정을 위해 정규성 검정 */
proc univariate data= TEmp1 normal plot;
var age HE_wt HE_ht HE_wc N_en N_cho N_fat BMI HE_chol;
run;
```
 - 확인 결과 연속형 변수에 대한 정규성 확인 
	 - age, HE-wt, HE_ht, HE_wc, BMI, HE_chol
		 - K-S 검정 결과 기각되었으나(특정 분포 따르지 않으나), n수 및 qqplot 고려하였을 때 정규성 띔
	 - 예시(HE_wc)

![스크린샷 2026-03-27 오후 7.58.25.png\|grid\|228](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-27%20%EC%98%A4%ED%9B%84%207.58.25.png)  ![스크린샷 2026-03-27 오후 8.04.44.png\|grid\|238](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-27%20%EC%98%A4%ED%9B%84%208.04.44.png)

 - 정규성 미확인
		 - N_en, N_cho,N_fat

![스크린샷 2026-03-27 오후 8.13.57.png\|grid](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-27%20%EC%98%A4%ED%9B%84%208.13.57.png) ![스크린샷 2026-03-27 오후 8.14.04.png\|grid](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-27%20%EC%98%A4%ED%9B%84%208.14.04.png)
## 상관계수 추정(피어슨, 스피어만)

```SAS
/*연속형+정규성o 에 대한 피어슨 상관계수 추정, 검정 -> HE_chol 과의 상관계수만 = with */
proc corr data= TEMP1 fisher(BIASADJ=NO alpha=0.05);
var age HE_wt HE_ht HE_wc BMI ;
with HE_chol;
run;
```
![스크린샷 2026-03-27 오후 8.17.43.png](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-27%20%EC%98%A4%ED%9B%84%208.17.43.png)
 - 피어슨 상관계수 검정 결과
	 - age, HE_ht, HE_wc 는 HE_chol 과 유의미한 선형 관계를 지닌다.
```sas
/*연속형+정규성x, 범주형에 대한 스피어만 상관계수 추정, 검정*/
proc corr data = temp1 spearman fisher(BIASADJ=NO alpha=0.05);
var N_en N_cho N_fat AGEGP BS1_1 BD1;
with HE_chol;
run;
```

![스크린샷 2026-03-27 오후 8.17.53.png](/img/user/5.%20Attatchments/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202026-03-27%20%EC%98%A4%ED%9B%84%208.17.53.png)
 - 스피어만 상관계수 검정 결과
	 - N_cho, N_fat, AGEGP, BD1은 유의미한 선형관계에 있다. 
