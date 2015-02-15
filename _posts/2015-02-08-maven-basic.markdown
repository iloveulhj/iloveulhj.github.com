---
layout: post
title:  "[Maven] 기초"
date:   2015-02-08 00:00:00
categories: jekyll update
---

#메이븐 설정 파일
  
- settings.xml: 첫째는 메이븐 빌드 툴과 관련한 설정 파일  
- pom.xml: 메이븐 기반 프로젝트에서 사용하는 설정 파일  
  
---

#settings.xml 설정파일  
  
MAVEN_HOME/conf/settings.xml:  모든 사용자에 동일한 설정을 하기 위한 용도  
USER_HOME/.m2/settings.xml: 사용자 별로 다르게 설정  
로컬 저장소: 개발자PC에 의존관계에 있는 라이브러리, 플러그인을 저장하는 디렉토리  
USER_HOME/.m2/repository: `<settings>`의 `<localRepository>`로 설정할 수 있다   
  

---

#pom.xml 설정파일
**Project Object Model**  
  
- pom.xml파일이 아닌 다른 설정 파일을 사용하고자 한다면 --file옵션을 사용  
  
**프로젝트 기본 정보**  
  
- 프로젝트 이름  
- 프로젝트 URL  
- 프로젝트 참여자  
- 라이센스  
  
**빌드 설정**  
  
- 소스/ 테스트 소스 디렉토리  
- 리소스(자원) 디렉토리  
- 플러그인  
- 리포팅(문서화)  
  
**프로젝트 관계 설정**  
  
- groupId, artifactId, version  
- 모듈  
- 상수  
- 의존 라이브러리 관리  
  
**빌드 환경**  
  
- 빌드할 환경에 따른 정보  
- 프로파일  
    groupId: 프로젝트를 생성하는 조직의 고유 아이디를 결정, 일반적으로 도메인 이름을 사용   
    artifactId:  프로젝트를 식별하는 유일한 아이디를 의미, groupId+artifactId는 유일한 값이 되어야 한다   
    packaging: 프로젝트를 어떤 형태로 패키징할지 결정, jar, war, ear, pom   
    version: 프로젝트의 현재 버전, 프로젝트 개발 중에는 SNAPSHOP을 접미사로 사용  
    outputDirectory: sourceDirectory의 소스를 컴파일한 결과물이 위치하는 디렉토리, 기본값은 target/classes   
    resources: 서비스에 사용되는 자원을 관리하는 디렉토리, 기본 값은  src/main/resources  

---

#라이프사이클
**기본 라이프사이클**  
  
- compile: 소스코드를 컴파일한다  
- test: JUnit, TestNG와 같은 단위 테스트 프레임워크로 단위 테스트  
- package: 단위 테스트가 성공하면 <packaging/>의 엘리먼트 값에 따라 압축한다  
- install: 로컬 저장소에 압축한 파일을 배포  
- deploy: 원격 저장소에 압축한 파일을 배포  
  
**clean 라이프사이클**
  
- clean 페이즈를 이용해 실행  
- 메이븐 빌드를 통하여 생성된 모든 산출물을 삭제  
  
**site 라이프사이클**  
  
- site와 site-deploy 페이즈를 이용하여 실행  
- 기본 설정, 플러그인 설정에 따라 target/site 디렉토리에 문서 사이트를 생성  
  
---  

#메이븐 플러그인  
  
`<build>``<plugins>``<plugin>` 엘리먼트 아래 사용하고자 하는 플러그인의 groupId, artifactId, version을 설정하면 된다  
  
- 실행 명령어: mvn groupId:artifactId:version:goal  
ex)  mvn org.apache.maven.plugins:maven-compiler-plugin:2.1:compile  
- version을 생략하면 로컬저장소에 있는 최신 버전의 플러그인이 실행(groupId:artifactId:goal)  
-  artifactId가 'maven-$name-plugin'과 '$name-maven-plugin' 규칙을 따른다면  
groupId:$name:goal 형식으로 실행할 수 있다  
ex) mvn org.apache.maven.plugins:compiler:compile  
- settings.xml에 `<pluginGroups>``<pluginGroup>`를 통해 관리하면 $name:goal 형식으로 실행 가능 
ex) compiler:compile  

출처: `박재성, "자바 세상의 빌드를 이끄는 메이븐", 한빛미디어, 2011` 
