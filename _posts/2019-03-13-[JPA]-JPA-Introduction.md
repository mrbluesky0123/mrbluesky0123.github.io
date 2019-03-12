---
layout: post
title:  "[JPA] ORM 이전의 PERSISTENCE"
date:   2019-03-13 00:30:00 +0900
categories: JPA
---
_* JPA 카테고리의 내용은 "Java ORM 표준 JPA 프로그래밍 -김영한 저" 책을 보고 공부 한 내용을 정리한 것이다._

- ORM 이전의 persistence
    - SQL로 DB를 조회하여 객체에 맵핑 → 객체는 데이터의 전달 목적으로 사용
    - 객체 모델링이 정교해질수록 객체를 DB에 저장, 조회하기 복잡해짐 → 이 차이를 극복하기 위하여 더 많은 SQL을 작성
    - 지루하고 반복적인 CRUD SQL 작성 → 생산성 저하
- SQL을 직접 다룰 때의 문제점
    1. 계속된 반복 작업
    SQL 작성 → 데이터 조회 → 객체 맵핑
    Insert SQL 작성 → 객체의 값을 등록 → API를 이용한 SQL 실행
    2. 패러다임의 불일치: 객체와 관계형 데이터베이스는 지향하는 목적이 서로 다름(ex. 상속 - DB 테이블에는 상속 구조가 없음)
    객체의 속성이 바뀔 때마다 SQL도 바뀌어야 함
    객체가 또 다른 객체를 참조할 경우 SQL에서는 더욱더 복잡한 join을 이용해야함

        class Member{
        	
        	private String mbrId;
        	private String name;
        
        	private Team team;  // Member가 team을 참조
        	// private Long teamId;  // 테이블 구조에 맞춘 객체 모델링
        	
        }

    (객체는 team 이라는 객체의 참조를 통하여 접근이 가능하지만 sql을 이용하여 테이블에 맞추어 이를 해결할 경우 teamId를 foreign key로 가지고 있어야 한다.)

    3. 객체 그래프 탐색 시, SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해짐. 

        ```
        	Member -------------- Team 
            |
          Order --------------- OrderItem -----------Item
        ```

    와 같이 객체의 연관관계가 설정이 되어 있을 때 Member가 소속된 팀을 조회할 때에는 

        Team team = member.getTeam();

    Member가 주문한 Item을 조회할 때에는

        Item item = member.getOrder().getOrderItem();

    와 같이 member 객체 하나로 자유로운 객체 탐색이 가능하다. 하지만 SQL을 통해 이를 구현할 경우에는 이와 같은 탐색이 제한된다. 가령 member를 통해 아래와 같은 SQL로 소속팀을 구할 수 있다.  

        SELECT M.*, T.*
          FROM MBR_MASTER M
          JOIN TEAM_MASTER T ON M.TEAM_ID = T.TEAM_ID

    하지만 member가 주문한 Item을 조회하려면 SQL을 통하여 다시 한 번 ORDER 테이블과의 JOIN이 필요하다.

- 정리: 객체 모델과 관계형 데이터베이스 모델은 서로 지향하는 바가 다르기 때문에 그 차이의 극복에 개발자가 많은 시간과 노력을 쏟아야 했다. 패러다임의 불일치로 인한 제한 사항 뿐만 아니라 수 많은 반복 작업이 수반되어야 했다. JPA는 패러다임의 불일치를 해결하고 정교한 객체 모델링을 유지하도록 해준다.