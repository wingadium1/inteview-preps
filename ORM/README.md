# ORM (Object-Relational Mapping) - Interview Preparation

## Table of Contents
1. [ORM Fundamentals](#orm-fundamentals)
2. [Core ORM Concepts](#core-orm-concepts)
3. [Popular ORM Frameworks](#popular-orm-frameworks)
4. [Framework Comparison](#framework-comparison)
5. [Common Interview Questions](#common-interview-questions)
6. [Code Examples](#code-examples)
7. [Best Practices](#best-practices)

---

## ORM Fundamentals

### What is ORM?
**Object-Relational Mapping (ORM)** is a programming technique that converts data between incompatible type systems in object-oriented programming languages. ORM creates a "virtual object database" that can be used from within the programming language.

**Tiếng Việt:** ORM là kỹ thuật ánh xạ dữ liệu giữa các đối tượng trong lập trình hướng đối tượng và cơ sở dữ liệu quan hệ.

### Why Use ORM?
- **Database Independence**: Switch databases with minimal code changes
- **Reduced SQL Code**: Less boilerplate SQL code to write
- **Object-Oriented Approach**: Work with objects instead of SQL queries
- **Productivity**: Faster development with automatic CRUD operations
- **Type Safety**: Compile-time type checking
- **Maintainability**: Easier to maintain and refactor

### Disadvantages of ORM
- **Performance Overhead**: Additional abstraction layer can impact performance
- **Learning Curve**: Need to understand both ORM framework and underlying SQL
- **Complex Queries**: Sometimes native SQL is more efficient
- **Memory Consumption**: Object representations can use more memory

---

## Core ORM Concepts

### 1. Persistence (Tính bền vững)
**Persistence** is the ability to save object states to a database and retrieve them later.

**Tiếng Việt:** Persistence là khả năng lưu trạng thái của đối tượng vào cơ sở dữ liệu và truy xuất lại sau này.

#### Object States / Trạng thái đối tượng
- **Transient (Tạm thời)**: Object created but not associated with any database session
- **Persistent (Bền vững)**: Object associated with a session and has a database representation
- **Detached (Tách rời)**: Object was persistent but session is closed
- **Removed (Đã xóa)**: Object marked for deletion

```java
// Transient state
User user = new User("John");

// Persistent state - associate with Hibernate Session
Session session = sessionFactory.openSession();
session.save(user);

// Detached state - session is closed
session.close();

// Removed state - mark for deletion
session = sessionFactory.openSession();
session.delete(user);
```

### 2. Transactions (Giao dịch)
**Transaction** is a sequence of operations that are treated as a single unit of work.

**Tiếng Việt:** Transaction là một chuỗi các thao tác được xử lý như một đơn vị công việc duy nhất.

#### ACID Properties
- **Atomicity (Tính nguyên tử)**: All or nothing - tất cả hoặc không có gì
- **Consistency (Tính nhất quán)**: Data remains consistent - dữ liệu luôn nhất quán
- **Isolation (Tính cô lập)**: Transactions don't interfere with each other - các giao dịch không can thiệp lẫn nhau
- **Durability (Tính bền vững)**: Changes are permanent - các thay đổi được lưu vĩnh viễn

#### Isolation Levels / Mức độ cô lập
- **READ_UNCOMMITTED**: Lowest isolation, dirty reads possible
- **READ_COMMITTED**: Prevents dirty reads
- **REPEATABLE_READ**: Prevents dirty and non-repeatable reads
- **SERIALIZABLE**: Highest isolation, prevents all anomalies

```java
Transaction tx = session.beginTransaction();
try {
    // Perform operations
    session.save(user);
    session.update(order);
    tx.commit();
} catch (Exception e) {
    tx.rollback();
    throw e;
}
```

### 3. Caching (Bộ nhớ đệm)
**Caching** stores frequently accessed data in memory to improve performance.

**Tiếng Việt:** Caching lưu trữ dữ liệu thường xuyên truy cập trong bộ nhớ để cải thiện hiệu suất.

#### Cache Levels / Các mức cache
- **First-Level Cache (L1)**: Session/EntityManager level cache (mandatory)
- **Second-Level Cache (L2)**: SessionFactory/EntityManagerFactory level cache (optional)
- **Query Cache**: Caches query results

#### Cache Strategies / Chiến lược cache
- **READ_ONLY**: For immutable data, best performance - Dữ liệu không thay đổi
- **READ_WRITE**: For data that can be updated, uses soft locks - Dữ liệu có thể cập nhật
- **NONSTRICT_READ_WRITE**: For data that rarely changes, no locks - Dữ liệu ít thay đổi
- **TRANSACTIONAL**: Full transactional cache, JTA required - Cache giao dịch đầy đủ

```java
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    @Id
    private Long id;
    private String name;
}
```

### 4. Fetching Strategies (Chiến lược tải dữ liệu)
**Fetching** determines when and how related entities are loaded.

**Tiếng Việt:** Fetching xác định thời điểm và cách thức tải các entity liên quan.

#### Fetch Types
- **LAZY Loading**: Load data on-demand when accessed - Tải dữ liệu khi cần
- **EAGER Loading**: Load data immediately with parent entity - Tải dữ liệu ngay lập tức

#### Fetching Patterns
- **N+1 Problem**: Multiple queries for related entities - Vấn đề nhiều truy vấn
- **JOIN FETCH**: Single query with join to load related entities
- **Batch Fetching**: Load related entities in batches - Tải theo lô
- **Subselect Fetching**: Use subselect to load related entities

```java
// Lazy loading
@ManyToOne(fetch = FetchType.LAZY)
private User user;

// Eager loading
@ManyToOne(fetch = FetchType.EAGER)
private Category category;

// JOIN FETCH to avoid N+1
SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id
```

### 5. Relationships and Mapping (Quan hệ và ánh xạ)
**Relationships** define how entities are related to each other.

#### Relationship Types / Các loại quan hệ
- **One-to-One**: One entity relates to one other entity - Một-một
- **One-to-Many**: One entity relates to many entities - Một-nhiều
- **Many-to-One**: Many entities relate to one entity - Nhiều-một
- **Many-to-Many**: Many entities relate to many entities - Nhiều-nhiều

#### Cascade Operations / Thao tác lan truyền
- **PERSIST**: Cascade persist operations
- **MERGE**: Cascade merge operations
- **REMOVE**: Cascade delete operations
- **REFRESH**: Cascade refresh operations
- **DETACH**: Cascade detach operations
- **ALL**: All cascade operations

### 6. Locking Mechanisms (Cơ chế khóa)
**Locking** prevents concurrent modification conflicts.

**Tiếng Việt:** Locking ngăn chặn xung đột khi thay đổi đồng thời.

#### Optimistic Locking (Khóa lạc quan)
- Uses version number or timestamp - Sử dụng số phiên bản
- Checks version before update - Kiểm tra phiên bản trước khi cập nhật
- Throws exception if version mismatch - Ném ngoại lệ nếu phiên bản không khớp
- Better for read-heavy workloads - Tốt cho khối lượng công việc đọc nhiều

```java
@Entity
public class Account {
    @Id
    private Long id;
    
    @Version
    private Long version;
    
    private BigDecimal balance;
}
```

#### Pessimistic Locking (Khóa bi quan)
- Locks database row - Khóa hàng cơ sở dữ liệu
- Prevents other transactions from accessing - Ngăn các giao dịch khác truy cập
- Better for write-heavy workloads - Tốt cho khối lượng công việc ghi nhiều
- Can cause deadlocks - Có thể gây ra deadlock

```java
// Pessimistic read lock
User user = em.find(User.class, id, LockModeType.PESSIMISTIC_READ);

// Pessimistic write lock
User user = em.find(User.class, id, LockModeType.PESSIMISTIC_WRITE);
```

---

## Popular ORM Frameworks

### 1. Hibernate (Most Popular / Phổ biến nhất)

**Hibernate** is the most widely used ORM framework for Java applications.

**Tiếng Việt:** Hibernate là framework ORM được sử dụng rộng rãi nhất cho các ứng dụng Java.

#### Key Features / Tính năng chính
- Full JPA implementation - Triển khai đầy đủ JPA
- HQL (Hibernate Query Language)
- Native SQL support - Hỗ trợ SQL gốc
- Extensive caching support - Hỗ trợ caching mở rộng
- Mature and stable - Trưởng thành và ổn định
- Large community - Cộng đồng lớn

#### When to Use Hibernate / Khi nào dùng Hibernate
✅ **Use when / Sử dụng khi:**
- Building enterprise applications - Xây dựng ứng dụng doanh nghiệp
- Need complex object-relational mapping - Cần ánh xạ phức tạp
- Require advanced caching - Yêu cầu caching nâng cao
- Want JPA compliance with Hibernate extensions - Muốn tuân thủ JPA với extension của Hibernate
- Need extensive documentation and community support - Cần tài liệu và hỗ trợ cộng đồng

❌ **Avoid when / Tránh khi:**
- Building simple CRUD applications - Xây dựng ứng dụng CRUD đơn giản
- Performance is critical and queries are simple - Hiệu suất quan trọng và truy vấn đơn giản
- Team lacks ORM experience - Đội thiếu kinh nghiệm ORM

#### Example
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String username;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders;
}

// HQL Query
String hql = "FROM User u WHERE u.username = :username";
User user = session.createQuery(hql, User.class)
    .setParameter("username", "john")
    .uniqueResult();
```

### 2. Spring Data JPA (Modern Approach / Cách tiếp cận hiện đại)

**Spring Data JPA** is a higher-level abstraction over JPA (typically uses Hibernate as provider).

**Tiếng Việt:** Spring Data JPA là lớp trừu tượng cấp cao hơn trên JPA (thường sử dụng Hibernate làm provider).

#### Key Features / Tính năng chính
- Repository pattern - Mẫu Repository
- Query methods from method names - Tự động tạo query từ tên method
- No boilerplate code - Không cần code mẫu
- Pagination and sorting support - Hỗ trợ phân trang và sắp xếp
- Integration with Spring ecosystem - Tích hợp với hệ sinh thái Spring
- Specification API for dynamic queries - API Specification cho truy vấn động

#### When to Use Spring Data JPA / Khi nào dùng Spring Data JPA
✅ **Use when / Sử dụng khi:**
- Building Spring Boot applications - Xây dựng ứng dụng Spring Boot
- Want to minimize boilerplate code - Muốn giảm thiểu code mẫu
- Need automatic repository implementation - Cần triển khai repository tự động
- Prefer convention over configuration - Ưu tiên quy ước hơn cấu hình
- Working with standard CRUD operations - Làm việc với CRUD chuẩn

❌ **Avoid when / Tránh khi:**
- Not using Spring framework - Không dùng Spring framework
- Need fine-grained control over queries - Cần kiểm soát chi tiết truy vấn
- Working with very complex mappings - Làm việc với ánh xạ rất phức tạp

#### Example
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String email;
}

// Repository interface - no implementation needed!
// Interface Repository - không cần implement!
public interface UserRepository extends JpaRepository<User, Long> {
    // Method name is automatically converted to query
    // Tên method tự động chuyển thành query
    User findByUsername(String username);
    
    List<User> findByEmailContaining(String email);
    
    @Query("SELECT u FROM User u WHERE u.username = :username")
    User findUserByUsername(@Param("username") String username);
}

// Usage / Sử dụng
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public User getUser(String username) {
        return userRepository.findByUsername(username);
    }
}
```

### 3. MyBatis (SQL-First Approach / Tiếp cận SQL trước)

**MyBatis** is a persistence framework that focuses on SQL mapping rather than full ORM.

**Tiếng Việt:** MyBatis là framework persistence tập trung vào ánh xạ SQL hơn là ORM đầy đủ.

#### Key Features / Tính năng chính
- SQL-centric approach - Tiếp cận tập trung SQL
- XML or annotation-based SQL mapping - Ánh xạ SQL qua XML hoặc annotation
- Better performance for complex queries - Hiệu suất tốt hơn cho truy vấn phức tạp
- More control over SQL - Kiểm soát SQL nhiều hơn
- Dynamic SQL support - Hỗ trợ SQL động
- Integration with Spring - Tích hợp với Spring

#### When to Use MyBatis / Khi nào dùng MyBatis
✅ **Use when / Sử dụng khi:**
- Need fine-grained control over SQL - Cần kiểm soát chi tiết SQL
- Working with legacy databases - Làm việc với cơ sở dữ liệu cũ
- Team is strong in SQL - Đội mạnh về SQL
- Performance is critical - Hiệu suất rất quan trọng
- Have complex stored procedures - Có stored procedure phức tạp
- Need dynamic SQL generation - Cần tạo SQL động

❌ **Avoid when / Tránh khi:**
- Want automatic CRUD operations - Muốn CRUD tự động
- Prefer working with objects over SQL - Thích làm việc với object hơn SQL
- Need database portability - Cần tính di động cơ sở dữ liệu
- Building simple applications - Xây dựng ứng dụng đơn giản

#### Example
```java
// Entity
public class User {
    private Long id;
    private String username;
    private String email;
    // Getters and setters
}

// Mapper interface
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);
    
    @Insert("INSERT INTO users(username, email) VALUES(#{username}, #{email})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insert(User user);
    
    @Update("UPDATE users SET username=#{username}, email=#{email} WHERE id=#{id}")
    void update(User user);
}

// XML Mapping (alternative to annotations)
// Ánh xạ XML (thay thế cho annotation)
<!-- UserMapper.xml -->
<mapper namespace="com.example.UserMapper">
    <select id="findById" resultType="User">
        SELECT * FROM users WHERE id = #{id}
    </select>
    
    <select id="findByDynamicConditions" resultType="User">
        SELECT * FROM users
        <where>
            <if test="username != null">
                AND username = #{username}
            </if>
            <if test="email != null">
                AND email = #{email}
            </if>
        </where>
    </select>
</mapper>
```

### 4. Other Notable ORMs / Các ORM đáng chú ý khác

#### JOOQ (Java Object Oriented Querying)
- Type-safe SQL construction - Xây dựng SQL an toàn kiểu
- Code generation from database schema - Tạo code từ schema database
- Great for complex queries - Tuyệt vời cho truy vấn phức tạp
- SQL-first but object-oriented - SQL trước nhưng hướng đối tượng

#### EclipseLink
- Reference implementation of JPA - Triển khai tham chiếu của JPA
- Advanced features like multi-tenancy - Tính năng nâng cao như multi-tenancy
- Good for enterprise applications - Tốt cho ứng dụng doanh nghiệp

#### QueryDSL
- Type-safe query construction - Xây dựng truy vấn an toàn kiểu
- Works with JPA, SQL, MongoDB, etc. - Hoạt động với JPA, SQL, MongoDB, v.v.
- Can be used alongside other ORMs - Có thể dùng cùng với ORM khác

---

## Framework Comparison / So sánh các Framework

### Feature Comparison Table / Bảng so sánh tính năng

| Feature / Tính năng | Hibernate | Spring Data JPA | MyBatis | JOOQ |
|---------|-----------|-----------------|---------|------|
| **Approach / Tiếp cận** | Full ORM | Repository Pattern | SQL Mapping | SQL DSL |
| **Learning Curve / Độ khó học** | Medium-High | Low | Low-Medium | Medium |
| **Boilerplate Code / Code mẫu** | Medium | Minimal | Medium | Low |
| **SQL Control / Kiểm soát SQL** | Limited | Limited | Full | Full |
| **Performance / Hiệu suất** | Good | Good | Excellent | Excellent |
| **Type Safety / An toàn kiểu** | Good | Good | Limited | Excellent |
| **Database Portability / Tính di động** | Excellent | Excellent | Limited | Limited |
| **Caching** | Advanced | Advanced | Basic | Basic |
| **Dynamic Queries / Truy vấn động** | Good (Criteria) | Good (Specification) | Excellent | Excellent |
| **Community / Cộng đồng** | Large | Large | Medium | Growing |
| **Spring Integration** | Good | Native | Good | Good |
| **Best For / Tốt nhất cho** | Enterprise apps | Spring apps | Legacy/Complex SQL | Type-safe SQL |

### Performance Comparison / So sánh hiệu suất

```
Read Operations (Simple) / Thao tác đọc (Đơn giản):
MyBatis > JOOQ > Hibernate ≈ Spring Data JPA

Read Operations (Complex) / Thao tác đọc (Phức tạp):
MyBatis ≈ JOOQ > Hibernate > Spring Data JPA

Write Operations / Thao tác ghi:
MyBatis ≈ JOOQ > Hibernate ≈ Spring Data JPA

Bulk Operations / Thao tác hàng loạt:
MyBatis > JOOQ > Hibernate > Spring Data JPA

Development Speed / Tốc độ phát triển:
Spring Data JPA > Hibernate > MyBatis ≈ JOOQ
```

### Decision Matrix / Ma trận quyết định

#### Choose **Hibernate** when / Chọn Hibernate khi:
- Building traditional enterprise Java application - Xây dựng ứng dụng doanh nghiệp Java truyền thống
- Need JPA compliance with extensions - Cần tuân thủ JPA với extension
- Require advanced caching mechanisms - Yêu cầu cơ chế caching nâng cao
- Want mature, well-documented framework - Muốn framework trưởng thành, tài liệu đầy đủ
- Team familiar with ORM concepts - Đội quen thuộc với khái niệm ORM

#### Choose **Spring Data JPA** when / Chọn Spring Data JPA khi:
- Building Spring Boot application - Xây dựng ứng dụng Spring Boot
- Want rapid development - Muốn phát triển nhanh
- Need standard CRUD with minimal code - Cần CRUD chuẩn với code tối thiểu
- Working with microservices - Làm việc với microservices
- Prefer convention over configuration - Ưu tiên quy ước hơn cấu hình

#### Choose **MyBatis** when / Chọn MyBatis khi:
- Working with legacy database schema - Làm việc với schema cơ sở dữ liệu cũ
- Need full SQL control - Cần kiểm soát SQL hoàn toàn
- Have complex stored procedures - Có stored procedure phức tạp
- Performance is critical priority - Hiệu suất là ưu tiên quan trọng
- Team has strong SQL skills - Đội có kỹ năng SQL mạnh

#### Choose **JOOQ** when / Chọn JOOQ khi:
- Need type-safe SQL construction - Cần xây dựng SQL an toàn kiểu
- Want compile-time query validation - Muốn xác thực truy vấn lúc compile
- Working with complex SQL queries - Làm việc với truy vấn SQL phức tạp
- Require database-first approach - Yêu cầu tiếp cận database trước
- Need excellent IDE support - Cần hỗ trợ IDE tuyệt vời

### Hybrid Approaches / Tiếp cận kết hợp

Many applications use **multiple frameworks** together / Nhiều ứng dụng sử dụng nhiều framework cùng nhau:

```java
// Spring Data JPA for simple CRUD / cho CRUD đơn giản
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}

// MyBatis for complex reporting queries / cho truy vấn báo cáo phức tạp
@Mapper
public interface ReportMapper {
    @Select("Complex SQL with multiple joins and aggregations")
    List<ReportDTO> generateSalesReport(ReportCriteria criteria);
}

// Service layer combines both / Lớp service kết hợp cả hai
@Service
public class UserService {
    @Autowired private UserRepository userRepository;
    @Autowired private ReportMapper reportMapper;
    
    public User getUser(Long id) {
        return userRepository.findById(id).orElse(null);
    }
    
    public List<ReportDTO> getReport(ReportCriteria criteria) {
        return reportMapper.generateSalesReport(criteria);
    }
}
```

---

## Common Interview Questions / Câu hỏi phỏng vấn thường gặp

### General ORM Questions / Câu hỏi ORM chung

1. **What is ORM and why use it? / ORM là gì và tại sao sử dụng?**
   - Explain object-relational mapping concept
   - Benefits: productivity, database independence, object-oriented
   - Trade-offs: performance overhead, learning curve

2. **Explain the difference between optimistic and pessimistic locking / Giải thích sự khác biệt giữa khóa lạc quan và bi quan**
   - Optimistic: Version-based, better for reads
   - Pessimistic: Row-level locks, better for writes
   - Use cases for each

3. **What is the N+1 problem and how do you solve it? / Vấn đề N+1 là gì và cách giải quyết?**
   - Problem: 1 query + N queries for related entities
   - Solutions: JOIN FETCH, batch fetching, @EntityGraph
   - Example with and without solution

4. **Explain lazy loading vs eager loading / Giải thích lazy loading và eager loading**
   - Lazy: Load on-demand, better performance
   - Eager: Load immediately, risk of loading too much
   - When to use each

5. **What are the different levels of caching in ORM? / Các mức caching trong ORM là gì?**
   - First-level: Session/EntityManager level
   - Second-level: SessionFactory/EntityManagerFactory level
   - Query cache: Cache query results

### Framework-Specific Questions / Câu hỏi theo framework

6. **Difference between Hibernate and JPA? / Sự khác biệt giữa Hibernate và JPA?**
   - JPA: Specification / Đặc tả
   - Hibernate: Implementation of JPA with extensions / Triển khai JPA với extension
   - HQL vs JPQL

7. **Difference between save() and persist() in Hibernate?**
   - save(): Returns generated ID, Hibernate-specific
   - persist(): Void return, JPA standard
   - Transaction boundary differences

8. **How does Spring Data JPA work? / Spring Data JPA hoạt động như thế nào?**
   - Proxy pattern for repositories
   - Query derivation from method names
   - Behind-the-scenes implementation

9. **When would you choose MyBatis over Hibernate? / Khi nào chọn MyBatis thay vì Hibernate?**
   - Complex SQL requirements / Yêu cầu SQL phức tạp
   - Legacy database schema / Schema cơ sở dữ liệu cũ
   - Performance-critical applications / Ứng dụng yêu cầu hiệu suất cao
   - Team SQL expertise / Chuyên môn SQL của đội

10. **What is the difference between first-level and second-level cache? / Sự khác biệt giữa cache cấp 1 và cấp 2?**
    - Scope: Session vs SessionFactory
    - Lifetime: Transaction vs Application
    - Configuration and providers

### Advanced Questions / Câu hỏi nâng cao

11. **How do you handle bi-directional relationships? / Cách xử lý quan hệ hai chiều?**
    - Owner side vs inverse side
    - mappedBy attribute
    - Helper methods for consistency

12. **Explain cascade types in ORM / Giải thích các loại cascade trong ORM**
    - PERSIST, MERGE, REMOVE, REFRESH, DETACH, ALL
    - Orphan removal
    - Best practices

13. **How do you implement soft delete? / Cách triển khai soft delete?**
    - @Where annotation (Hibernate)
    - @SQLDelete annotation
    - Filter configuration

14. **What is the difference between get() and load()? / Sự khác biệt giữa get() và load()?**
    - get(): Returns null if not found
    - load(): Returns proxy, throws exception if not found
    - Performance implications

15. **How do you handle database migrations with ORM? / Cách xử lý migration cơ sở dữ liệu với ORM?**
    - Flyway or Liquibase
    - Version control for schema
    - hbm2ddl.auto settings and risks

---

## Code Examples / Ví dụ code

### Example 1: Complete Entity Mapping (Hibernate/JPA)

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_username", columnList = "username"),
    @Index(name = "idx_email", columnList = "email")
})
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username", nullable = false, unique = true, length = 50)
    private String username;
    
    @Column(name = "email", nullable = false, unique = true, length = 100)
    private String email;
    
    @Column(name = "password_hash", nullable = false)
    private String passwordHash;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false)
    private UserStatus status;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, 
               fetch = FetchType.LAZY, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    @Version
    private Long version;
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Helper methods
    public void addOrder(Order order) {
        orders.add(order);
        order.setUser(this);
    }
    
    public void removeOrder(Order order) {
        orders.remove(order);
        order.setUser(null);
    }
    
    // Getters and setters
}

@Entity
@Table(name = "orders")
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "order_number", unique = true, nullable = false)
    private String orderNumber;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, 
               orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    
    @Column(name = "total_amount")
    private BigDecimal totalAmount;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    // Helper methods
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }
    
    // Getters and setters
}
```

### Example 2: Spring Data JPA Repository

```java
// Repository with custom query methods
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Query method from method name / Tạo query từ tên method
    Optional<User> findByUsername(String username);
    
    Optional<User> findByEmail(String email);
    
    List<User> findByStatus(UserStatus status);
    
    // Custom JPQL query - demonstrating LIKE with pattern
    @Query("SELECT u FROM User u WHERE u.email LIKE CONCAT('%@', :domain)")
    List<User> findByEmailDomain(@Param("domain") String domain);
    
    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE created_at > :date", 
           nativeQuery = true)
    List<User> findUsersCreatedAfter(@Param("date") LocalDateTime date);
    
    // JOIN FETCH to avoid N+1
    @Query("SELECT DISTINCT u FROM User u " +
           "LEFT JOIN FETCH u.orders " +
           "WHERE u.id = :id")
    Optional<User> findByIdWithOrders(@Param("id") Long id);
    
    // Projection
    @Query("SELECT u.username, COUNT(o) FROM User u " +
           "LEFT JOIN u.orders o " +
           "GROUP BY u.username")
    List<Object[]> getUserOrderCounts();
    
    // Modifying query
    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateUserStatus(@Param("id") Long id, @Param("status") UserStatus status);
    
    // Pagination and sorting
    Page<User> findByStatusOrderByCreatedAtDesc(UserStatus status, Pageable pageable);
}

