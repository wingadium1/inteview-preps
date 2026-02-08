# Câu Hỏi Phỏng Vấn Hibernate / Hibernate Interview Questions

## Mục Lục / Table of Contents
1. [Câu Hỏi Cơ Bản](#câu-hỏi-cơ-bản--basic-questions)
2. [Câu Hỏi Về N+1 Problem](#câu-hỏi-về-n1-problem--n1-problem-questions)
3. [Câu Hỏi Về Performance & Optimization](#câu-hỏi-về-performance--optimization)
4. [Câu Hỏi Về Caching](#câu-hỏi-về-caching)
5. [Câu Hỏi Về Relationships & Mapping](#câu-hỏi-về-relationships--mapping)
6. [Câu Hỏi Nâng Cao](#câu-hỏi-nâng-cao--advanced-questions)

---

## Câu Hỏi Cơ Bản / Basic Questions

### 1. Hibernate là gì và những lợi ích chính của nó là gì?
**What is Hibernate and what are its main advantages?**

**Trả lời / Answer:**
Hibernate là một framework ORM (Object-Relational Mapping) cho Java, giúp map các đối tượng Java với các bảng trong cơ sở dữ liệu quan hệ.

Lợi ích chính:
- **Database Independence**: Độc lập với cơ sở dữ liệu
- **Giảm code SQL**: Tự động sinh SQL queries
- **Caching**: Hỗ trợ first-level và second-level cache
- **Lazy Loading**: Load dữ liệu khi cần thiết
- **Transaction Management**: Quản lý transaction hiệu quả
- **HQL**: Object-oriented query language

---

### 2. Sự khác biệt giữa save() và persist() là gì?
**What is the difference between save() and persist()?**

**Trả lời / Answer:**

| save() | persist() |
|--------|-----------|
| Hibernate-specific method | JPA standard method |
| Trả về generated identifier | Không trả về gì (void) |
| Có thể được gọi bên ngoài transaction | Phải được gọi trong transaction |
| Có thể save một detached object | Chỉ save transient object |

```java
// save() example
Long id = (Long) session.save(user); // Returns generated ID

// persist() example
session.persist(user); // Void, no return
Long id = user.getId(); // Get ID from object
```

---

### 3. Sự khác biệt giữa get() và load() là gì?
**What is the difference between get() and load()?**

**Trả lời / Answer:**

| get() | load() |
|-------|--------|
| Hits database ngay lập tức | Trả về proxy object (lazy) |
| Trả về null nếu không tìm thấy | Throw ObjectNotFoundException nếu không tìm thấy |
| Sử dụng khi bạn không chắc object tồn tại | Sử dụng khi bạn chắc object tồn tại |

```java
// get() - Immediate database hit
User user = session.get(User.class, 1L);
if (user == null) {
    System.out.println("User not found");
}

// load() - Returns proxy
User user = session.load(User.class, 1L);
// Database hit happens when you access properties
String name = user.getName(); // Database hit here!
```

---

### 4. Các trạng thái của một Hibernate entity là gì?
**What are the different states of a Hibernate entity?**

**Trả lời / Answer:**

1. **Transient**: Object mới tạo, chưa associate với Session
2. **Persistent**: Object được quản lý bởi Session và có representation trong database
3. **Detached**: Object từng là persistent nhưng Session đã closed
4. **Removed**: Object được đánh dấu để xóa

```java
// Transient
User user = new User();

// Persistent
session.save(user);

// Detached
session.close();

// Removed
session.delete(user);
```

---

## Câu Hỏi Về N+1 Problem / N+1 Problem Questions

### 5. N+1 Problem là gì? Tại sao nó là vấn đề nghiêm trọng trong Hibernate?
**What is the N+1 Problem? Why is it a serious issue in Hibernate?**

**Trả lời / Answer:**

N+1 Problem xảy ra khi Hibernate thực hiện 1 query để lấy N records, sau đó thực hiện thêm N queries riêng biệt để lấy data related cho mỗi record.

**Ví dụ vấn đề:**
```java
// 1 query to get all users
List<User> users = session.createQuery("FROM User", User.class).list();

// N additional queries - one for each user's orders!
for (User user : users) {
    List<Order> orders = user.getOrders(); // Lazy loading triggers query
    System.out.println("User: " + user.getName() + ", Orders: " + orders.size());
}
```

**Kết quả:**
```sql
-- 1 query cho users
SELECT * FROM users;

-- N queries cho orders (nếu có 100 users = 100 queries!)
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2;
SELECT * FROM orders WHERE user_id = 3;
-- ... 97 queries nữa
```

**Vấn đề:**
- Performance degradation nghiêm trọng
- Tăng database load
- Slow response time
- Network overhead

---

### 6. Làm thế nào để phát hiện N+1 Problem trong ứng dụng?
**How do you detect the N+1 Problem in your application?**

**Trả lời / Answer:**

**Các cách phát hiện:**

1. **Enable SQL Logging**
```properties
# application.properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

2. **Use Hibernate Statistics**
```java
SessionFactory sessionFactory = ...;
Statistics stats = sessionFactory.getStatistics();
stats.setStatisticsEnabled(true);

// After operation
System.out.println("Queries executed: " + stats.getQueryExecutionCount());
System.out.println("Entities loaded: " + stats.getEntityLoadCount());
```

3. **Performance Monitoring Tools**
- Hibernate Query Analyzer
- P6Spy
- JProfiler
- YourKit

4. **Database Query Logs**
- Monitor slow query logs
- Use database profiling tools

**Ví dụ output khi có N+1:**
```
Hibernate: SELECT * FROM users
Hibernate: SELECT * FROM orders WHERE user_id = ?
Hibernate: SELECT * FROM orders WHERE user_id = ?
Hibernate: SELECT * FROM orders WHERE user_id = ?
... (nhiều queries tương tự)
```

---

### 7. Các giải pháp để khắc phục N+1 Problem là gì?
**What are the solutions to fix the N+1 Problem?**

**Trả lời / Answer:**

#### **Giải pháp 1: JOIN FETCH trong HQL/JPQL**
```java
// Thay vì lazy loading
String hql = "SELECT u FROM User u JOIN FETCH u.orders";
List<User> users = session.createQuery(hql, User.class).list();
// Chỉ 1 query với JOIN!

// Hoặc với WHERE clause
String hql = "SELECT u FROM User u JOIN FETCH u.orders WHERE u.active = true";
```

**SQL Generated:**
```sql
SELECT u.*, o.* 
FROM users u 
INNER JOIN orders o ON u.id = o.user_id;
```

#### **Giải pháp 2: @EntityGraph (JPA 2.1+)**
```java
@Entity
@NamedEntityGraph(
    name = "User.orders",
    attributeNodes = @NamedAttributeNode("orders")
)
public class User {
    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}

// Sử dụng
EntityGraph<?> graph = em.getEntityGraph("User.orders");
Map<String, Object> hints = new HashMap<>();
hints.put("javax.persistence.fetchgraph", graph);
User user = em.find(User.class, userId, hints);
```

#### **Giải pháp 3: Criteria API với Fetch**
```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> root = cq.from(User.class);
root.fetch("orders", JoinType.LEFT); // Fetch orders

List<User> users = session.createQuery(cq).getResultList();
```

#### **Giải pháp 4: @BatchSize Annotation**
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @BatchSize(size = 10)
    private List<Order> orders;
}
```

**Thay vì N queries, sẽ có N/10 queries:**
```sql
SELECT * FROM orders WHERE user_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
SELECT * FROM orders WHERE user_id IN (11, 12, 13, 14, 15, 16, 17, 18, 19, 20);
```

#### **Giải pháp 5: Subselect Fetching**
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @Fetch(FetchMode.SUBSELECT)
    private List<Order> orders;
}
```

**SQL Generated:**
```sql
-- First query
SELECT * FROM users WHERE ...;

-- Second query with subselect (only 1 additional query!)
SELECT * FROM orders WHERE user_id IN (
    SELECT id FROM users WHERE ...
);
```

---

### 8. So sánh JOIN FETCH vs EAGER loading để giải quyết N+1 Problem
**Compare JOIN FETCH vs EAGER loading for solving N+1 Problem**

**Trả lời / Answer:**

| Aspect | JOIN FETCH | EAGER Loading |
|--------|------------|---------------|
| **Khi nào apply** | Per-query basis | Toàn bộ entity |
| **Flexibility** | Flexible, có thể chọn khi nào fetch | Fixed behavior |
| **Performance** | Tốt hơn, chỉ fetch khi cần | Có thể over-fetch |
| **Cartesian Product** | Có thể xảy ra với multiple fetches | Có thể xảy ra |
| **Control** | Developer có full control | Automatic |

**Ví dụ vấn đề với EAGER:**
```java
@Entity
public class User {
    @OneToMany(fetch = FetchType.EAGER)
    private List<Order> orders;
    
    @OneToMany(fetch = FetchType.EAGER)
    private List<Address> addresses;
}

// Mỗi lần load User, luôn fetch orders và addresses
// Ngay cả khi bạn không cần!
User user = session.get(User.class, 1L);
// Orders và Addresses đã được loaded
```

**Best Practice:**
- Sử dụng LAZY loading mặc định
- Dùng JOIN FETCH khi cần eager load
- Tránh EAGER loading trừ khi thực sự cần thiết

---

### 9. N+1 Problem có thể xảy ra với Eager Loading không?
**Can the N+1 Problem occur with Eager Loading?**

**Trả lời / Answer:**

**Có! N+1 vẫn có thể xảy ra ngay cả với EAGER loading.**

**Ví dụ:**
```java
@Entity
public class User {
    @OneToMany(fetch = FetchType.EAGER)
    private List<Order> orders;
}

@Entity  
public class Order {
    @ManyToOne(fetch = FetchType.EAGER)
    private Product product;
}

// Query users
List<User> users = session.createQuery("FROM User", User.class).list();
```

**Kết quả:**
```sql
-- 1 query cho users
SELECT * FROM users;

-- N queries cho orders (mỗi user 1 query)
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2;
-- ...

-- Thêm M queries cho products
SELECT * FROM products WHERE id = 101;
SELECT * FROM products WHERE id = 102;
-- ...
```

**Giải pháp:**
```java
// Sử dụng JOIN FETCH cho multiple levels
String hql = """
    SELECT DISTINCT u 
    FROM User u 
    JOIN FETCH u.orders o 
    JOIN FETCH o.product
""";
List<User> users = session.createQuery(hql, User.class).list();
```

---

### 10. Làm thế nào để xử lý N+1 Problem với pagination?
**How do you handle the N+1 Problem with pagination?**

**Trả lời / Answer:**

Pagination với JOIN FETCH có thể phức tạp vì Hibernate không thể apply LIMIT trực tiếp trong SQL.

**Vấn đề:**
```java
// Hibernate sẽ warning về memory issue!
String hql = "SELECT u FROM User u JOIN FETCH u.orders";
List<User> users = session.createQuery(hql, User.class)
    .setFirstResult(0)
    .setMaxResults(10)
    .list();

// WARNING: firstResult/maxResults specified with collection fetch; 
// applying in memory!
```

**Giải pháp 1: Two-Step Loading**
```java
// Step 1: Load user IDs với pagination
String hql = "SELECT u.id FROM User u";
List<Long> userIds = session.createQuery(hql, Long.class)
    .setFirstResult(0)
    .setMaxResults(10)
    .list();

// Step 2: Load users với orders bằng IDs
if (!userIds.isEmpty()) {
    String fetchHql = """
        SELECT DISTINCT u 
        FROM User u 
        JOIN FETCH u.orders 
        WHERE u.id IN :ids
    """;
    List<User> users = session.createQuery(fetchHql, User.class)
        .setParameter("ids", userIds)
        .list();
}
```

**Giải pháp 2: @BatchSize**
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @BatchSize(size = 10)
    private List<Order> orders;
}

// Pagination query
List<User> users = session.createQuery("FROM User", User.class)
    .setFirstResult(0)
    .setMaxResults(10)
    .list();
// Orders sẽ được fetched trong batches
```

---

## Câu Hỏi Về Performance & Optimization

### 11. Làm thế nào để optimize performance của Hibernate queries?
**How do you optimize Hibernate query performance?**

**Trả lời / Answer:**

**1. Tránh N+1 Problem**
- Sử dụng JOIN FETCH
- BatchSize annotation
- EntityGraph

**2. Sử dụng Projection cho queries chỉ cần một số fields**
```java
// Thay vì load toàn bộ entity
String hql = "SELECT new UserDTO(u.id, u.name, u.email) FROM User u";
List<UserDTO> users = session.createQuery(hql, UserDTO.class).list();
```

**3. Pagination**
```java
query.setFirstResult(0).setMaxResults(20);
```

**4. Query Hints**
```java
// Read-only entities
query.setReadOnly(true);

// Query timeout
query.setTimeout(30);
```

**5. Batch Processing**
```java
int batchSize = 50;
for (int i = 0; i < objects.size(); i++) {
    session.save(objects.get(i));
    if (i % batchSize == 0) {
        session.flush();
        session.clear();
    }
}
```

**6. Proper Indexing**
- Index foreign keys
- Index frequently queried columns
- Composite indexes cho multi-column queries

**7. Connection Pooling**
```properties
hibernate.hikari.maximumPoolSize=20
hibernate.hikari.minimumIdle=5
```

---

### 12. LazyInitializationException là gì và làm thế nào để giải quyết?
**What is LazyInitializationException and how do you solve it?**

**Trả lời / Answer:**

**LazyInitializationException** xảy ra khi bạn cố access một lazy-loaded collection sau khi Hibernate Session đã closed.

**Ví dụ lỗi:**
```java
User user;
try (Session session = sessionFactory.openSession()) {
    user = session.get(User.class, 1L);
} // Session closed here

// Exception! Session is closed
List<Order> orders = user.getOrders();
// org.hibernate.LazyInitializationException: 
// could not initialize proxy - no Session
```

**Giải pháp:**

**1. Fetch data trong Session**
```java
try (Session session = sessionFactory.openSession()) {
    User user = session.get(User.class, 1L);
    // Access lazy collection before closing
    user.getOrders().size(); // Initialize collection
    return user;
}
```

**2. JOIN FETCH**
```java
String hql = "SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id";
User user = session.createQuery(hql, User.class)
    .setParameter("id", 1L)
    .uniqueResult();
```

**3. Open Session In View (Anti-pattern cho production)**
```properties
spring.jpa.open-in-view=true
```

**4. DTO Projection**
```java
String hql = "SELECT new UserDTO(u.id, u.name, SIZE(u.orders)) FROM User u";
```

**5. Hibernate.initialize()**
```java
User user = session.get(User.class, 1L);
Hibernate.initialize(user.getOrders());
```

---

## Câu Hỏi Về Caching

### 13. Sự khác biệt giữa First-level và Second-level cache là gì?
**What is the difference between First-level and Second-level cache?**

**Trả lời / Answer:**

| Aspect | First-level Cache | Second-level Cache |
|--------|------------------|-------------------|
| **Scope** | Session-level | SessionFactory-level |
| **Default** | Enabled (không thể disable) | Disabled (phải config) |
| **Lifetime** | Tồn tại trong Session | Tồn tại across Sessions |
| **Data** | Entity instances | Entity data (dehydrated state) |
| **Thread-safe** | Không | Có |

**First-level Cache Example:**
```java
Session session = sessionFactory.openSession();

User user1 = session.get(User.class, 1L); // Database hit
User user2 = session.get(User.class, 1L); // Cache hit - no DB query!
System.out.println(user1 == user2); // true - same instance

session.close();
```

**Second-level Cache Example:**
```java
// Session 1
Session session1 = sessionFactory.openSession();
User user1 = session1.get(User.class, 1L); // DB hit, stores in L2 cache
session1.close();

// Session 2
Session session2 = sessionFactory.openSession();
User user2 = session2.get(User.class, 1L); // L2 cache hit!
session2.close();

System.out.println(user1 == user2); // false - different instances
System.out.println(user1.equals(user2)); // true - same data
```

**Configuration:**
```properties
# Enable second-level cache
hibernate.cache.use_second_level_cache=true
hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    // ...
}
```

---

### 14. Query Cache là gì và khi nào nên sử dụng?
**What is Query Cache and when should you use it?**

**Trả lời / Answer:**

**Query Cache** lưu trữ result sets của queries, không chỉ entities.

**Configuration:**
```properties
hibernate.cache.use_query_cache=true
```

**Usage:**
```java
// Enable query cache cho specific query
Query<User> query = session.createQuery("FROM User WHERE status = :status", User.class);
query.setParameter("status", "ACTIVE");
query.setCacheable(true);
query.setCacheRegion("userCache"); // Optional
List<User> users = query.list();
```

**Khi nào sử dụng:**
- ✅ Queries thực thi thường xuyên với parameters giống nhau
- ✅ Data ít thay đổi
- ✅ Read-heavy operations

**Khi nào KHÔNG nên sử dụng:**
- ❌ Data thay đổi thường xuyên
- ❌ Queries với dynamic parameters
- ❌ Large result sets

**Lưu ý quan trọng:**
```java
// Query cache stores IDs, not full entities
// First query - hits database
List<User> users1 = query.list(); 

// Second query - gets IDs from cache
// Then loads entities from second-level cache or database
List<User> users2 = query.list();
```

---

## Câu Hỏi Về Relationships & Mapping

### 15. Cascade types trong Hibernate là gì? Khi nào sử dụng mỗi loại?
**What are Cascade types in Hibernate? When to use each?**

**Trả lời / Answer:**

**Cascade Types:**

1. **CascadeType.PERSIST** - Lưu child khi save parent
```java
@OneToMany(cascade = CascadeType.PERSIST)
private List<Order> orders;

// Save user sẽ tự động save orders
User user = new User();
user.addOrder(new Order());
session.persist(user); // Orders also persisted
```

2. **CascadeType.MERGE** - Update child khi update parent
```java
@OneToMany(cascade = CascadeType.MERGE)
private List<Order> orders;

user.getOrders().get(0).setStatus("COMPLETED");
session.merge(user); // Order status updated
```

3. **CascadeType.REMOVE** - Delete child khi delete parent
```java
@OneToMany(cascade = CascadeType.REMOVE)
private List<Order> orders;

session.delete(user); // All orders deleted
```

4. **CascadeType.REFRESH** - Refresh child khi refresh parent
```java
@OneToMany(cascade = CascadeType.REFRESH)
private List<Order> orders;

session.refresh(user); // Orders refreshed from DB
```

5. **CascadeType.DETACH** - Detach child khi detach parent
```java
@OneToMany(cascade = CascadeType.DETACH)
private List<Order> orders;

session.detach(user); // Orders also detached
```

6. **CascadeType.ALL** - Tất cả các operations trên
```java
@OneToMany(cascade = CascadeType.ALL)
private List<Order> orders;
```

**Best Practices:**
- Sử dụng `CascadeType.ALL` cho owned relationships (composition)
- Cẩn thận với `CascadeType.REMOVE` - có thể delete unintentional data
- Không cascade cho `@ManyToOne` hoặc `@ManyToMany` trong hầu hết cases

---

### 16. orphanRemoval vs CascadeType.REMOVE khác nhau như thế nào?
**What is the difference between orphanRemoval and CascadeType.REMOVE?**

**Trả lời / Answer:**

| orphanRemoval = true | CascadeType.REMOVE |
|---------------------|-------------------|
| Delete child khi remove from collection | Delete child khi delete parent |
| Chỉ áp dụng cho `@OneToOne`, `@OneToMany` | Áp dụng cho tất cả relationships |
| Child entity bị delete khi không còn parent | Child chỉ bị delete khi parent bị delete |

**Ví dụ orphanRemoval:**
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", orphanRemoval = true)
    private List<Order> orders;
}

// Remove order from collection
user.getOrders().remove(order);
session.merge(user);
// Order is DELETED from database (orphan removal)
```

**Ví dụ CascadeType.REMOVE:**
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", cascade = CascadeType.REMOVE)
    private List<Order> orders;
}

// Remove order from collection
user.getOrders().remove(order);
session.merge(user);
// Order is NOT deleted (just dissociated)

// Delete user
session.delete(user);
// NOW orders are deleted (cascade remove)
```

**Kết hợp cả hai:**
```java
@OneToMany(
    mappedBy = "user", 
    cascade = CascadeType.ALL, 
    orphanRemoval = true
)
private List<Order> orders;
// Provides maximum control over child lifecycle
```

---

## Câu Hỏi Nâng Cao / Advanced Questions

### 17. Optimistic vs Pessimistic Locking - khi nào sử dụng?
**Optimistic vs Pessimistic Locking - when to use each?**

**Trả lời / Answer:**

**Optimistic Locking:**
```java
@Entity
public class Product {
    @Id
    private Long id;
    
    @Version
    private Long version;
    
    private Integer stock;
}

// Usage
Product product = session.get(Product.class, 1L); // version = 1
product.setStock(product.getStock() - 1);
session.update(product);
// SQL: UPDATE product SET stock = ?, version = 2 WHERE id = ? AND version = 1
// If version changed, throws OptimisticLockException
```

**Pessimistic Locking:**
```java
// Pessimistic Read Lock (shared lock)
Product product = session.get(Product.class, 1L, 
    LockMode.PESSIMISTIC_READ);
// SQL: SELECT ... FOR SHARE

// Pessimistic Write Lock (exclusive lock)
Product product = session.get(Product.class, 1L, 
    LockMode.PESSIMISTIC_WRITE);
// SQL: SELECT ... FOR UPDATE
```

**Khi nào sử dụng:**

| Scenario | Optimistic | Pessimistic |
|----------|-----------|-------------|
| **Read-heavy** | ✅ | ❌ |
| **Write-heavy** | ❌ | ✅ |
| **Low contention** | ✅ | ❌ |
| **High contention** | ❌ | ✅ |
| **Long transactions** | ❌ | ❌ (deadlock risk) |
| **Short transactions** | ✅ | ✅ |

---

### 18. Inheritance Mapping strategies trong Hibernate là gì?
**What are the Inheritance Mapping strategies in Hibernate?**

**Trả lời / Answer:**

**1. Single Table (SINGLE_TABLE)**
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "type")
public abstract class Vehicle {
    @Id
    private Long id;
    private String brand;
}

@Entity
@DiscriminatorValue("CAR")
public class Car extends Vehicle {
    private Integer doors;
}

@Entity
@DiscriminatorValue("BIKE")
public class Bike extends Vehicle {
    private String bikeType;
}
```

**Bảng:**
```sql
CREATE TABLE vehicle (
    id BIGINT,
    type VARCHAR(31), -- Discriminator
    brand VARCHAR(255),
    doors INTEGER,    -- Nullable for Bike
    bike_type VARCHAR(255) -- Nullable for Car
);
```

**Pros:** Performance tốt, chỉ 1 table, no joins
**Cons:** Nullable columns, data integrity issues

---

**2. Joined Table (JOINED)**
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Vehicle {
    @Id
    private Long id;
    private String brand;
}

@Entity
public class Car extends Vehicle {
    private Integer doors;
}
```

**Bảng:**
```sql
CREATE TABLE vehicle (id BIGINT, brand VARCHAR(255));
CREATE TABLE car (id BIGINT, doors INTEGER, 
                  FOREIGN KEY (id) REFERENCES vehicle(id));
```

**Pros:** Normalized, no nullable columns
**Cons:** Requires joins, slower performance

---

**3. Table Per Class (TABLE_PER_CLASS)**
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Vehicle {
    @Id
    private Long id;
    private String brand;
}
```

**Bảng:**
```sql
CREATE TABLE car (id BIGINT, brand VARCHAR(255), doors INTEGER);
CREATE TABLE bike (id BIGINT, brand VARCHAR(255), bike_type VARCHAR(255));
```

**Pros:** Simple, no joins, no discriminator
**Cons:** Polymorphic queries are slow, data duplication

---

### 19. Custom Hibernate UserType là gì và khi nào cần implement?
**What is a Custom Hibernate UserType and when do you need to implement it?**

**Trả lời / Answer:**

**UserType** cho phép bạn map custom Java types to database columns.

**Khi nào cần:**
- Map complex types (e.g., JSON, XML)
- Encrypt/decrypt data automatically
- Convert between different representations

**Ví dụ - JSON Mapping:**
```java
public class JsonType implements UserType<Map<String, Object>> {
    
    @Override
    public int getSqlType() {
        return Types.VARCHAR;
    }
    
    @Override
    public Class<Map<String, Object>> returnedClass() {
        return (Class) Map.class;
    }
    
    @Override
    public Map<String, Object> nullSafeGet(ResultSet rs, int position,
                                           SharedSessionContractImplementor session, 
                                           Object owner) throws SQLException {
        String json = rs.getString(position);
        if (json == null) return null;
        
        try {
            return new ObjectMapper().readValue(json, Map.class);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    
    @Override
    public void nullSafeSet(PreparedStatement st, Map<String, Object> value, 
                           int index, SharedSessionContractImplementor session) 
                           throws SQLException {
        if (value == null) {
            st.setNull(index, Types.VARCHAR);
        } else {
            try {
                String json = new ObjectMapper().writeValueAsString(value);
                st.setString(index, json);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

**Usage:**
```java
@Entity
public class User {
    @Id
    private Long id;
    
    @Type(JsonType.class)
    @Column(columnDefinition = "TEXT")
    private Map<String, Object> metadata;
}
```

---

### 20. StatelessSession là gì và khi nào sử dụng?
**What is StatelessSession and when to use it?**

**Trả lời / Answer:**

**StatelessSession** là lightweight session không có first-level cache và không track entity changes.

**Đặc điểm:**
- Không có first-level cache
- Không có dirty checking
- Không có cascade
- Không có lazy loading
- Faster performance cho batch operations

**Khi nào sử dụng:**
- Batch inserts/updates
- Read-only operations  
- ETL processes
- Report generation

**Ví dụ:**
```java
// Regular Session - slower for bulk ops
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();
for (int i = 0; i < 100000; i++) {
    User user = new User("user" + i);
    session.save(user);
    if (i % 50 == 0) {
        session.flush();
        session.clear(); // Must clear cache manually
    }
}
tx.commit();

// StatelessSession - faster
StatelessSession session = sessionFactory.openStatelessSession();
Transaction tx = session.beginTransaction();
for (int i = 0; i < 100000; i++) {
    User user = new User("user" + i);
    session.insert(user); // insert, not save
}
tx.commit();
```

**Lưu ý:**
- Không có cascade - phải save manually
- Không có version checking
- Associations không được managed

---

## Best Practices Summary

### ✅ DO:
1. Sử dụng LAZY loading mặc định
2. Tránh N+1 với JOIN FETCH hoặc BatchSize
3. Enable SQL logging trong development
4. Sử dụng transactions
5. Close sessions properly (try-with-resources)
6. Index foreign keys và frequently queried columns
7. Sử dụng projection cho readonly queries
8. Monitor query performance
9. Use connection pooling
10. Implement proper exception handling

### ❌ DON'T:
1. Không sử dụng EAGER loading everywhere
2. Không ignore N+1 warnings
3. Không skip transaction boundaries
4. Không over-cache
5. Không forget to close sessions
6. Không sử dụng SELECT * queries
7. Không cascade operations carelessly
8. Không skip performance testing
9. Không ignore LazyInitializationException
10. Không disable first-level cache

---

## Kết Luận / Conclusion

Hibernate là một công cụ mạnh mẽ nhưng cũng cần hiểu sâu để sử dụng hiệu quả. N+1 Problem là một trong những vấn đề phổ biến nhất và nghiêm trọng nhất, nhưng có thể giải quyết với kiến thức đúng đắn.

**Key Takeaways:**
- Luôn monitor SQL queries
- Hiểu rõ lazy vs eager loading
- Sử dụng JOIN FETCH khi appropriate
- Profile và optimize queries
- Test performance với realistic data volumes

Good luck với phỏng vấn! 🚀
