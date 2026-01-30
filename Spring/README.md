# Spring Framework Interview Preparation

## Core Spring Concepts

### 1. Inversion of Control (IoC) & Dependency Injection (DI)
- **IoC Container**: BeanFactory vs ApplicationContext
- **Dependency Injection Types**:
  - Constructor injection (recommended)
  - Setter injection
  - Field injection
- **Bean Scopes**: Singleton, Prototype, Request, Session, Application

### 2. Spring Core Annotations
- `@Component`: Generic stereotype for any Spring-managed component
- `@Service`: Business logic layer
- `@Repository`: Data access layer
- `@Controller`: Web layer (Spring MVC)
- `@RestController`: RESTful web services
- `@Autowired`: Dependency injection
- `@Qualifier`: Specify which bean to inject
- `@Value`: Inject property values
- `@Configuration`: Java-based configuration
- `@Bean`: Bean definition in configuration class

### 3. Spring Boot
- **Auto-configuration**: Automatic configuration based on classpath
- **Starter Dependencies**: spring-boot-starter-web, spring-boot-starter-data-jpa
- **Application Properties**: application.properties or application.yml
- **Embedded Servers**: Tomcat, Jetty, Undertow
- **Spring Boot Actuator**: Health checks, metrics, monitoring

### 4. Spring MVC
- **DispatcherServlet**: Front controller pattern
- **Controllers**: Handle HTTP requests
- **Request Mapping**: @GetMapping, @PostMapping, @PutMapping, @DeleteMapping
- **Model and View**: ModelAndView, Model
- **Exception Handling**: @ExceptionHandler, @ControllerAdvice
- **Interceptors**: Pre/post processing of requests

### 5. Spring Data JPA
- **Repository Pattern**: CrudRepository, JpaRepository
- **Query Methods**: Derived queries from method names
- **@Query**: Custom JPQL or native SQL queries
- **Pagination and Sorting**: Pageable, Sort
- **Specifications**: Dynamic queries
- **Auditing**: @CreatedDate, @LastModifiedDate

### 6. Spring Security
- **Authentication**: Username/password, OAuth2, JWT
- **Authorization**: Role-based access control
- **Security Configurations**: SecurityConfig
- **Method Security**: @PreAuthorize, @Secured
- **CSRF Protection**
- **Password Encoding**: BCryptPasswordEncoder

### 7. Spring AOP (Aspect-Oriented Programming)
- **Aspect**: Cross-cutting concern
- **Join Point**: Point in program execution
- **Advice**: Action taken at join point (Before, After, Around)
- **Pointcut**: Expression to match join points
- **Use Cases**: Logging, transaction management, security

### 8. Spring Transaction Management
- **@Transactional**: Declarative transaction management
- **Propagation**: REQUIRED, REQUIRES_NEW, NESTED, etc.
- **Isolation Levels**: READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE
- **Rollback Rules**: Exception-based rollback

## Common Interview Questions

### Basic Questions
1. What is Spring Framework and what are its benefits?
2. Explain Dependency Injection
3. What is the difference between @Component, @Service, and @Repository?
4. What is Spring Boot and how is it different from Spring?
5. Explain the Spring Bean lifecycle

### Intermediate Questions
1. How does Spring Boot auto-configuration work?
2. What is the difference between @RestController and @Controller?
3. Explain Spring AOP and its use cases
4. How do you handle exceptions in Spring Boot?
5. What is the difference between CrudRepository and JpaRepository?

### Advanced Questions
1. How does Spring manage transactions?
2. Explain Spring Security authentication and authorization flow
3. How do you implement caching in Spring Boot?
4. What are the different bean scopes and when to use them?
5. How do you optimize Spring Boot application performance?

## Code Examples

### Example 1: REST Controller
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User saved = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, 
                                          @Valid @RequestBody User user) {
        return userService.update(id, user)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteById(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Example 2: Service Layer with Transaction
```java
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public List<User> findAll() {
        return userRepository.findAll();
    }
    
    public Optional<User> findById(Long id) {
        return userRepository.findById(id);
    }
    
    public User save(User user) {
        return userRepository.save(user);
    }
    
    public Optional<User> update(Long id, User userDetails) {
        return userRepository.findById(id)
            .map(user -> {
                user.setName(userDetails.getName());
                user.setEmail(userDetails.getEmail());
                return userRepository.save(user);
            });
    }
    
    public void deleteById(Long id) {
        userRepository.deleteById(id);
    }
}
```

### Example 3: Spring Security Configuration
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF for stateless JWT authentication
            // For session-based auth, keep CSRF enabled
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic();
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Example 4: Exception Handling
```java
@Slf4j  // Lombok annotation, or use: private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
            ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        // Log the exception for debugging
        log.error("Unexpected error occurred", ex);
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An error occurred: " + ex.getMessage(),
            LocalDateTime.now()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

## Best Practices
1. Use constructor injection over field injection
2. Keep controllers thin, business logic in services
3. Use DTOs for data transfer
4. Implement proper exception handling
5. Use Spring profiles for different environments
6. Implement proper logging
7. Write integration tests
8. Use Spring Boot Actuator for monitoring
9. Follow RESTful conventions
10. Secure your endpoints properly

## Resources
- Spring Framework Documentation
- Spring Boot Documentation
- Baeldung Spring Tutorials
- Spring in Action book
