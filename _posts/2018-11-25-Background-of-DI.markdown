---
layout: post
title:  "Background of DI(Dependency Injection)"
date:   2018-11-25 22:00:00 +0900
categories: Framework
---
### (참고) [Dependency란?](http://www.nextree.co.kr/p6753/) 

# < 관심사의 분리 >

__User Domain (코드1)__
~~~java
packcage springbook.user.domain;

public class User{

    String id;
    String name;

    public String getId(){
        return id;
    }

    public String getName(){
        return name;
    }

}
~~~

__UserDao (코드2)__
~~~ java
packcage springbook.user.dao;

public class UserDao{

    public void add(User user) throws ClassNotFoundException, SQLException{

        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                                "jdbc:mysql://localhost/springbook", "aaa", "bbb"
                        );
        PreparedStatement ps = c.prpareStatement(
            "INSERT INTO USERS(id, name) VALUES (?,?)"
        );

        ps.seString(1, user.getId());
        ps.seString(2, user.getName());

        ps.executeUpdate();

        ps.close();
        c.close();

    }

    public User get(String id) throws ClassNotFoundException, SQLException{

        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                                "jdbc:mysql://localhost/springbook", "aaa", "bbb"
                        );
        PreparedStatement ps = c.prpareStatement()
            "SELECT * FROM USERS WHERE id = ?"
        );

        ps.seString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User u = new User();
        u.setId(rs.getString("id"));
        u.setName(rs.getString("name"));

        rs.close();
        ps.close();
        c.close();

        return u;

    }

}
~~~
코드 2는 DB 연결 및 종료와 관련된 기술적인 내용과 사용자의 데이터를 인출하고 변경하는 비즈니스 로직이 모두 포함되어있다. 하지만 DB를 MySQL에서 Oracle로 바꾸게 되거나 DB접속용 계정 및 암호를 변경하려면  또는 트랜잭션 기술을 다른 것으로 바뀌어 비즈니스 로직이 바뀌었을 경우 모든 DAO를 수정해야한다. 
이와 같은 여러 종류의 관심사(ex. 기술적인 관심, 비즈니스적인 관심 등)들을 적절하게 구분하고 따로 분리하는 작업을 해줘야 한다. 이를 __리팩토링(Refactoring)__ 이라고 한다.

> 리팩토링이란 기존의 코드를 외부의 동작방식에는 변화 없이 내부 구조를 변경하여 재구성하는 작업 또는 기술을 말한다. 리팩토링을 하면 코드 내부의 설계가 개선되어 코드를 이해하기 더 편해지고, 변화에 효율적으로 대응할 수 있기 때문에 유지보수가 용이해지고 견고하면서도 유연한 제품을 개발할 수 있다.

위의 코드2에서는 DB 커넥션을 가져오는 중복된 코드를 분리할 수 있다. 중복된 DB연결 코드를 getConnection() 이라는 독립적인 메소드로 만들 수 있다. 아래 코드는 그 일부이다.


~~~java
private Connection getConnection() throws ClassNotFoundExeception, SQLException {
    
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection(
            "jdbc:mysql://localhost/springbook", "aaa", "bbb"
    );
    return c;

}
~~~

  

# < 상속을 통한 확장 >

>(상황) UserDao가 N사, D사 두 회사에 납품되었다. 두 회사는 다른 종류의 DB를 사용하고 있기 때문에 DB 커넥션을 가져오는데 독자적인 방법을 적용하고싶어 한다. 그리고 이 방식은 종종 변경될 가능성이 높다.

UserDao를 추상 클래스로 만들고 getConnection()을 추상 메소드로 만든다. 그리고 이 추상 클래스를 N사와 D사에 배포한다. 두 회사는 이 클래스를 상속하여 각각 NUserDao, DUserDao라는 서브클래스를 만든다. 이렇게 하면 UserDao의 소스코드를 수정하지 않아도 getConnectio() 메소드만 원하는 방식으로 확장한 후에 UserDao의 기능을 함께 사용할 수 있다.

~~~java

public abstract class UserDao{

    public void add(User user) throws ClassNotFoundException, SQLException{

            Connection c = getConnection();
            // (생략)

    }

    public User get(String id) throws ClassNotFoundException, SQLException{

            Connection c = getConnection();
            // (생략)

    }

    // 추상 메소드
    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;

}

public class NUserDao extends UserDao{

    public Connection getConnection() throws ClassNotFoundException, SQLException{

        // N사의 DB connection 생성 코드

    }

}

