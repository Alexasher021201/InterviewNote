可以。你现在复习 Java / 后端面试的话，JPA 不需要一上来钻特别底层，先把 **「它是什么 → 怎么写 → 实体关系 → 查询 → 事务 → 高频坑」** 这条线掌握住。

# Java JPA 复习笔记

## 1. JPA 是什么？

**JPA（Java Persistence API）是 Java 定义的一套 ORM 规范，用来解决 Java 对象和关系型数据库之间的数据映射问题。**

注意：

> **JPA 是规范，不是具体实现。**

常见关系：

```text
Java 应用
   ↓
Spring Data JPA
   ↓
JPA 规范
   ↓
Hibernate（常见实现）
   ↓
JDBC
   ↓
MySQL / PostgreSQL...
```

所以面试中如果问：

**JPA 和 Hibernate 有什么区别？**

可以回答：

> JPA 是 Java 官方定义的 ORM 规范，规定了实体映射、持久化、查询等 API；Hibernate 是 JPA 最常见的实现之一。

而 **Spring Data JPA** 又是在 JPA 之上的进一步封装，帮助我们少写 DAO 层代码。

---

# 2. ORM 是什么？

ORM：

```text
Object Relational Mapping
对象关系映射
```

它解决的是：

```text
Java Object        Database

User         ←→    user 表

id           ←→    id
name         ←→    name
age          ←→    age
```

例如数据库：

```sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    age INT
);
```

Java 可以写：

```java
@Entity
@Table(name = "user")
public class User {

    @Id
    private Long id;

    private String name;

    private Integer age;
}
```

JPA 就负责：

```text
数据库的一行数据
      ↓
   User对象
```

因此以前 JDBC 可能要自己写：

```java
ResultSet rs = ...
User user = new User();
user.setId(rs.getLong("id"));
user.setName(rs.getString("name"));
```

用了 JPA 后，这部分映射工作通常由 ORM 框架完成。

---

# 3. 最基本的 Entity

这是 JPA 最重要的代码之一：

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username")
    private String username;

    private Integer age;
}
```

几个注解必须认识。

### `@Entity`

表示：

> 这是一个 JPA 实体类。

```java
@Entity
public class User {
}
```

通常对应数据库中的一张表。

---

### `@Table`

指定对应哪张表：

```java
@Table(name = "users")
```

即：

```text
User class
   ↓
users table
```

---

### `@Id`

指定主键：

```java
@Id
private Long id;
```

---

### `@GeneratedValue`

指定主键生成策略：

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

例如 MySQL 自增：

```text
1
2
3
4
...
```

常见策略包括：

```java
GenerationType.IDENTITY
GenerationType.SEQUENCE
GenerationType.AUTO
GenerationType.TABLE
```

面试/开发里重点知道 `IDENTITY` 和 `SEQUENCE` 即可。

---

### `@Column`

字段映射：

```java
@Column(name = "user_name")
private String username;
```

表示：

```text
Java                  DB

username      ←→      user_name
```

还可以：

```java
@Column(
    name = "username",
    nullable = false,
    unique = true,
    length = 50
)
```

---

# 4. Spring Data JPA 的 Repository

实际 Spring Boot 项目里，你最常见到的是：

```java
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```

这里：

```text
User → 操作哪个 Entity
Long → User 主键是什么类型
```

`JpaRepository` 已经帮你提供很多 CRUD 方法。

例如：

```java
userRepository.save(user);
```

插入/更新。

```java
userRepository.findById(1L);
```

查询。

```java
userRepository.findAll();
```

查询全部。

```java
userRepository.deleteById(1L);
```

删除。

```java
userRepository.count();
```

统计。

所以 Spring Data JPA 一个核心价值就是：

> **很多简单 CRUD 不需要自己写 SQL 和 DAO 实现。**

---

# 5. JPA 最方便的地方：方法名查询

假设：

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;

    private Integer age;
}
```

