---
layout: post
title:  "[Eclipse] Eclipse 성능 최적화"
date:   2015-02-22 00:00:00
categories: posts eclipse
---

이 포스트는 [SLiPP 스터디](http://www.slipp.net/wiki/pages/viewpage.action?pageId=5177633)의 위키 페이지에 정리된 내용입니다.

---

# eclipse.ini 설정 
        -Xverify:none           유효성 검사 생략  
        -XX:+UseParallelGC      병렬 처리  
        -XX:-UseConcMarkSweepGC 컴파일러의 소수점 최적화 기능 활성화   
        -XX:PermSize=64M        Perment generation size  
        -XX:MaxPermSize=512M    Max permanent generation size  
        -XX:MaxNewSize=512M     New generation  
        -XX:NewSize=128M        Max new generation  
        -Xms1024m               이클립스 heap memory size min  
        -Xmx1024m               이클립스 heap memory size max  
        
        
# 소스 자동 폴딩 해제
        Java > Editor > Folding > Enable folding

# 코드 자동완성기능 해제
        Java > Editor > Content Assist > Enable auto activation

# Spelling 체크 설정 해제
        General > Editors > Text Editors >  Spelling > Enable spell checking

# Validation 설정 해제
        validation

# 불필요한 플러그인 삭제
        Install/Update > Uninstall or update

# 이클립스 구동 속도 개선
        General > Startup and Shutdown > Plug-ins activated on startup

# 자동 업데이트 해제
        Intall/Update > Automatic Updates

---

출처 : [SLiPP 스터디, http://www.slipp.net/wiki/display/SLS/Home](http://www.slipp.net/wiki/pages/viewpage.action?pageId=5177633)