public class DUserDao extends UserDao{

    public Connection getConnection() throws ClassNotFoundException, SQLException{

        // N사의 DB connection 생성 코드

    }

}

~~~

이러한 방식으로 UserDao의 수정 없이 DB연결 기능이 새롭게 정의된 클래스를 만들 수 있다. 
이렇게 슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 메소드 등으로 만든 뒤 서브 클래스에서 이런 메소드를 필요에 맞게 구현하여 사용하는 방법을 __템플릿 메소드 패턴(Template method pattern)__ 이라고 한다.

>템플릿 메소드 패턴이란 슈퍼클래스의 기능을 확장할 때 사용하는 가장 대표적인 방법이다. 변하지 않는 기능은 서브클래스에서 만들도록 한다. 슈퍼클래스에서는 미리 추상메소드 또는 오버라이드 가능한 메소드를 정의해두고 이를 활용해 코드의 기본 알고리즘을 담고 있는 템플릿 메소드를 만든다.

__즉 추상클래스를 이용하여 변화의 성격이 다른 것을 분리하였기 때문에 서로 영향을 주지 않은채로 각각 필요한 시점에 독립적으로 변경할 수 있다.__


#### [문제점]

상속은 간단하고 사용도 편리하지만, 이미 다른 목적을 위해 누군가 UserDao를 상속하고 있다면? (자바에서는 다중 상속이 허용되지 않는다.) 또는 슈퍼클래스(UserDao)의 구조가 바뀔 경우 - 예를 들어 `getConnection()` 함수의 이름이 `openConnection()`으로 바뀔 경우 - 서브클래스(NUserDao, DUserDao)를 함께 수정하거나 다시 개발해야할 수도 있다. 또한 다른 DAO 클래스에 적용할 수 없다는 것도 큰 단점이 될 수 있다. 다른 DAO 클래스가 계속 만들어질 경우 `getConnection()` 의 코드가 반복되어 나타날 것이다.



# < 클래스의 분리 >
지금까지는 관심사를 분리하는 작업을 하는데 있어, 독립된 메소드를 만들었고 그 다음은 상하위 클래스로 분리하였다. 그리고 이는 완전히 독립된 클래스를 구현하여 해결할 수도 있다.

__UserDao (코드2)__
~~~ java
packcage springbook.user.dao;

public class UserDao{

    private SimpleConnectionMaker simpleConnectionMaker;

    public UserDao(){

        this.simpleConnectionMaker = new SimpleConnectionMaker();

    }

    public void add(User user) throws ClassNotFoundException, SQLException{

        Connection c = this.simpleConnectionMaker.makeNewConnection();

        PreparedStatement ps = c.prpareStatement(
            "INSERT INTO USERS(id, name) VALUES (?,?)"
        );

        ps.seString(1, user.getId());
        ps.seString(2, user.getName());

        ps.executeUpdate();

        ps.close();
        c.close();

    }

    public User get(String id) throws ClassNotFoundException, SQLException{

        Connection c = this.simpleConnectionMaker.makeNewConnection();
        
        PreparedStatement ps = c.prpareStatement()
            "SELECT * FROM USERS WHERE id = ?"
        );

        ps.seString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User u = new User();
        u.setId(rs.getString("id"));
        u.setName(rs.getString("name"));

        rs.close();
        ps.close();
        c.close();

        return u;

    }

}

public class SimpleConnectionMaker{

    public Connection makeNewConnection(){

        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                                "jdbc:mysql://localhost/springbook", "aaa", "bbb"
                        );
        return c;
        
    }

}
~~~

#### [문제점]
UserDao가 SimpleConnectionMaker라는 특정 클래스에 종속되어 있기 때문에 상속을 사용했을 때 처럼 UserDao 코드의 수정 없이 DB 커넥션 생성 기능을 변경 할 수 없다.

__[문제점 1]__  SimpleConnectionMaker 클래스는 makeConnection() 이라는 이름으로 DB 커넥션을 가져온다. 하지만 N사의 UserDao의 DB 커넥션을 가져오는 메소드의 이름이 `getConnection()` 이고 D사의 UserDao에는 `openConncetion()`일 경우 UserDao 클래스의 메소드들이 모두 수정되어야 한다.
__[문제점 2]__  DB커넥션 제공하는 클래스가 어떤 것인지를 UserDao가 구체적으로 알고 이어야 한다는 점이다. N사에서 다른 클래스를 구현하여 DB 접속을 맺고자 하면 어쩔 수 없이 UserDao를 수정해야한다.


