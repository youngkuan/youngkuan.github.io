---
title: MyIsam存储引擎
date: 2019-04-10 10:18:51
tags: 
    - MyIsam
    - 数据库
    - 问题
categories: 数据库
---

1. Spring Boot的@Transactional注解对MyISAM存储引擎生效的问题？

未解决问题，一脸懵逼。。。。

<!-- more --> 
其中，数据表对应的实体对象如下，
```
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class User {

    @GeneratedValue(strategy = GenerationType.AUTO)
    @Id
    private Long id;

    private String name;
    private String password;

    public User() {
    }

    public User(String name, String password) {
        this.name = name;
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```
使用Spring Data Jpa进行数据操作，代码如下：
```
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

@Repository("user_repository")
public interface UserRepository extends JpaRepository<User, Long> {

    Integer deleteUserById(Long id);

    @Modifying(clearAutomatically = true)
    @Query(value = "update User set name=?2,password=?3 where id=?1")
    Integer updateUserById(Long id,String name,String password);

}
```

业务逻辑层代码如下：
```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service("uService")
public class UserService {

    @Autowired
    UserRepository userRepository;


    @Transactional(rollbackFor = Exception.class)
    public Integer deleteUserById(Long id){
        Integer i = userRepository.deleteUserById(id);
        if(id.equals(718L)){
            throw new IllegalArgumentException("删除714错误，回滚");
        }
        return i;
    }

    @Transactional(rollbackFor = {IllegalArgumentException.class})
    public User saveUser(User user){
        User u = userRepository.save(user);

        if(u.getName()=="yang1"){
            throw new IllegalArgumentException("存入yang，导致回滚");
        }
        return u;
    }

    @Transactional(rollbackFor = {IllegalArgumentException.class})
    public Integer updateUserById(User user){
        Integer i = userRepository.updateUserById(user.getId(),user.getName(),user.getPassword());
        return i;
    }
}
```
最后是测试代码，
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceTest {

    @Autowired
    UserService userService;

    private final Logger LOGGER = LoggerFactory.getLogger(this.getClass());

    @Test
    public void deleteUserById() {
        Integer i = userService.deleteUserById(718L);
        LOGGER.error("delete by user id successful: "+ i);
    }

    @Test
    public void saveUser(){
        User user = new User("yang","adasdsa");
        User u = userService.saveUser(user);
        LOGGER.error(u.toString());
    }

    @Test
    public void updateUser(){
        User user = new User("yang","adasdsa");
        user.setId(714L);
        int i = userService.updateUserById(user);
        LOGGER.error("update by user id successful: "+ i);
    }
}
```

在测试删除用户（deleteUserById）以及保存用户（saveUser）时，发现抛出异常会导致回滚，但是我的数据库引擎是MyIsam，讲道理不应该支持事务。