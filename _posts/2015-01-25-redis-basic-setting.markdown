---
layout: post
title:  "[Redis] 기초 설정"
date:   2015-01-25 00:00:00
categories: posts redis
---

## info 명령의 그룹 정보
- Server: 실행 중인 레디스 서버의 실행 정보
- Clients: 서버와 연결된 클라이언트들의 통계 정보
- Memory: 메모리 사용 정보 및 메모리 통계 정보
- Persistence: 영구 저장소와 관련된 저장 정보
- Stats: 명령 수행과 저장된 키에 대한 통계 정보
- Replication: 현재 동작 중인 복제 정보
- CPU:  CPU 사용률 통계 정보
- Keyspace: 데이터베이스별로 저장된 키 정보

## info 명령 결과 정보
- redis_version: 레디스 서버 버전 정보
- arch_bits: 레디스 서버의 아키텍쳐 비트
- process_id: 레디스 서버의 시스템 프로세스 ID
- connected_clients: 현재 연결된 클라이언트 커넥션 수
- connected_slaves: 복제를 위해 연결된 슬레이브 수
- used_memory: 레디스 서버가 사용하는 메모리 양(byte)
- used_memory_human: used_memory를 보기 쉬운 단위로 출력
- used_memory_peak: 레디스 서버가 사용한 최대 메모리 크기(byte)
- used_memory_peak_human: used_memory_peak를 보기 쉬운 단위로 출력
- mem_fragmentation_ratio: 연속되지 않은 공간에 저장된 비율
- role:master: 마스터-슬레이브 동작 모드, 단일 모드의 경우 마스터로 표시

## 메모리 사용량
`Clients connected \<0 slaves\>, 1179897 bytes in use`

- \[1179896 bytes in use\]: 레디스가 사용하고 있는 메모리 용량. 서버의 가용 메모리보다 더 큰 메모리를 할당 받으면 메모리와 하드디스크와의 스왑이 발생. 성능 저하(1/20~ 1/1000)가 발생할 수 있음

## 버전 관리법
`X.Y.Z`

- X: 메이저 버전: 커다란 기능 변화 발생시 변경
- Y: 마이너 버전: 작은 기능 추가시 변경(짝-안정화, 홀-비안정화)
- Z: 패치레벨: 마이너 버전에 대한 패치 수행 횟수

## 알고리즘의 정렬 안정성
같은 내용을 가지는 키값의 배열이 정렬 후에도 상대적 순서가 그대로 유지되는지 여부

## redis-benchmark option

- -h \<hostname\>
- -p \<port\>
- -s \<socket\>
- -c \<clients\> 테스트를 위한 가상 클라이언트의 동시 접속 수
- -n \<requests\> 각 명령의 테스트 횟수
- -d \<size\> 테스트에 사용할 데이터 크기
- -k \<boolean\> 테스트를 위한 가상 클라이언트의 접속 유지 여부  1:접속 유지, 0:접속 유지하지 않음
- -r \<keyspacelen\> 테스트에 사용한 랜덤 키의 범위
- -P \<numreq\> 파이프라인 명령을 사용한 테스트와 파이프라인당 요청할 명령의 갯수  0:파이프라인 미사용
- --csv 테스트 결과를 csv포맷으로 출력
- -l 브레이크를 걸기전까기 계속 수행
- -t \<tests\> 쉼표로 구분된 테스트 명령의 목록

---

출처: `정경석, "이것이 레디스다", 한빛미디어, 2013`