# < 인터페이스의 도입 >
클래스 분리로 인한 문제점은 __인터페이스(interface)__ 를 사용함으로서 해결할 수 있다. 즉 두 개의 클래스가 긴밀하게 연결되어 있지 않도록 중간에 인터페이스라는 추상적인 느슨한 연결고리를 만들어 주는 것이다. 인터페이스는 자신을 구현할 클래스에 대한 구체적인 정보는 모두 감추어 버린다. 인트페이스를 통해 접근하는 쪽에서는 오브젝트를 만들 때 사용할 클래스가 무엇인지 몰라도 된다.

__ConnectionMaker 인터페이스__
~~~java
public interface ConnectionMaker {

    public Connection makeConnection() throws ClassNotFoundException, SQLException;

}
~~~

__ConnectionMaker 구현 클래스__
~~~java
public class DConnectionMaker implements ConnectionMaker{

    public Connection makeConnection() throws ClassNotFoundException, SQLException{

        // D사의 독자정인 방법으로 Connection 을 생성하는 코드

    }

}
~~~

__ConnectionMaker 인터페이스를 사용하도록 개선한 UserDao__
~~~ java
public class UserDao{

    private ConnectionMaker connectionMaker;

    public UserDao(){

        this.connectionMaker = new DConnectionMaker();

    }
    public void add(User user) throws ClassNotFoundException, SQLException{

        Connection c = connectionMaker.makeConnection();
        
        // ...

    }

    public User get(String id) throws ClassNotFoundException, SQLException{

        Connection c = connectionMaker.makeConnection();
        
        // ...

    }

}
~~~
#### [문제점]
인터페이스를 사용함으로서 클래스간의 관계를 느슨하게 하는 작업은 성공했다. 하지만 여전히 DB커넥션을 제공하는 클래스에 대한 구체적인 정보가 코드에 제거되지 않고 남아있다. 생성자의 `connectionMaker = new DConnectionMaker();` 와 같이 구체적인 클래스의 이름이 남아있으므로 자유로운 수정이 불가능하다.

# < 관계설정 책임의 분리 >
UserDao에는 어떤 ConnectionMaker 구현 클래스를 사용할지를 결정하는 `new DConnectionMaker()` 라는 코드가 있다. `new DConnectionMaker();`는 UserDao와 UserDao가 사용할 특정 구현 클래스 간의 __관계를 설정해주는__  코드이다. 이 코드를 UserDao에서 분리하지 않으면 UserDao는 결코 독립적으로 확장가능한 클래스가 될 수 없다.
오브젝트 사이의 관계가 만들어지려면 만들어진 오브젝트가 있어야 한다. 이는 직접 생성자를 호출하여 직접 오브젝트를 만들수도 있지만 외부에서 만들어준 것을 가져오는 방법도 있다. UserDao 오브젝트가 다른 오브젝트와 관계를 맺으려면 관계를 맺을 오브젝트가 있어야 하고 이는 꼭 UserDao 코드 내에서 만들 필요는 없다. 오브젝트는 외부에서 만든 것을 가져올 수도 있다.

~~~java
public class UserDao{

    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectionMaker){

        this.connectionMaker = connectionMaker;

    }
    public void add(User user) throws ClassNotFoundException, SQLException{

        Connection c = connectionMaker.makeConnection();
        
        // ...

    }

    public User get(String id) throws ClassNotFoundException, SQLException{

        Connection c = connectionMaker.makeConnection();
        
        // ...

    }

}
~~~

이 코드에서는 `new DConnectionMaker()` 가 사라졌다. 이는 UserDao와 특정 ConnectionMaker 구현 클래스의 오브젝트 간 관계를 맺는 책임을 담당하였으나, 이제 이 책임은 UserDao를 호출하는 클래스로 넘어갔다.

__UserDao와 ConnectionMaker 구현 클래스의 관계설정 책임을 갖게된 UserDaoTest__
~~~java
public class UserDaoTest{

    public static void main(String[] args) throws ClassNotFounException, SQLException{
        
        // D사의 경우
        ConnectionMaker connectionMaker = new DConnectionMaker();
        // N사의 경우
        // ConnectionMaker connectionMaker = new NConnectionMaker();

        UserDao dao = new UserDao(connectionMaker);

    }

}
~~~