可以直接：

```java
public interface UserRepository
        extends JpaRepository<User, Long> {

    List<User> findByName(String name);

}
```

Spring Data JPA 会根据方法名理解你的意思。

相当于：

```sql
SELECT *
FROM user
WHERE name = ?
```

甚至：

```java
List<User> findByAgeGreaterThan(Integer age);
```

相当于：

```sql
WHERE age > ?
```

还可以：

```java
List<User> findByNameAndAge(
    String name,
    Integer age
);
```

相当于：

```sql
WHERE name = ?
AND age = ?
```

常见关键词建议记住：

```text
findBy
And
Or
GreaterThan
LessThan
Between
Like
Containing
In
OrderBy
```

例如：

```java
findByAgeGreaterThanOrderByAgeDesc(...)
```

---

# 6. JPQL

复杂查询可以自己写。

例如：

```java
@Query("""
    SELECT u
    FROM User u
    WHERE u.age > :age
""")
List<User> findAdultUsers(
    @Param("age") Integer age
);
```

这里有个非常重要的点：

JPQL：

```sql
SELECT u
FROM User u
WHERE u.age > :age
```

不是直接操作数据库表，而是操作：

```text
Entity + Entity属性
```

所以：

```text
User → Entity
age  → Java属性
```

这和 SQL：

```sql
SELECT *
FROM users
WHERE age > ?
```

不是完全一个概念。

---

# 7. Native SQL

如果确实想直接写 SQL：

```java
@Query(
    value = "SELECT * FROM users WHERE age > :age",
    nativeQuery = true
)
List<User> findUsers(Integer age);
```

区别：

```text
JPQL
↓
操作 Entity

Native SQL
↓
直接操作数据库表
```

一般业务优先考虑 JPQL / Repository 方法查询；复杂数据库特性、特殊优化场景可能使用 Native SQL。

---

# 8. JPA 最重要的部分：实体关系

面试非常喜欢问。

数据库里经常有：

```text
User
Order
Product
Department
Employee
```

它们之间存在关系。

JPA 主要有四种：

```text
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
```

---

## 9. `@OneToMany` 和 `@ManyToOne`

这是实际开发里非常常见的。

比如：

```text
User 1 -------- N Order

一个用户
可以有多个订单

一个订单
属于一个用户
```

Order：

```java
@Entity
public class Order {

    @Id
    private Long id;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

数据库：

```text
orders

id
product_name
user_id
```

这里：

```java
@JoinColumn(name = "user_id")
```

表示外键字段。

User 可以写：

```java
@OneToMany(mappedBy = "user")
private List<Order> orders;
```

这里最容易搞混的是：

```java
mappedBy = "user"
```

这个 `user` 指的是：

```java
class Order {

    @ManyToOne
    private User user;
                 ↑
             就是这里
}
```

不是数据库字段名。

---

# 10. 谁维护关系？

JPA 双向关系里有一个：

```text
Owning Side
关系维护方
```

例如：

```java
@ManyToOne
@JoinColumn(name = "user_id")
private User user;
```

通常有 `@JoinColumn` 的这一侧是维护方。

另一边：

```java
@OneToMany(mappedBy = "user")
private List<Order> orders;
```

`mappedBy` 表示：

> 我不是关系维护方，关系由 Order.user 管理。

这也是 JPA 面试比较容易问的知识点。

---

# 11. FetchType：懒加载和立即加载

非常重要。

JPA 有：

```java
FetchType.LAZY
FetchType.EAGER
```

### LAZY

需要的时候再查询：

```text
查询 User
   ↓
暂时不查询 orders

user.getOrders()
   ↓
此时再加载 orders
```

### EAGER

查询 User 时直接把关联对象也加载。

```text
User
 +
Orders
一起加载
```

一般来说，关联数据很多时：

```text
LAZY
```

更容易避免无意义的数据加载。

JPA 默认值也值得记：

```text
@OneToMany   → LAZY
@ManyToMany  → LAZY

