---
layout: post
title: 출근 시간과 전반적 일자리 만족도의 관계 분석 (1)
subtitle: "늦게 출근하는 일자리는 만족스러울까?"
type: "project"
project: true
text: true
author: "Eunhee Kim"
post-header: true
header-img: "img/header.jpg"
order: 1
---

## 🤔 통학생의 고민

당시 월화수목 9시 수업이라는 제정신 아닌 시간표대로 살고 있었다. 심지어 9시 수업들은 전공이라 뺄 수도 없어서 눈물 흘리며 꾸역꾸역 만원 버스와 지하철을 타던 기억이 난다. 피로도 덜 풀린 채로 일단 아홉시 수업에 출석은 했는데, 수업을 들었다는 흔적만 기억에 남는 경우가 꽤 있었다(내 경우에는 적어도 10시 반 수업부터 집중하며 수업을 들을 수 있었다).    

등교 시간이 10시만 되어도 정말 행복한데, <u>출근 시간도 늦춰진다면 일자리 만족도도 증가하지 않을까?</u> 졸리지도 않고, 출근 시간대가 아니라 교통 체증도 없을 테니 말이다.

## 💭 데이터 클리닝

데이터는 [한국노동패널조사](https://www.kli.re.kr/klips/index.do)의 17차 조사에서 가져왔다. 데이터는 설명변수인 <u>출근시간</u>, 종속변수인 <u>전반적 일자리 만족도</u>와 혼동변수를 포함하여 클리닝했다.

필요한 패키지 ```foreign```, ```dplyr``` 설치


```R
install.packages("foreign")
install.packages("dplyr")
library(foreign)
library(dplyr)
```

    Warning message:
    "unable to access index for repository http://www.stats.ox.ac.uk/pub/RWin/bin/windows/contrib/3.5:
      URL 'http://www.stats.ox.ac.uk/pub/RWin/bin/windows/contrib/3.5/PACKAGES'를 열 수 없습니다"
    
    package 'foreign' successfully unpacked and MD5 sums checked
    
    The downloaded binary packages are in
    	C:\Users\eunee\AppData\Local\Temp\Rtmpeu61lL\downloaded_packages


    Warning message:
    "unable to access index for repository http://www.stats.ox.ac.uk/pub/RWin/bin/windows/contrib/3.5:
      URL 'http://www.stats.ox.ac.uk/pub/RWin/bin/windows/contrib/3.5/PACKAGES'를 열 수 없습니다"
    
    package 'dplyr' successfully unpacked and MD5 sums checked
    
    The downloaded binary packages are in
    	C:\Users\eunee\AppData\Local\Temp\Rtmpeu61lL\downloaded_packages


​    
​    Attaching package: 'dplyr'
​    
​    The following objects are masked from 'package:stats':
​    
​        filter, lag
​    
​    The following objects are masked from 'package:base':
​    
        intersect, setdiff, setequal, union


​    

17차 조사에는 부가조사와 개인별 조사가 있다. 부가조사와 개인별 조사에서 변수를 가져올 것이기 때문에, ```read.spss```명령어를 활용하여 불러들인다.   

```dt17_a``` : 부가조사   
```dt17_p``` : 개인별 조사


```R
dt17_a <- read.spss("C:/Users/eunee/khu_sda_project/data/klips17a.sav",
                   use.value.labels = FALSE, to.data.frame = TRUE)
dt17_p <- read.spss("C:/Users/eunee/khu_sda_project/data/klips17p_i.sav",
                   use.value.labels = FALSE, to.data.frame = TRUE)
```

###  ```rdt_a``` : 부가조사

부가조사 데이터에서 pid, 출근 시간(시, 분), 퇴근 시간(시, 분) 자료만 선택해 ```rdt_a```로 만든다.   
    
```pid``` : pid    
```wstart_h``` : 출근 시간(시)   
```wstart_m``` : 출근 시간(분)
```wend_h``` : 퇴근 시간(시)   
```wend_m``` : 퇴근 시간(분)


```R
rdt_a <- subset(dt17_a,
                select = c(pid, a177102, a177103, a177105, a177106))
names(rdt_a) <- c("pid", "wstart_h", "wstart_m", "wend_h", "wend_m")
```

### ```rdt_p``` : 개인별 조사

개인별 조사 데이터에서 pid, 출근 시간(시, 분), 퇴근 시간(시, 분) 자료만 선택해 ```rdt_a```로 만든다.   
    
```pid``` : pid     
```wspacesat``` : 전반적 일자리 만족도   
```gend``` : 성별   
```birth``` : 출생년도   
```comp_type``` : 기업형태   
```wage``` : 임금   
```jobty``` : 종사상 지위   
```educ``` : 교육수준   


```R
rdt_p <- subset(dt17_p,
                select = c(pid, p174321, p170101, p170104, p170401, p171642, p170314, p170110))
names(rdt_p) <- c("pid", "wspacesat", "gend", "birth", "comp_type", "wage", "jobty", "educ")
```

### ```rdt_i``` : ```rdt_a```, ```rdt_p``` 에 대한 merge

```pid``` 기준으로 merge하여 ```rdt_i```로 만든다.


```R
rdt_i <- merge(rdt_a, rdt_p, by = "pid")
```

단, merge한 ```rdt_i```는 아래처럼 ```NA```가 존재하는데, ```colSums```와 ```is.na``` 명령어를 통해 ```NA```의  개수를 파악할 수 있다.


```R
str(rdt_i)
```

    'data.frame':	11679 obs. of  12 variables:
     $ pid      : num  101 102 203 401 402 ...
     $ wstart_h : num  7 8 NA 9 13 9 NA NA 4 10 ...
     $ wstart_m : num  0 30 NA 0 0 0 NA NA 0 30 ...
     $ wend_h   : num  18 18 NA 18 15 17 NA NA 14 22 ...
     $ wend_m   : num  0 0 NA 0 0 0 NA NA 0 30 ...
     $ wspacesat: num  3 3 NA 2 2 2 NA NA 3 3 ...
     $ gend     : num  2 1 2 1 2 2 2 2 1 2 ...
     $ birth    : num  1941 1968 1979 1970 1969 ...
     $ comp_type: num  6 1 NA 3 NA 4 NA NA 1 1 ...
     $ wage     : num  25 250 NA 500 NA 300 NA NA 70 130 ...
     $ jobty    : num  3 3 NA 1 4 1 NA NA 1 1 ...
     $ educ     : num  4 5 6 7 6 8 7 2 3 4 ...



```R
colSums(is.na(rdt_i))
```


<dl class=dl-horizontal>
	<dt>pid</dt>
		<dd>0</dd>
	<dt>wstart_h</dt>
		<dd>4349</dd>
	<dt>wstart_m</dt>
		<dd>4349</dd>
	<dt>wend_h</dt>
		<dd>4349</dd>
	<dt>wend_m</dt>
		<dd>4349</dd>
	<dt>wspacesat</dt>
		<dd>4349</dd>
	<dt>gend</dt>
		<dd>0</dd>
	<dt>birth</dt>
		<dd>0</dd>
	<dt>comp_type</dt>
		<dd>6505</dd>
	<dt>wage</dt>
		<dd>6515</dd>
	<dt>jobty</dt>
		<dd>4349</dd>
	<dt>educ</dt>
		<dd>0</dd>
</dl>



```rdt_i```는 ```pid```를 ```key```로 한 inner join이기 때문에 ```pid```의 ```NA```는 0이다.   
설명변수인 ```wstart_h```와 ```wstart_m```에 해당하지 않거나(```NA```), 모름/무응답(```-1```)인 사례는 연구관심 밖이므로 가장 먼저 제거가 필요하다.

### 설명변수(출근시간) 만들기


```R
nrdt_i <- rdt_i
nrdt_i$nwstart_h <- ifelse(nrdt_i$wstart_h == -1, NA, nrdt_i$wstart_h)
nrdt_i$nwstart_m <- ifelse(nrdt_i$wstart_m == -1, NA, nrdt_i$wstart_m)
```

분석 가능한 설명변수를 추리기 위해  ```nrdt_i```의 ```wstart_h```가 ```-1```(모름/무응답) 인 경우 ```NA```, 그렇지 않으면 원래 값을 주도록 했다. 이후 ```complete.cases```명령어로  ```NA```를 제거하면 ```NA```와 모름/무응답을 함께 제거할 수 있다. (모름/무응답에 해당하는 ```-1```도 위에서 ```NA```로 만들었으므로)


```R
nrdt_i <- nrdt_i[complete.cases(nrdt_i[ , c("nwstart_h", "nwstart_m")]), ]
```


```R
colSums(is.na(nrdt_i))
```


<dl class=dl-horizontal>
	<dt>pid</dt>
		<dd>0</dd>
	<dt>wstart_h</dt>
		<dd>0</dd>
	<dt>wstart_m</dt>
		<dd>0</dd>
	<dt>wend_h</dt>
		<dd>0</dd>
	<dt>wend_m</dt>
		<dd>0</dd>
	<dt>wspacesat</dt>
		<dd>0</dd>
	<dt>gend</dt>
		<dd>0</dd>
	<dt>birth</dt>
		<dd>0</dd>
	<dt>comp_type</dt>
		<dd>2128</dd>
	<dt>wage</dt>
		<dd>2134</dd>
	<dt>jobty</dt>
		<dd>0</dd>
	<dt>educ</dt>
		<dd>0</dd>
	<dt>nwstart_h</dt>
		<dd>0</dd>
	<dt>nwstart_m</dt>
		<dd>0</dd>
</dl>



분석의 편의를 위해 시간을 '시' 단위로 통일한다. 예를 들어 오전 10시 30분은 10.5로 표시될 것이다.


```R
nrdt_i$nwstart_m <- as.numeric(nrdt_i$nwstart_m)
nrdt_i$wstart_m <- (nrdt_i$nwstart_h)*60 + (nrdt_i$nwstart_m)
nrdt_i$wstart_h <- (nrdt_i$wstart_m)/60
nrdt_i$wstart <- nrdt_i$wstart_h
```


```R
summary(nrdt_i$wstart)
```


       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
      0.000   8.000   9.000   9.071   9.000  23.250 


### 종속변수(전반적일자리만족도) 클리닝


```R
summary(nrdt_i$wspacesat)
```


       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
     -1.000   2.000   3.000   2.698   3.000   5.000 


모름/무응답 을 ```NA```로 함께 처리해야 한다.


```R
nrdt_i$nwspacesat <- ifelse(nrdt_i$wspacesat == -1, NA, nrdt_i$wspacesat)
```

```-1```인 경우 ```NA```로, 그렇지 않을 경우 ```nrdt_i$wspacesat```을 그대로 가지며, 설명변수를 클리닝한 것과 같은 방식으로 클리닝한다.


```R
nrdt_i <- nrdt_i[complete.cases(nrdt_i[ , "wspacesat"]), ]
```

단, 분석의 편의를 위해 1을 매우 불만족, 5를 매우 만족으로 할 수 있게 역코딩을 실시한다.


```R
nrdt_i$nwspacesat_c <- c(5,4,3,2,1)[match(nrdt_i$wspacesat, c(1,2,3,4,5))]
```


```R
summary(nrdt_i$nwspacesat_c)
```


       Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
      1.000   3.000   3.000   3.283   4.000   5.000      36 


### 혼동변수 클리닝
#### 연령
출생년도 ```birth```를 이용해 연령 ```age``` 혼동변수를 만들어 관리한다.


```R
nrdt_i <- nrdt_i %>% 
  mutate(age = 2014-birth)
```

조사년도인 2014년에서 출생년도를 빼서 조사 당시의 연령변수를 만든다.

#### 종사상 지위
(1) 상용직   
(2) 임시직   
(3) 일용직   
(4) 고용주/자영업자   
(5) 무급가족종사자

중 상용직, 임시직, 일용직만 가져와 범주화한다.


```R
nrdt_i$jobty_c <- c(0,1,2)[match(nrdt_i$jobty, c(1,2,3))]
nrdt_i$jobty_c <- factor(nrdt_i$jobty_c)
```

0 : 상용직   
1 : 임시직   
2 : 일용직  

#### 기업형태
(1) : 민간회사 또는 개인 사업체  
(2) : 외국인 회사   
(3) : 정부투자기관, 정부출연기관, 공사합동기업   
(4) : 법인단체    
(5) : 정부기관   
(6) : 나는 특정회사나 사업체에 소속되어 있지 않다   
(7) : 시민단체, 종교단체  
(8) : 기타    


```R
nrdt_i$comp_type <- ifelse(nrdt_i$comp_type == -1, NA, nrdt_i$comp_type)
nrdt_i$comp_type <- ifelse(nrdt_i$comp_type == 8, NA, nrdt_i$comp_type)
nrdt_i$comp_type <- ifelse(nrdt_i$comp_type == 7, NA, nrdt_i$comp_type)
nrdt_i$comp_type <- ifelse(nrdt_i$comp_type == 6, NA, nrdt_i$comp_type)
```

분석에 불필요한 범주는 ```NA```처리한다.


```R
nrdt_i$comp_type <- c(1, 1 ,2, 1, 2)[match(nrdt_i$comp_type, c(1, 2, 3, 4, 5))]
```

이후   
0 : 민간회사, 개인 사업체, 외국인 회사, 법인단체     
1 : 정부투자기관, 정부출연기관, 공사합동기업, 정부기관   
의 두 범주로 기업혀태를 범주화한다.


```R
nrdt_i$comp_type_c <- cut(nrdt_i$comp_type, breaks = c(1,2,3),
                          right = FALSE,
                          labels = c("0", "1"))
```

#### 성별


```R
nrdt_i$gend <- ifelse(nrdt_i$gend == -1, NA, nrdt_i$gend)
```


```R
nrdt_i$gend_c <- cut(nrdt_i$gend, breaks = c(1,2,3),
                     right = FALSE,
                     labels = c("0", "1"))
```


```R
summary(nrdt_i$gend_c)
```


<dl class=dl-horizontal>
	<dt>0</dt>
		<dd>4319</dd>
	<dt>1</dt>
		<dd>2914</dd>
</dl>



0 : 남성   
1 : 여성

성별 범주 중 ```-1```은 모름/무응답이므로 ```NA```로 만든다.

#### 학력


```R
nrdt_i$educ <- c(0,0,0,0,1,2,3,4,4)[match(nrdt_i$educ, c(1,2,3,4,5,6,7,8,9))]
nrdt_i$educ <- factor(nrdt_i$educ)
```


```R
summary(nrdt_i$educ)
```


<dl class=dl-horizontal>
	<dt>0</dt>
		<dd>1437</dd>
	<dt>1</dt>
		<dd>2550</dd>
	<dt>2</dt>
		<dd>1148</dd>
	<dt>3</dt>
		<dd>1743</dd>
	<dt>4</dt>
		<dd>355</dd>
</dl>



0 : 미취학, 무학, 초등학교, 중학교   
1 : 고등학교   
2 : 2년제 대학, 전문대학   
3 : 4년제 대학
4: 대학원 석사, 대학원 박사   
    
학력변수를 생성한 뒤 위처럼 재범주화하였다.

#### 임금
임금은 로그를 취하여 만든다.


```R
nrdt_i$wage <- ifelse(nrdt_i$wage<0, NA, nrdt_i$wage)
nrdt_i$lnwage <- log(nrdt_i$wage+0.01)
```


```R
summary(nrdt_i$lnwage)
```


       Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
     -4.605   4.942   5.298   5.278   5.704   8.294    2134 


```summary```로 보면 ```lnwage```에 음수 값이 존재하는데, 이는 outlier이므로 제거한다. 


```R
nrdt_i$lnwage <- ifelse(nrdt_i$lnwage <=0, NA, nrdt_i$lnwage)
```

#### 근로시간
출근시간과 퇴근시간의 차이로 근로시간을 계산한다.


```R
nrdt_i$wend_h <- ifelse(nrdt_i$wend_h == -1, NA, nrdt_i$wend_h)
nrdt_i$wend_m <- ifelse(nrdt_i$wend_m == -1, NA, nrdt_i$wend_m)
```

퇴근 시간과 분에서 ```-1```은 모름/무응답이므로 ```NA```로 만든다.


```R
nrdt_i$nwstart_t <- (strtoi(nrdt_i$nwstart_m)+(strtoi(nrdt_i$nwstart_h)*60))
nrdt_i$wend_t <- (strtoi(nrdt_i$wend_m)+(strtoi(nrdt_i$wend_h)*60))
```

계산의 편의를 위해 출근시간과 퇴근시간을 분 단위로 표시한다.


```R
nrdt_i$w_time_m <- nrdt_i$wend_t - nrdt_i$nwstart_t
```

출근시간과 퇴근시간의 차이로 분 단위의 근무시간 변수를 만든다.


```R
nrdt_i$w_time_m <- ifelse(nrdt_i$w_time_m <= 0, nrdt_i$w_time_m+ (24*60), nrdt_i$w_time_m)
```

단, 출퇴근 시간의 차이가 음수인 경우 24시간*60분을 더해 보정한다. 예를 들어 23시 출근, 8시 퇴근인 경우


```R
nrdt_i$w_time <- nrdt_i$w_time_m/60
```

분 단위로 만들어진 근무시간을 다시 시간 단위로 변환한다.

## 💡 최종 데이터 셋

정리한 자료를 한데 모으고 이름을 붙여준다.


```R
nrdt_attach <- data.frame(nrdt_i$pid, nrdt_i$wstart, nrdt_i$nwspacesat_c, nrdt_i$w_time, nrdt_i$gend_c, nrdt_i$age, 
                          nrdt_i$comp_type_c, nrdt_i$educ, nrdt_i$jobty_c, nrdt_i$lnwage)
```


```R
nrdt_names <- c("pid", "wstart", "wspacesat", "wtime", "gend", "age", "comp_type", "educ", "jobty", "lnwage")
names(nrdt_attach) <- nrdt_names
```

최종적으로 ```NA```를 제거한다.


```R
nrdt_attach <- nrdt_attach[complete.cases(nrdt_attach),]
```


```R
summary(nrdt_attach)
```


          pid               wstart         wspacesat         wtime         gend    
     Min.   :     102   Min.   : 0.000   Min.   :1.000   Min.   : 0.3333   0:2901  
     1st Qu.:  193278   1st Qu.: 8.000   1st Qu.:3.000   1st Qu.: 9.0000   1:1999  
     Median :  353304   Median : 9.000   Median :3.000   Median : 9.0000           
     Mean   : 1870533   Mean   : 9.046   Mean   :3.309   Mean   : 9.5761           
     3rd Qu.:  596777   3rd Qu.: 9.000   3rd Qu.:4.000   3rd Qu.:10.0000           
     Max.   :11014502   Max.   :23.000   Max.   :5.000   Max.   :24.0000           
          age       comp_type educ     jobty        lnwage     
     Min.   :18.0   0:4236    0: 666   0:3669   Min.   :2.304  
     1st Qu.:35.0   1: 664    1:1644   1: 813   1st Qu.:4.942  
     Median :42.0             2: 931   2: 418   Median :5.298  
     Mean   :43.6             3:1375            Mean   :5.296  
     3rd Qu.:52.0             4: 284            3rd Qu.:5.704  
     Max.   :80.0                               Max.   :8.294  


```NA``` 없는 예쁜 자료가 만들어졌다.


```R
save.image(file="C:/Users/eunee/khu_sda_project/data_cleaning.RData")
```

## 👍 최종 데이터 셋 변수 정리
- ```pid``` : pid
- ```wstart``` : 출근시간 (연속형)   
- ```wspacesat``` : 전반적일자리만족도 (연속형)  
  - 1 : 매우 불만족
  - 2 : 불만족
  - 3 : 보통
  - 4 : 만족
  - 5 : 매우 만족
- ```wtime``` : 주당 평균 근로시간 (연속형)
- ```gend``` : 성별 (이분형)  
  - 0 : 남성
  - 1 : 여성
- ```age``` : 연령 (연속형)
- ```comp_type``` : 기업형태 (범주형) 
  - 0 : 민간 (민간회사, 개인 사업체, 외국인 회사, 법인단체)
  - 1 : 비민간 (정부투자기관, 정부출연기관, 공사합동기업, 정부기관)
- ```educ``` : 학력 (범주형)  
  - 0 : 미취학, 무학, 초등학교, 중학교
  - 1 : 고등학교
  - 2 : 2년제 대학, 전문대학
  - 3 : 4년제 대학
  3대학원 석사, 대학원 박사
- ```jobty``` : 종사상지위 (범주형)  
  - 0 : 상용직
  - 1 : 임시직
  - 2 : 일용직
- ```lnwage``` : 로그 임금 (연속형) - 단, 0 이상

## 📝 생각
- 근로시간을 단지 출근시간과 퇴근시간의 차로 설명할 수 있을까?  

   근로시간에 관한 직접적인 자료가 데이터 셋에 없어서 출근시간과 퇴근시간의 차이를 이용해 임의로 근로시간 변수를 만들었다. 단순 출퇴근 시간의 차이로만 계산하였기 때문에 휴게시간을 제외한 순수 근로시간을 고려할 수 없었다.  
   업무를 준비하기 위해 일찍 회사에 도착하는 경우가 많은데, 업무를 준비하는 시간 역시 근로시간에 포함하여 계산된다는 한계가 있다.

- 시간을 소수점 단위로 나타내는 것이 직관적이지 않다. R에서 시간을 60진법으로 다루면서 동시에 통계 분석을 수행할 수는 없을까?
