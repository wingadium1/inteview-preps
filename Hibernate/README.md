# Hibernate Interview Preparation

## Core Hibernate Concepts

### 1. ORM (Object-Relational Mapping)
- **Definition**: Mapping Java objects to database tables
- **Benefits**: Database independence, reduced SQL code, object-oriented approach
- **Hibernate Architecture**: SessionFactory, Session, Transaction, Query

### 2. Hibernate Configuration
- **hibernate.cfg.xml**: XML-based configuration
- **Persistence.xml**: JPA configuration
- **Properties**: Database connection, dialect, show_sql, hbm2ddl.auto
- **Annotations**: JPA annotations vs Hibernate-specific

### 3. Entity Mapping
- **@Entity**: Mark class as entity
- **@Table**: Specify table name
- **@Id**: Primary key
- **@GeneratedValue**: Auto-generate primary key
  - GenerationType.AUTO
  - GenerationType.IDENTITY
  - GenerationType.SEQUENCE
  - GenerationType.TABLE
- **@Column**: Column mapping
- **@Transient**: Exclude field from persistence

### 4. Relationships
- **@OneToOne**: One-to-one relationship
- **@OneToMany**: One-to-many relationship
- **@ManyToOne**: Many-to-one relationship
- **@ManyToMany**: Many-to-many relationship
- **Cascade Types**: ALL, PERSIST, MERGE, REMOVE, REFRESH, DETACH
- **Fetch Types**: LAZY vs EAGER
- **Bidirectional vs Unidirectional**

### 5. Hibernate Session
- **Session Lifecycle**: Transient, Persistent, Detached, Removed
- **Session Methods**:
  - save() / persist()
  - update() / merge()
  - delete() / remove()
  - get() / load()
  - saveOrUpdate()

### 6. HQL (Hibernate Query Language)
- **Object-oriented query language**
- **Named queries**: @NamedQuery
- **Positional parameters**: ?1, ?2
- **Named parameters**: :paramName
- **Joins**: inner join, left join, right join
- **Aggregations**: count, sum, avg, max, min

### 7. Criteria API
- **Type-safe queries**
- **CriteriaBuilder**: Build criteria queries
- **CriteriaQuery**: Define query structure
- **Root**: Query root entity
- **Predicates**: Where conditions

### 8. Caching
- **First-level Cache**: Session-level cache (enabled by default)
- **Second-level Cache**: SessionFactory-level cache
  - EHCache
  - Infinispan
  - Hazelcast
- **Query Cache**: Cache query results
- **@Cacheable**: Mark entity as cacheable
- **Cache Strategies**: READ_ONLY, READ_WRITE, NONSTRICT_READ_WRITE, TRANSACTIONAL

### 9. Performance Optimization
- **Lazy Loading**: Load data on-demand
- **Batch Processing**: Process large datasets efficiently
- **N+1 Problem**: Avoid multiple queries
- **JOIN FETCH**: Eager fetch associations
- **Pagination**: First result, max results
- **Native SQL**: Use when HQL is not sufficient

### 10. Transaction Management
- **ACID Properties**: Atomicity, Consistency, Isolation, Durability
- **Isolation Levels**: READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE
- **Optimistic Locking**: @Version
- **Pessimistic Locking**: LockMode

## Common Interview Questions

### Basic Questions
1. What is Hibernate and what are its advantages?
2. Explain the difference between save() and persist()
3. What is the difference between get() and load()?
4. Explain the Hibernate architecture
5. What are the different states of a Hibernate object?

### Intermediate Questions
1. What is the N+1 problem and how do you solve it?
2. Explain lazy loading vs eager loading
3. What is the difference between first-level and second-level cache?
4. How do you map relationships in Hibernate?
5. What is HQL and how is it different from SQL?

### Advanced Questions
1. How does Hibernate implement caching?
2. Explain optimistic vs pessimistic locking
3. How do you handle performance issues in Hibernate?
4. What are the different cascade types?
5. How do you implement batch processing in Hibernate?

## Code Examples

### Example 1: Entity Mapping
```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username", nullable = false, unique = true)
    private String username;
    
    @Column(name = "email", nullable = false)
    private String email;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, 
               fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
    
    // Getters and setters
}
```

### Example 2: One-to-Many Relationship
```java
@Entity
@Table(name = "orders")
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "order_number")
    private String orderNumber;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, 
               orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
    
    // Helper methods
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }
    
    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);
    }
}
```

### Example 3: HQL Query
```java
// Simple HQL query
String hql = "FROM User u WHERE u.username = :username";
Query<User> query = session.createQuery(hql, User.class);
query.setParameter("username", "john_doe");
User user = query.uniqueResult();

// HQL with join
String hql = "SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id";
Query<User> query = session.createQuery(hql, User.class);
query.setParameter("id", 1L);
User user = query.uniqueResult();

// Named query
@NamedQuery(
    name = "User.findByEmail",
    query = "FROM User u WHERE u.email = :email"
)
```

### Example 4: Criteria API
```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> root = cq.from(User.class);

// Add predicates
Predicate usernamePredicate = cb.equal(root.get("username"), "john_doe");
Predicate emailPredicate = cb.like(root.get("email"), "%@example.com");
cq.where(cb.and(usernamePredicate, emailPredicate));

List<User> users = session.createQuery(cq).getResultList();
```

### Example 5: Batch Processing
```java
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

int batchSize = 50;
for (int i = 0; i < 10000; i++) {
    User user = new User();
    user.setUsername("user" + i);
    user.setEmail("user" + i + "@example.com");
    session.save(user);
    
    if (i % batchSize == 0) {
        session.flush();
        session.clear();
    }
}

tx.commit();
session.close();
```

### Example 6: Second-Level Cache Configuration
```java
// Enable second-level cache in properties
hibernate.cache.use_second_level_cache=true
hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory

// Mark entity as cacheable
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    // Entity fields
}
```

## Best Practices
1. Always use transactions
2. Close sessions properly (try-with-resources)
3. Use lazy loading by default
4. Avoid N+1 queries (use JOIN FETCH)
5. Use batch processing for bulk operations
6. Enable query caching for frequently executed queries
7. Use criteria API for dynamic queries
8. Version entities for optimistic locking
9. Use connection pooling (HikariCP)
10. Monitor and tune Hibernate performance

## Common Pitfalls
1. Not closing sessions
2. Using eager loading everywhere
3. Ignoring N+1 problem
4. Not using transactions
5. Improper cascade configuration
6. Not handling lazy initialization exceptions
7. Not using batch processing for bulk operations

## Resources
- Hibernate Documentation
- Java Persistence with Hibernate
- High-Performance Java Persistence
- Baeldung Hibernate Tutorials
