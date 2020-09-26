---
title: "cmd Oracle에서 한글이 나오지 않거나 깨질 때"
excerpt: "cmd에서 Oracle을 사용할 때 한글이 물음표나 '옜'으로 나온다면 애플리케이션-운영체제-Oracle간 인코딩 형식이 일치하지 않다는 것이다."
tags: [Windows, cmd, Oracle, NLS]
---
<h2>cmd Oracle에서 한글이 나오지 않거나 깨질 때</h2>
<ol>
  <li><a href="#1">문제</a></li>
  <li><a href="#2">Oracle NLS Parameter 확인</a></li>
  <li><a href="#3">OS NLS_LANG 레지스트리 데이터 확인</a></li>
  <li><a href="#4">cmd chcp 확인</a></li>
</ol>
<h3 id="1">문제</h3>
<p>환경: VMWare Workstation 15 Player, Windows 10, Oracle 11g XE, Command Prompt</p>
<p>cmd에서 한글로 된 데이터를 입력하거나 조회하면 물음표나 빈 칸으로 표시된다. 해결해보려다 근본의 부재로 미루고 급한대로 Eclipse에 Oracle을 연결하여 사용했다. 그렇다는 것은 UTF-8 인코딩 형식의 데이터는 잘 처리한다는 것 같은데... 한 달만에 다시 해결하기 위해 처음부터 다시 시작했고 이 <a href="https://m.blog.naver.com/PostView.nhn?blogId=tyboss&logNo=70036575256" target="_blank">글</a>을 통해 해결은 했다.</p>
<p>문제는 기대하는 데이터 인코딩 형식이 다르면서 엉뚱한 형식으로 인코딩하는 것이었다. 가령 나는 cmd에서 MS949 형식의 데이터를 입력하는데 운영체제는 입력되는 데이터가 UTF-8이라고 알고 있다. 데이터베이스의 데이터 인코딩 형식이 UTF-8이라면 MS949 → UTF-8 변환을 해야하는데 UTF-8 → UTF-8 변환을 하게 되는 것이다. 따라서 내가 입력한 MS949 형식의 데이터는 다른 값을 표시하게 된다.</p>
<img href="../rsc/img/cmd-to-database-encoding.png" alt="how characters are encoded via application to database"/>
<h3 id="2">Oracle NLS Parameter 확인</h3>
<p>가장 먼저 Oracle을 확인한다. 응답을 받았다는 것은 요청 문제는 아니기 때문에 응답의 시발점인 Oracle부터 확인한다. NLS는 National Language Support의 약자로 특정 언어나 형식으로 데이터를 처리할 수 있게 해주는 Oracle의 언어 지원이다. 언어뿐만 아니라 통화, 시간 등 각 국가나 지역의 관습에 맞는 형식을 지원한다. 현재 사용 중인 데이터베이스의 NLS 설정 값은 'nls_database_parameter' 테이블에서 확인할 수 있다. 찾아볼 칼럼은 'NLS_LANGUAGE'로 현재 데이터베이스가 사용하는 문자 인코딩 형식을 의미한다. 쿼리는 다음과 같다. 
```SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_LANGUAGE'```나는 기본 값이 'AMERICAN_AMERICA.AL32UTF8'이었다.</p>
<h3 id="3">OS NLS_LANG 레지스트리 데이터 확인</h3>
<p>이제 클라이언트의 운영체제에 설정된 Oracle 인코딩 형식을 확인할 차례다. 방금 Oracle의 인코딩 형식을 확인한 것이 아닌가? 아니다. 위에서 확인한 것은 Oracle 데이터베이스에서 데이터를 처리할 때 사용할 인코딩 형식이다. 지금 확인하려는 것은 클라이언트가 기대하는 Oracle 데이터베이스의 인코딩 형식이다. 즉 클라이언트는 데이터베이스가 이러한 형식으로 데이터를 인코딩해서 전송할 것이라고 기대한다는 것이다(암호를 받는 측이 암호를 보낸 측에서 특정 암호책으로 암호화했을 것라고 기대하는 것과 같다). 지금 확인한 값이 위 값과 다르면 결과는 조작된다.</p>
<p>실행(윈도우 키 + R)에서 regedit을 입력하여 레지스트리 편집기를 실행한다. HKEY_LOCAL_MACHINE\SOFTWARE 아래 ORACLE 경로에 있는 레지스트리에서 'NLS_LANG'을 찾는다. 어떤 레지스트리에 있을 지는 환경마다 다르니 레지스트리 검색 또는 구글의 도움을 받아 찾는다. 찾은 'NLS_LANG'의 값이 앞서 찾은 'NLS_LANGUAGE'와 다르다면 'NLS_LANG'의 값을 'NLS_LANGUAGE'에 맞춰 수정한다. 내 경우는 'NLS_LANG'의 값이 'AMERICAN_AMERICA.WE8MSWIN1252'였다. 따라서 'WE8MSWIN1252'를 'AL32UTF8'으로 수정했다. 여기서 cmd에서 Oracle을 실행하여 한글이 나오는지 확인한다. 아직 한글이 나오지 않는다면 다음 단계로 넘어간다.</p>
<h3 id="4">cmd chcp 확인</h3>
<p>마지막으로 cmd의 인코딩 형식을 확인한다. cmd를 실행하고 chcp를 입력한다. chcp는 Change Code Page의 약자이다. ASCII의 범위를 넘는 문자값에 대해 인코딩할 형식을 정할 수 있다. 결과 값은 앞서 확인한 인코딩 형식을 지원해야 한다. 나의 초기 Code Page는 United States의 437이었다. 내가 설정한 'AL32UTF8'의 인코딩 형식인 'UTF-8'을 지원하는 Code Page는 65001이다. 따라서 cmd에서 'chcp 65001'을 입력하여 Code Page를 변경했다. 여기까지 하니 나는 한글을 볼 수 있었다. 만약 데이터베이스와 클라이언트를 'MSWIN949'로 설정했다면 'chcp 949'를 입력하면 될 것이다.</p>