@ManyToOne   → EAGER
@OneToOne    → EAGER
```

但实际项目通常会根据查询需求显式控制，而不是完全依赖默认值。

---

# 12. N+1 问题

这是 **JPA/Hibernate 面试超高频题**。

假设：

```java
List<User> users = userRepository.findAll();
```

产生一次：

```sql
SELECT * FROM users;
```

然后：

```java
for (User user : users) {
    System.out.println(user.getOrders());
}
```

如果每个 User 的 orders 都单独查询：

```sql
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2;
SELECT * FROM orders WHERE user_id = 3;
...
```

假设 100 个用户：

```text
查询 User：1 次

查询 Order：100 次

总共：
1 + 100 = 101 次
```

这就是：

> **N+1 Query Problem**

常见解决方式之一是 `fetch join`：

```java
@Query("""
    SELECT DISTINCT u
    FROM User u
    LEFT JOIN FETCH u.orders
""")
List<User> findUsersWithOrders();
```

这样可以在需要的时候一次性获取关联数据。

---

# 13. Entity 的生命周期

JPA 中 Entity 有几个状态。

这个是面试重点：

```text
Transient
    ↓
Managed
    ↓
Detached
    ↓
Removed
```

### Transient

普通 Java 对象：

```java
User user = new User();
```

还没有被 EntityManager 管理。

### Managed / Persistent

对象已经进入 Persistence Context：

```java
entityManager.persist(user);
```

JPA 开始管理它。

### Detached

原本被管理，但后来脱离 Persistence Context。

### Removed

准备被删除：

```java
entityManager.remove(user);
```

---

# 14. Persistence Context

这个概念非常重要。

可以理解成：

> **EntityManager 管理 Entity 的一个上下文/容器。**

例如：

```java
User user = entityManager.find(User.class, 1L);
```

查询出来之后：

```text
Persistence Context

id=1 → User对象
```

如果再次：

```java
entityManager.find(User.class, 1L);
```

在合适的上下文中，JPA 可以直接使用已经管理的 Entity。

这涉及：

> **一级缓存（First-Level Cache）**

一级缓存是 Persistence Context 级别的。

---

# 15. Dirty Checking

这是 JPA 非常核心的机制。

例如：

```java
@Transactional
public void updateUser(Long id) {

    User user = userRepository.findById(id)
        .orElseThrow();

    user.setName("Tom");
}
```

你可能发现：

```java
userRepository.save(user);
```

都没有调用。

但事务提交的时候依然可能执行：

```sql
UPDATE users
SET name = 'Tom'
WHERE id = ?;
```

为什么？

因为这个 `user` 是 **Managed Entity**。

Hibernate/JPA 会追踪实体状态变化：

```text
查询 User
   ↓
进入 Persistence Context
   ↓
user.setName()
   ↓
对象发生变化
   ↓
事务提交 / flush
   ↓
检测变化
   ↓
生成 UPDATE SQL
```

这就是：

> **Dirty Checking（脏检查）**

面试时这是一个非常值得讲出来的点。

---

# 16. `save()` 到底是什么？

Spring Data JPA：

```java
repository.save(user);
```

底层通常根据实体状态选择类似：

```java
EntityManager.persist()
```

或者：

```java
EntityManager.merge()
```

不是简单粗暴地：

```text
save = INSERT
```

所以不要背成：

```java
save() 就是 insert
```

更准确是：

> `save()` 用于保存实体，Spring Data JPA 会根据实体是否被认为是新的，决定采用 `persist` 或 `merge` 等持久化行为。

---

# 17. `@Transactional`

JPA 和事务关系非常紧密。

例如：

```java
@Transactional
public void transfer() {

    accountA.setMoney(...);

    accountB.setMoney(...);
}
```

事务保证：

```text
操作 A
操作 B