// Service usage
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public User createUser(User user) {
        return userRepository.save(user);
    }
    
    public Optional<User> getUserByUsername(String username) {
        return userRepository.findByUsername(username);
    }
    
    public Page<User> getActiveUsers(int page, int size) {
        Pageable pageable = PageRequest.of(page, size, 
            Sort.by("createdAt").descending());
        return userRepository.findByStatusOrderByCreatedAtDesc(
            UserStatus.ACTIVE, pageable);
    }
    
    @Transactional(readOnly = true)
    public User getUserWithOrders(Long id) {
        return userRepository.findByIdWithOrders(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

### Example 3: MyBatis Mapper

```java
// Entity (POJO)
public class User {
    private Long id;
    private String username;
    private String email;
    private String passwordHash;
    private UserStatus status;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    // Getters and setters
}

// Mapper interface
@Mapper
public interface UserMapper {
    
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);
    
    @Select("SELECT * FROM users WHERE username = #{username}")
    User findByUsername(String username);
    
    @Insert("INSERT INTO users(username, email, password_hash, status, created_at) " +
            "VALUES(#{username}, #{email}, #{passwordHash}, #{status}, #{createdAt})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insert(User user);
    
    @Update("UPDATE users SET email=#{email}, updated_at=#{updatedAt} " +
            "WHERE id=#{id}")
    void update(User user);
    
    @Delete("DELETE FROM users WHERE id = #{id}")
    void delete(Long id);
    
    // Complex query with joins
    @Select("SELECT u.*, o.* FROM users u " +
            "LEFT JOIN orders o ON u.id = o.user_id " +
            "WHERE u.id = #{id}")
    @Results({
        @Result(property = "id", column = "id"),
        @Result(property = "username", column = "username"),
        @Result(property = "orders", javaType = List.class,
                column = "id",
                many = @Many(select = "findOrdersByUserId"))
    })
    User findByIdWithOrders(Long id);
    
    // Dynamic SQL using XML
    List<User> findByDynamicCriteria(UserSearchCriteria criteria);
}
```

```xml
<!-- UserMapper.xml for dynamic SQL -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">
    
    <!-- Result map for complex mapping -->
    <resultMap id="UserWithOrdersMap" type="User">
        <id property="id" column="user_id"/>
        <result property="username" column="username"/>
        <result property="email" column="email"/>
        <collection property="orders" ofType="Order">
            <id property="id" column="order_id"/>
            <result property="orderNumber" column="order_number"/>
            <result property="totalAmount" column="total_amount"/>
        </collection>
    </resultMap>
    
    <!-- Dynamic SQL query -->
    <select id="findByDynamicCriteria" resultType="User">
        SELECT * FROM users
        <where>
            <if test="username != null and username != ''">
                AND username LIKE CONCAT('%', #{username}, '%')
            </if>
            <if test="email != null and email != ''">
                AND email LIKE CONCAT('%', #{email}, '%')
            </if>
            <if test="status != null">
                AND status = #{status}
            </if>
            <if test="createdAfter != null">
                <![CDATA[AND created_at >= #{createdAfter}]]>
            </if>
        </where>
        ORDER BY created_at DESC
    </select>
    
    <!-- Batch insert -->
    <insert id="batchInsert" parameterType="list">
        INSERT INTO users(username, email, password_hash, status, created_at)
        VALUES
        <foreach collection="list" item="user" separator=",">
            (#{user.username}, #{user.email}, #{user.passwordHash}, 
             #{user.status}, #{user.createdAt})
        </foreach>
    </insert>
    
</mapper>
```

### Example 4: Solving N+1 Problem / Giải quyết vấn đề N+1

```java
// ❌ BAD: N+1 Problem / Vấn đề N+1
@Service
public class OrderService {
    
    public List<OrderDTO> getAllOrdersWithUsers() {
        List<Order> orders = orderRepository.findAll(); // 1 query / 1 truy vấn
        return orders.stream()
            .map(order -> {
                User user = order.getUser(); // N queries! / N truy vấn!
                return new OrderDTO(order, user.getUsername());
            })
            .collect(Collectors.toList());
    }
}

// ✅ GOOD: Solution 1 - JOIN FETCH / Giải pháp 1
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @Query("SELECT o FROM Order o JOIN FETCH o.user")
    List<Order> findAllWithUsers(); // Single query with join / 1 truy vấn với join
}

@Service
public class OrderService {
    
    public List<OrderDTO> getAllOrdersWithUsers() {
        List<Order> orders = orderRepository.findAllWithUsers(); // 1 query
        return orders.stream()
            .map(order -> new OrderDTO(order, order.getUser().getUsername()))
            .collect(Collectors.toList());
    }
}

// ✅ GOOD: Solution 2 - Entity Graph / Giải pháp 2
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @EntityGraph(attributePaths = {"user", "items"})
    List<Order> findAll();
}

// ✅ GOOD: Solution 3 - Batch Fetching (Hibernate) / Giải pháp 3
@Entity
public class Order {
    
    @ManyToOne(fetch = FetchType.LAZY)
    @BatchSize(size = 10) // Fetch in batches of 10 - tune based on typical data / Tải theo lô 10
    private User user;
}
```

### Example 5: Caching Configuration / Cấu hình Cache

```java
// Spring Boot configuration
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(
            new ConcurrentMapCache("users"),
            new ConcurrentMapCache("orders")
        ));
        return cacheManager;
    }
}

// Entity with second-level cache
@Entity
@Cacheable
@org.hibernate.annotations.Cache(
    usage = CacheConcurrencyStrategy.READ_WRITE,
    region = "users"
)
public class User {
    // Entity fields
}

// Repository with caching
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Cacheable(value = "users", key = "#username")
    Optional<User> findByUsername(String username);
    
    @CacheEvict(value = "users", key = "#user.username")
    User save(User user);
    
    @CacheEvict(value = "users", allEntries = true)
    void deleteAll();
}

// application.properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
spring.jpa.properties.hibernate.cache.use_query_cache=true
```

### Example 6: Transaction Management / Quản lý giao dịch

```java
// Declarative transaction management / Quản lý giao dịch khai báo
@Service
@Transactional
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private InventoryService inventoryService;
    
    // Method is transactional by default (class-level)
    // Method mặc định là transactional (cấp lớp)
    public Order createOrder(OrderDTO orderDTO) {
        Order order = new Order();
        // Map DTO to entity
        order = orderRepository.save(order);
        
        // If this throws exception, transaction rolls back
        // Nếu ném ngoại lệ, giao dịch sẽ rollback
        inventoryService.decreaseStock(orderDTO.getItems());
        
        return order;
    }
    
    // Read-only transaction for better performance
    // Giao dịch chỉ đọc cho hiệu suất tốt hơn
    @Transactional(readOnly = true)
    public Order getOrder(Long id) {
        return orderRepository.findById(id)
            .orElseThrow(() -> new OrderNotFoundException(id));
    }
    
    // Custom transaction settings / Cấu hình giao dịch tùy chỉnh
    @Transactional(
        isolation = Isolation.SERIALIZABLE,
        propagation = Propagation.REQUIRES_NEW,
        timeout = 30,
        rollbackFor = Exception.class
    )
    public void processPayment(Long orderId, PaymentDTO payment) {
        // Critical payment processing
    }
}

// Programmatic transaction management / Quản lý giao dịch theo chương trình
@Service
public class OrderService {
    
    @Autowired
    private TransactionTemplate transactionTemplate;
    
    public Order createOrder(OrderDTO orderDTO) {
        return transactionTemplate.execute(status -> {
            try {
                Order order = new Order();
                order = orderRepository.save(order);
                inventoryService.decreaseStock(orderDTO.getItems());
                return order;
            } catch (Exception e) {
                status.setRollbackOnly();
                throw e;
            }
        });
    }
}
```

---

## Best Practices / Thực hành tốt nhất

### 1. Entity Design / Thiết kế Entity
✅ **Do / Nên:**
- Use appropriate fetch types (LAZY by default) - Sử dụng fetch type phù hợp (LAZY mặc định)
- Implement equals() and hashCode() properly - Triển khai equals() và hashCode() đúng cách
- Use @Version for optimistic locking - Dùng @Version cho khóa lạc quan
- Keep entities simple and focused - Giữ entity đơn giản và tập trung

❌ **Don't / Không nên:**
- Use EAGER loading everywhere - Dùng EAGER loading ở mọi nơi
- Create deep entity hierarchies - Tạo cấu trúc entity sâu
- Put business logic in entities - Đặt logic nghiệp vụ trong entity
- Expose entities directly to API - Expose entity trực tiếp ra API

### 2. Performance / Hiệu suất
✅ **Do / Nên:**
- Use JOIN FETCH to avoid N+1 problem - Dùng JOIN FETCH tránh vấn đề N+1
- Enable second-level cache for read-heavy entities - Bật cache cấp 2 cho entity đọc nhiều
- Use pagination for large result sets - Dùng phân trang cho tập kết quả lớn
- Profile and monitor query performance - Theo dõi hiệu suất truy vấn
- Use batch processing for bulk operations - Dùng xử lý batch cho thao tác hàng loạt

❌ **Don't / Không nên:**
- Load entire collections eagerly - Tải toàn bộ collection eagerly
- Ignore the N+1 problem - Bỏ qua vấn đề N+1
- Use * in SELECT queries - Dùng * trong SELECT
- Fetch more data than needed - Tải nhiều dữ liệu hơn cần thiết

### 3. Transactions / Giao dịch
✅ **Do / Nên:**
- Keep transactions short - Giữ giao dịch ngắn
- Use read-only transactions for queries - Dùng giao dịch read-only cho truy vấn
- Handle exceptions properly - Xử lý ngoại lệ đúng cách
- Use appropriate isolation levels - Dùng mức cô lập phù hợp

❌ **Don't / Không nên:**
- Make external API calls within transactions - Gọi API bên ngoài trong giao dịch
- Use write transactions for read-only operations - Dùng giao dịch ghi cho thao tác chỉ đọc
- Ignore transaction boundaries - Bỏ qua ranh giới giao dịch
- Mix transactional and non-transactional code - Trộn code có và không có giao dịch

### 4. Querying / Truy vấn
✅ **Do / Nên:**
- Use Criteria API for dynamic queries - Dùng Criteria API cho truy vấn động
- Use native SQL when necessary - Dùng SQL gốc khi cần thiết
- Index frequently queried columns - Đánh index các cột hay truy vấn
- Use projections to fetch only needed data - Dùng projection chỉ lấy dữ liệu cần

❌ **Don't / Không nên:**
- Use native SQL for simple queries - Dùng SQL gốc cho truy vấn đơn giản
- Concatenate strings to build queries (SQL injection) - Nối chuỗi tạo truy vấn (SQL injection)
- Fetch entire entities when only few fields needed - Tải toàn bộ entity khi chỉ cần vài field
- Ignore database indexes - Bỏ qua index database

### 5. Caching
✅ **Do / Nên:**
- Cache immutable or rarely changing data - Cache dữ liệu không đổi hoặc ít thay đổi
- Use appropriate cache strategies - Dùng chiến lược cache phù hợp
- Monitor cache hit/miss ratios - Theo dõi tỷ lệ hit/miss của cache
- Set proper cache expiration - Đặt thời gian hết hạn cache đúng

❌ **Don't / Không nên:**
- Cache everything - Cache mọi thứ
- Ignore cache invalidation - Bỏ qua việc vô hiệu hóa cache
- Use caching for frequently changing data - Dùng cache cho dữ liệu thay đổi thường xuyên
- Share cache between environments - Chia sẻ cache giữa các môi trường

### 6. Maintenance / Bảo trì
✅ **Do / Nên:**
- Use database migration tools (Flyway, Liquibase) - Dùng công cụ migration (Flyway, Liquibase)
- Version your database schema - Quản lý phiên bản schema database
- Keep ORM framework updated - Cập nhật framework ORM
- Document complex mappings - Tài liệu hóa ánh xạ phức tạp

❌ **Don't / Không nên:**
- Use hbm2ddl.auto=update in production - Dùng hbm2ddl.auto=update trong production
- Modify schema manually - Sửa schema thủ công
- Skip database migrations - Bỏ qua migration database
- Leave orphaned data - Để lại dữ liệu mồ côi

---

## Common Pitfalls / Lỗi thường gặp

### 1. LazyInitializationException
```java
// ❌ Problem / Vấn đề
@Transactional
public User getUser(Long id) {
    return userRepository.findById(id).orElse(null);
}

// In controller - session is closed! / Trong controller - session đã đóng!
public UserDTO getUser(Long id) {
    User user = userService.getUser(id);
    return new UserDTO(user, user.getOrders()); // LazyInitializationException!
}

// ✅ Solution 1: Fetch in transaction / Giải pháp 1: Tải trong giao dịch
@Transactional(readOnly = true)
public UserDTO getUser(Long id) {
    User user = userRepository.findByIdWithOrders(id).orElse(null);
    return new UserDTO(user, user.getOrders());
}

// ✅ Solution 2: Use DTO projection / Giải pháp 2: Dùng DTO projection
@Query("SELECT new com.example.UserDTO(u.id, u.username, o.count) " +
       "FROM User u LEFT JOIN u.orders o WHERE u.id = :id")
UserDTO findUserDTO(@Param("id") Long id);
```

### 2. Bi-directional Relationship Issues / Vấn đề quan hệ hai chiều
```java
// ❌ Problem: Infinite recursion in JSON / Vấn đề: Đệ quy vô hạn trong JSON
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}

@Entity
public class Order {
    @ManyToOne
    private User user;
}

// ✅ Solution: Use @JsonManagedReference and @JsonBackReference
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @JsonManagedReference
    private List<Order> orders;
}

@Entity
public class Order {
    @ManyToOne
    @JsonBackReference
    private User user;
}

// ✅ Better solution: Use DTOs / Giải pháp tốt hơn: Dùng DTO
public class UserDTO {
    private Long id;
    private String username;
    private List<OrderSummaryDTO> orders;
}
```

### 3. Incorrect Transaction Boundaries / Ranh giới giao dịch sai
```java
// ❌ Problem: Transaction per query / Vấn đề: Giao dịch mỗi truy vấn
public void processOrders() {
    List<Order> orders = orderService.findPending(); // Transaction 1
    for (Order order : orders) {
        orderService.process(order); // Transaction 2, 3, 4...
    }
}

// ✅ Solution: Single transaction / Giải pháp: Giao dịch đơn
@Transactional
public void processOrders() {
    List<Order> orders = orderRepository.findByStatus(PENDING);
    for (Order order : orders) {
        order.setStatus(PROCESSED);
        orderRepository.save(order);
    }
}
```

---

## Resources / Tài liệu tham khảo

### Official Documentation / Tài liệu chính thức
- [Hibernate Documentation](https://hibernate.org/orm/documentation/)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
- [MyBatis Documentation](https://mybatis.org/mybatis-3/)
- [JPA Specification](https://jakarta.ee/specifications/persistence/)

### Books / Sách
- **Java Persistence with Hibernate** - Christian Bauer, Gavin King
- **High-Performance Java Persistence** - Vlad Mihalcea
- **Pro JPA 2** - Mike Keith, Merrick Schincariol
- **Spring Data** - Mark Pollack, Oliver Gierke

### Online Resources / Tài liệu trực tuyến
- [Baeldung ORM Tutorials](https://www.baeldung.com/learn-jpa-hibernate)
- [Vlad Mihalcea's Blog](https://vladmihalcea.com/)
- [Thoughts on Java](https://thorben-janssen.com/)

### Practice / Thực hành
- Build a complete CRUD application with each framework - Xây dựng ứng dụng CRUD hoàn chỉnh với mỗi framework
- Implement caching strategies - Triển khai chiến lược caching
- Optimize queries and solve N+1 problems - Tối ưu truy vấn và giải quyết vấn đề N+1
- Compare performance between frameworks - So sánh hiệu suất giữa các framework
- Implement complex relationships and mappings - Triển khai quan hệ và ánh xạ phức tạp

---

## Summary / Tóm tắt

ORM frameworks are essential tools for Java developers, providing abstraction over database operations and improving productivity. Understanding the core concepts—**persistence, transactions, caching, and fetching strategies**—is crucial for effective ORM usage.

**Tiếng Việt:** Các framework ORM là công cụ thiết yếu cho lập trình viên Java, cung cấp lớp trừu tượng cho các thao tác cơ sở dữ liệu và cải thiện năng suất. Hiểu các khái niệm cốt lõi—**persistence, transactions, caching, và các chiến lược fetching**—là rất quan trọng để sử dụng ORM hiệu quả.

**Key Takeaways / Điểm chính:**
- **Hibernate**: Best for enterprise applications needing full JPA compliance with extensions - Tốt nhất cho ứng dụng doanh nghiệp cần tuân thủ JPA đầy đủ với extension
- **Spring Data JPA**: Best for Spring Boot applications prioritizing development speed - Tốt nhất cho ứng dụng Spring Boot ưu tiên tốc độ phát triển
- **MyBatis**: Best for applications requiring fine SQL control and complex queries - Tốt nhất cho ứng dụng yêu cầu kiểm soát SQL chi tiết và truy vấn phức tạp
- **Hybrid approach**: Combine frameworks based on specific use case requirements - Tiếp cận kết hợp: Kết hợp framework dựa trên yêu cầu cụ thể

Choose the right tool for the job, understand the trade-offs, and always optimize for your specific use case!

**Tiếng Việt:** Chọn công cụ phù hợp cho công việc, hiểu các đánh đổi, và luôn tối ưu hóa cho trường hợp sử dụng cụ thể của bạn!
