---
title: "cmd Oracle에서 한글이 나오지 않거나 깨질 때"
excerpt: "cmd에서 Oracle을 사용할 때 한글이 물음표나 '옜'으로 나온다. NLS_LANG, chcp를 활용하여 인코딩 형식을 통일시켜 해결해보자"
tags: [Windows, cmd, Oracle, NLS]
---
* toc
{:toc}

## 1. 원인
환경: VMWare Workstation 15 Player, Windows 10, Oracle 11g XE, cmd


cmd에서 한글로 된 데이터를 입력하거나 조회하면 물음표나 빈 칸으로 표시된다. 우선 급한대로 Eclipse에 Oracle을 연결해 사용했다. 그렇다는 것은 UTF-8 인코딩 형식의 데이터는 잘 처리하는 것인데... 한 달 후 결국 이 [글](https://m.blog.naver.com/PostView.nhn?blogId=tyboss&logNo=70036575256){:target="_blank"}을 읽고 해결은 했다.


원인은 기대하는 데이터 인코딩 형식이 달라 엉뚱한 형식으로 인코딩하는 것이었다. 가령 나는 cmd에서 MS949 형식의 데이터를 입력하는데 데이터베이스는 입력될 데이터의 형식을 UTF-8으로 알고 있다. 이 때 데이터베이스의 데이터 인코딩 형식이 UTF-8이라면 `MS949 → UTF-8` 변환을 해야하는데 `UTF-8 → UTF-8` 변환을 하게 된다. MS949 데이터를 UTF-8로 간주하여 변환했기에 내가 입력한 데이터는 엉뚱한 형식으로 변환된다.


![how characters are encoded via application to database](/rsc/img/2020-08-29/cmd-to-database-encoding.png "how characters are encoded via application to database")


따라서 문제를 해결하기 위해서 변환 시의 인코딩 형식을 입출력하는 데이터의 인코딩 형식에 맞추거나 입출력하는 데이터의 인코딩 형식를 변환 시의 인코딩 형식에 맞춰야 한다.

## 2. Oracle NLS Parameter 확인
NLS는 National Language Support의 약자로 특정 언어나 형식으로 데이터를 처리할 수 있게 해주는 Oracle의 언어 지원이다. 언어뿐만 아니라 통화, 시간 등 각 국가나 지역의 관습에 맞는 형식을 지원한다. 현재 사용 중인 데이터베이스의 NLS 설정 값은 `nls_database_parameter` 테이블에서 확인할 수 있다. 찾아볼 칼럼은 `NLS_LANGUAGE`로 현재 데이터베이스가 사용하는 문자 인코딩 형식을 의미한다. 쿼리는 다음과 같다

```
SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_LANGUAGE';
```

나는 기본 값이 'AMERICAN_AMERICA.AL32UTF8'이었다. 되도록 다양한 언어를 지원하는 국제 표준인 UTF-8 형식을 사용하자.

> 주의: Oracle의 설정을 변경할 때는 의도치 않게 데이터가 변형될 수 있으므로 중요한 데이터는 항상 백업을 준비한다.

## 3. NLS_LANG 확인
이제 클라이언트의 운영체제에 설정된 클라이언트와 Oracle 간의 인코딩 형식을 확인할 차례다. 방금 인코딩 형식을 확인한 것이 아닌가? 아니다. 위에서 확인한 것은 Oracle 데이터베이스 내에서 데이터를 처리할 때 사용하는 인코딩 형식이다. 지금 확인하려는 것은 Oracle이 기대하는 클라이언트 측의 데이터 인코딩 형식이다. 즉 Oracle은 클라이언트가 이 형식으로 데이터를 인코딩해서 전송하거나 수신할 것으로 기대한다는 것이다.


실행`윈도우 키 + R`에서 `regedit`을 입력하여 레지스트리 편집기를 실행한다. `HKEY_LOCAL_MACHINE\SOFTWARE` 아래 ORACLE 경로에 있는 레지스트리에서 `NLS_LANG`을 찾는다. 어떤 경로에 있을 지는 환경마다 다르니 레지스트리 검색 또는 구글의 도움을 받아 찾는다. 찾은 `NLS_LANG`의 값을 cmd 또는 클라이언트 측 어플리케이션에서 사용하는 인코딩 형식으로 수정한다. 내 경우는 'NLS_LANG'의 초기 값이 `AMERICAN_AMERICA.WE8MSWIN1252`였다. 따라서 `WE8MSWIN1252`를 `KO16MSWIN949`으로 수정했다.


![how to change the value of nls_lang](/rsc/img/2020-08-29/nls_lang.png "how to change the value of nls_lang")


이제 cmd에서 Oracle을 실행하여 한글이 나오는지 확인한다. 아직 한글이 나오지 않는다면 다음 단계로 넘어간다.

## 4. cmd chcp 확인
cmd의 인코딩 형식을 확인한다. cmd를 실행하고 chcp를 입력한다. chcp는 Change Code Page의 약자이다. cmd에서 입출력하는 문자의 인코딩 형식을 정할 수 있다. 나의 초기 Code Page는 United States의 `437`이었다. 내가 설정한 `NLS_LANG`의 값 `KO16MSWIN949`의 인코딩 형식인 `MS949`를 지원하는 Code Page는 `949`이다. 따라서 cmd에서 `chcp 949`을 입력하여 Code Page를 변경한다. 


![how to change chcp](/rsc/img/2020-08-29/chcp.png "chcp")


이제 cmd에서 한글을 입출력할 수 있다.


*추신. 같은 원리로 `chcp 65001`, `NLS_LANG = AMERICAN_AMERICA.AL32UTF8` 등을 설정하여 UTF-8 형식으로 cmd에 데이터를 입출력하려 했으나 실패했다...*