要么一起成功
要么一起失败
```

而且前面说的 Dirty Checking 通常也是结合事务理解：

```text
@Transactional
      ↓
Persistence Context
      ↓
Managed Entity
      ↓
修改 Entity
      ↓
Dirty Checking
      ↓
flush
      ↓
SQL
      ↓
commit
```

这条链非常重要。

---

# 18. Cascade

实体之间存在关联时，经常出现：

```java
@OneToMany(
    mappedBy = "user",
    cascade = CascadeType.ALL
)
private List<Order> orders;
```

Cascade 表示：

> 对 User 做某些操作时，要不要把操作级联到 Order。

例如：

```java
CascadeType.PERSIST
```

保存 User 时级联保存 Order。

```java
CascadeType.REMOVE
```

删除 User 时级联删除 Order。

常见：

```java
CascadeType.ALL
```

代表多种操作都进行级联。

但是 `ALL` 不能无脑使用，尤其是删除关系复杂的数据时，可能造成意料之外的级联删除。

---

# 19. `orphanRemoval`

例如：

```java
@OneToMany(
    mappedBy = "user",
    orphanRemoval = true
)
private List<Order> orders;
```

如果：

```java
user.getOrders().remove(order);
```

Order 已经从这个关联中被移除。

`orphanRemoval = true` 可以让 JPA 把这个“孤儿”实体从数据库删除。

注意它和：

```java
CascadeType.REMOVE
```

概念并不完全相同。

可以简单理解：

```text
Cascade REMOVE
父实体被删除
    ↓
子实体跟着删除


orphanRemoval
子实体与父实体解除关系
    ↓
子实体被删除
```

---

# 20. 分页和排序

Spring Data JPA 经常：

```java
Pageable pageable =
    PageRequest.of(0, 10);
```

查询：

```java
Page<User> users =
    userRepository.findAll(pageable);
```

注意：

```text
PageRequest.of(0, 10)

0  → 第 0 页
10 → 每页 10 条
```

排序：

```java
PageRequest.of(
    0,
    10,
    Sort.by("age").descending()
);
```

---

# 21. 一张图理解整个 JPA

建议你把这条链记下来：

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Spring Data JPA
    ↓
JPA API
    ↓
Hibernate
    ↓
JDBC
    ↓
Database
```

数据映射：

```text
Java Entity
     ↕
   ORM
     ↕
Database Table
```

事务内部：

```text
@Transactional
      ↓
Persistence Context
      ↓
Managed Entity
      ↓
修改 Entity
      ↓
Dirty Checking
      ↓
flush
      ↓
SQL
      ↓
Database
```

---

# 22. 面试最值得背的 10 个问题

如果你现在是为了 **Java 后端面试复习**，建议优先掌握这些：

| 问题 | 核心回答 |
|---|---|
| JPA 是什么？ | Java ORM 规范 |
| JPA 和 Hibernate 区别？ | JPA 是规范，Hibernate 是实现 |
| Spring Data JPA 是什么？ | 对 JPA Repository 层进一步封装 |
| `@Entity` 是什么？ | 声明 JPA 实体 |
| `@OneToMany` / `@ManyToOne`？ | 描述实体关联关系 |
| LAZY 和 EAGER？ | 延迟加载 vs 立即加载 |
| N+1 是什么？ | 查询主体 1 次，关联对象额外查询 N 次 |
| Persistence Context 是什么？ | EntityManager 管理实体的上下文 |
| Dirty Checking 是什么？ | 自动检测 Managed Entity 的修改并同步 DB |
| `@Transactional` 和 JPA？ | 事务界定持久化上下文中的一组数据库操作 |

最后尤其建议你把 **`Persistence Context → Managed Entity → Dirty Checking → Flush → SQL`** 理清楚。很多 JPA 面试题表面上问 `save()`、事务、更新、缓存，实际上都绕不开这条主线。