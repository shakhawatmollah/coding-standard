# Java Enterprise Coding Standards & Best Practices Guide

**Tech Stack:** Java 17+ | Spring Boot 3.x | Spring Security | JPA/Hibernate | PostgreSQL | Maven/Gradle | Docker | CI/CD

**Version:** 1.0  
**Last Updated:** February 2026

---

## Table of Contents

1. [General Java Principles](#1-general-java-principles)
2. [Project Structure](#2-project-structure)
3. [Spring Boot Best Practices](#3-spring-boot-best-practices)
4. [Spring Security](#4-spring-security)
5. [JPA/Hibernate & Database](#5-jpahibernate--database)
6. [RESTful API Design](#6-restful-api-design)
7. [Exception Handling](#7-exception-handling)
8. [Testing Standards](#8-testing-standards)
9. [Security Best Practices](#9-security-best-practices)
10. [Performance Optimization](#10-performance-optimization)
11. [Build & Deployment](#11-build--deployment)
12. [Code Review Checklist](#12-code-review-checklist)

---

## 1. General Java Principles

### Rule Strength (Normative Language)
- **MUST / MUST NOT**: Mandatory requirement. Non-compliance blocks merge/release.
- **SHOULD / SHOULD NOT**: Strong recommendation. Deviations require documented rationale.
- **MAY**: Optional practice, used when context-dependent.

### 1.1 Naming Conventions

**MUST:**
```java
// Classes: PascalCase
public class UserAuthenticationService { }

// Interfaces: PascalCase (avoid 'I' prefix)
public interface PaymentProcessor { }

// Methods & variables: camelCase
public Optional<User> findUserByEmail(String emailAddress) { }

// Constants: UPPER_SNAKE_CASE
private static final int MAX_RETRY_ATTEMPTS = 3;
private static final String DEFAULT_ROLE = "ROLE_USER";

// Boolean methods: is/has/can prefix
public boolean isEmailVerified() { }
public boolean hasPermission() { }
public boolean canAccessResource() { }

// Collections: plural names
private List<Order> userOrders;
private Set<Permission> grantedPermissions;

// Packages: lowercase, hierarchical
package com.company.project.service.authentication;
```

**MUST NOT:**
```java
// Bad: Unclear abbreviations
public class UsrAuthSvc { }

// Bad: Hungarian notation
private String strEmail;
private int iCount;

// Bad: Single letter (except loops)
public User f(String e) { }

// Bad: Boolean without prefix
public boolean verified() { } // Should be isVerified()
```

### 1.2 Use Modern Java Features (Java 17+)

**MUST:**
```java
// Records for immutable DTOs
public record UserDTO(
    Long id,
    String email,
    String firstName,
    String lastName
) {
    // Compact constructor for validation
    public UserDTO {
        Objects.requireNonNull(email, "Email cannot be null");
        Objects.requireNonNull(firstName, "First name cannot be null");
    }
}

// Text blocks for readability
String query = """
    SELECT u.id, u.email, u.first_name
    FROM users u
    WHERE u.active = true
      AND u.created_at > ?
    ORDER BY u.created_at DESC
    """;

// Pattern matching for instanceof
if (payment instanceof CreditCardPayment cc) {
    return cc.getCardNumber().substring(0, 4);
} else if (payment instanceof PayPalPayment pp) {
    return pp.getEmail();
}

// Switch expressions
String status = switch (orderStatus) {
    case PENDING -> "Processing";
    case CONFIRMED -> "Confirmed";
    case SHIPPED -> "In Transit";
    case DELIVERED -> "Delivered";
    case CANCELLED -> "Cancelled";
};

// Sealed classes for restricted hierarchies
public sealed interface PaymentMethod 
    permits CreditCard, DebitCard, PayPal {
    Money processPayment(Money amount);
}

// Stream API for collections
List<String> activeEmails = users.stream()
    .filter(User::isActive)
    .map(User::getEmail)
    .sorted()
    .collect(Collectors.toList());
```

### 1.3 Null Safety

**MUST:**
```java
// Use Optional for return types
public Optional<User> findUserByEmail(String email) {
    return userRepository.findByEmail(email);
}

// Handle Optional properly
userService.findUserByEmail(email)
    .ifPresentOrElse(
        user -> log.info("User found: {}", user.getId()),
        () -> log.warn("User not found")
    );

// Transform with map
String userName = userService.findUserByEmail(email)
    .map(User::getFullName)
    .orElse("Unknown");

// Use Objects.requireNonNull for validation
public UserService(UserRepository repository) {
    this.repository = Objects.requireNonNull(repository, 
        "Repository cannot be null");
}

// Null-safe collection operations
List<String> emails = Optional.ofNullable(users)
    .orElse(Collections.emptyList())
    .stream()
    .map(User::getEmail)
    .toList();
```

**MUST NOT:**
```java
// Bad: Returning null
public User findUser(Long id) {
    return userRepository.findById(id); // May return null!
}

// Bad: No null check
public void process(User user) {
    String email = user.getEmail(); // NullPointerException!
}

// Bad: Unnecessary checks with Optional
Optional<User> user = findUser(id);
if (user != null && user.isPresent()) { // Optional never null!
    // process
}
```

---

## 2. Project Structure

### 2.1 Layered Architecture

```
src/main/java/com/company/project/
+-- config/                          # Configuration
|   +-- SecurityConfig.java
|   +-- DatabaseConfig.java
|   \\-- CacheConfig.java
|
+-- controller/                      # Presentation Layer
|   +-- UserController.java
|   \\-- advice/
|       \\-- GlobalExceptionHandler.java
|
+-- service/                         # Business Logic
|   +-- UserService.java
|   \\-- impl/
|       \\-- UserServiceImpl.java
|
+-- repository/                      # Data Access
|   \\-- UserRepository.java
|
+-- model/                           # Domain Model
|   +-- entity/
|   |   +-- User.java
|   |   \\-- BaseEntity.java
|   +-- dto/
|   |   +-- request/
|   |   |   \\-- CreateUserRequest.java
|   |   \\-- response/
|   |       \\-- UserResponse.java
|   \\-- enums/
|       \\-- UserRole.java
|
+-- security/                        # Security Components
|   +-- JwtTokenProvider.java
|   \\-- JwtAuthenticationFilter.java
|
+-- exception/                       # Custom Exceptions
|   \\-- ResourceNotFoundException.java
|
+-- mapper/                          # Entity-DTO Mappers
|   \\-- UserMapper.java
|
\\-- util/                           # Utilities
    \\-- ValidationUtil.java

src/main/resources/
+-- application.yml
+-- application-dev.yml
+-- application-prod.yml
\\-- db/migration/
    +-- V1__create_users_table.sql
    \\-- V2__create_roles_table.sql
```

### 2.2 Base Entity

**MUST:**
```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
@Setter
public abstract class BaseEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;
    
    @Version
    private Long version; // Optimistic locking
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof BaseEntity)) return false;
        BaseEntity that = (BaseEntity) o;
        return id != null && id.equals(that.id);
    }
    
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}

// Usage
@Entity
@Table(name = "users")
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class User extends BaseEntity {
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String passwordHash;
    
    private String firstName;
    private String lastName;
    
    @Enumerated(EnumType.STRING)
    private UserRole role;
    
    private Boolean active = true;
}
```

---

## 3. Spring Boot Best Practices

### 3.1 Dependency Injection

**MUST:** Use constructor injection
```java
@Service
@RequiredArgsConstructor // Lombok generates constructor
@Slf4j
public class UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;
    
    // Spring automatically injects via constructor
}

// Without Lombok
@Service
public class OrderService {
    
    private final OrderRepository repository;
    private final PaymentService paymentService;
    
    public OrderService(OrderRepository repository,
                       PaymentService paymentService) {
        this.repository = repository;
        this.paymentService = paymentService;
    }
}
```

**MUST NOT:**
```java
// Bad: Field injection
@Service
public class UserService {
    @Autowired
    private UserRepository repository; // Hard to test
}
```

### 3.2 Configuration Properties

**MUST:**
```java
@Configuration
@ConfigurationProperties(prefix = "app")
@Validated
@Data
public class AppProperties {
    
    @NotNull
    private Security security;
    
    @NotNull
    private Jwt jwt;
    
    @Data
    public static class Security {
        @Min(3)
        private int maxLoginAttempts = 5;
        
        private List<String> allowedOrigins;
    }
    
    @Data
    public static class Jwt {
        @NotBlank
        private String secret;
        
        @Min(1)
        private long expirationMinutes = 15;
    }
}
```

```yaml
# application.yml
app:
  security:
    max-login-attempts: 5
    allowed-origins:
      - https://example.com
  jwt:
    secret: ${JWT_SECRET}
    expiration-minutes: 15
```

### 3.3 Exception Handling

**MUST:**
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ApiResponse<?> handleNotFound(ResourceNotFoundException ex) {
        log.warn("Resource not found: {}", ex.getMessage());
        return ApiResponse.error(ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Map<String, String>> handleValidation(
            MethodArgumentNotValidException ex) {
        
        Map<String, String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                error -> error.getDefaultMessage() != null ? 
                    error.getDefaultMessage() : "Invalid"
            ));
        
        return ApiResponse.error("Validation failed", errors);
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiResponse<?> handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        return ApiResponse.error("An error occurred");
    }
}

// Response wrapper
@Data
@Builder
public class ApiResponse<T> {
    private String status;
    private String message;
    private T data;
    private LocalDateTime timestamp;
    
    public static <T> ApiResponse<T> success(T data) {
        return ApiResponse.<T>builder()
            .status("success")
            .data(data)
            .timestamp(LocalDateTime.now())
            .build();
    }
    
    public static <T> ApiResponse<T> error(String message) {
        return ApiResponse.<T>builder()
            .status("error")
            .message(message)
            .timestamp(LocalDateTime.now())
            .build();
    }
}
```

---

## 4. Spring Security

### 4.1 Security Configuration

**MUST:**
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final JwtAuthenticationFilter jwtAuthFilter;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) 
            throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfig()))
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(
                    "/api/v1/auth/**",
                    "/api/v1/public/**",
                    "/actuator/health",
                    "/v3/api-docs/**",
                    "/swagger-ui/**"
                ).permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, 
                UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
}
```

### 4.2 JWT Implementation

**MUST:**
```java
@Component
@Slf4j
public class JwtTokenProvider {
    
    @Value("${app.jwt.secret}")
    private String jwtSecret;
    
    @Value("${app.jwt.expiration-minutes}")
    private long expirationMinutes;
    
    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(jwtSecret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("type", "access");
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(
                System.currentTimeMillis() + 
                expirationMinutes * 60 * 1000
            ))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }
    
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    public <T> T extractClaim(String token, 
                             Function<Claims, T> resolver) {
        final Claims claims = extractAllClaims(token);
        return resolver.apply(claims);
    }
    
    private Claims extractAllClaims(String token) {
        try {
            return Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
        } catch (ExpiredJwtException e) {
            throw new InvalidTokenException("Token expired");
        } catch (MalformedJwtException | SignatureException e) {
            throw new InvalidTokenException("Invalid token");
        }
    }
    
    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && 
               !isTokenExpired(token);
    }
    
    private boolean isTokenExpired(String token) {
        return extractClaim(token, Claims::getExpiration)
            .before(new Date());
    }
}
```

---

## 5. JPA/Hibernate & Database

### 5.1 Entity Design

**MUST:**
```java
@Entity
@Table(
    name = "users",
    indexes = {
        @Index(name = "idx_email", columnList = "email"),
        @Index(name = "idx_created_at", columnList = "created_at")
    }
)
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@ToString(exclude = "passwordHash")
public class User extends BaseEntity {
    
    @Column(nullable = false, unique = true, length = 255)
    private String email;
    
    @Column(name = "password_hash", nullable = false)
    private String passwordHash;
    
    @Column(name = "first_name", nullable = false, length = 100)
    private String firstName;
    
    @Column(name = "last_name", nullable = false, length = 100)
    private String lastName;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserRole role = UserRole.USER;
    
    @Column(nullable = false)
    private Boolean active = true;
    
    @OneToMany(mappedBy = "user", 
               cascade = CascadeType.ALL, 
               orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    // Helper methods
    public void addOrder(Order order) {
        orders.add(order);
        order.setUser(this);
    }
}
```

**MUST NOT:**
```java
// Bad: No constraints or indexes
@Entity
@Data // Don't use @Data with JPA
public class User {
    @Id
    @GeneratedValue
    private Long id;
    
    private String email; // No validation
    private String password; // Plain text!
    
    @OneToMany(fetch = FetchType.EAGER) // Bad: Eager loading
    private List<Order> orders;
}
```

### 5.2 Repository

**MUST:**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    Optional<User> findByEmail(String email);
    
    boolean existsByEmail(String email);
    
    @Query("SELECT u FROM User u WHERE u.active = :active")
    List<User> findByActive(@Param("active") Boolean active);
    
    // Projection for efficiency
    @Query("""
        SELECT new com.company.dto.UserSummaryDTO(
            u.id, u.email, u.firstName, u.lastName
        )
        FROM User u WHERE u.active = true
        """)
    List<UserSummaryDTO> findAllActiveSummaries();
    
    // Fetch join to avoid N+1
    @Query("""
        SELECT DISTINCT u FROM User u 
        LEFT JOIN FETCH u.orders 
        WHERE u.id = :id
        """)
    Optional<User> findByIdWithOrders(@Param("id") Long id);
    
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.id = :id")
    void deactivate(@Param("id") Long id);
}
```

### 5.3 Avoid N+1 Queries

**MUST:**
```java
// Fetch join
@Query("""
    SELECT DISTINCT u FROM User u 
    LEFT JOIN FETCH u.orders 
    WHERE u.active = true
    """)
List<User> findAllActiveWithOrders();

// Entity graph
@EntityGraph(attributePaths = {"orders", "orders.items"})
List<User> findAllWithOrdersAndItems();

// Batch fetching
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @BatchSize(size = 10)
    private List<Order> orders;
}
```

**MUST NOT:**
```java
// Bad: N+1 query
List<User> users = userRepository.findAll(); // 1 query
for (User user : users) {
    List<Order> orders = user.getOrders(); // N queries!
}
```

### 5.4 Transactions

**MUST:**
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true) // Default
public class OrderService {
    
    private final OrderRepository orderRepository;
    
    @Transactional // Write transaction
    public OrderDTO createOrder(CreateOrderRequest request) {
        Order order = buildOrder(request);
        Order saved = orderRepository.save(order);
        return mapToDTO(saved);
    }
    
    public OrderDTO getOrder(Long id) {
        // Uses read-only transaction
        return orderRepository.findById(id)
            .map(this::mapToDTO)
            .orElseThrow(() -> new ResourceNotFoundException("Order not found"));
    }
}
```

---

## 6. RESTful API Design

### 6.1 Controller

**MUST:**
```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Validated
@Tag(name = "User Management")
public class UserController {
    
    private final UserService userService;
    
    @GetMapping
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Page<UserDTO>> getAll(
            @RequestParam(defaultValue = "0") @Min(0) int page,
            @RequestParam(defaultValue = "20") @Min(1) @Max(100) int size) {
        
        Pageable pageable = PageRequest.of(page, size);
        return ResponseEntity.ok(userService.getAll(pageable));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> getById(
            @PathVariable @Positive Long id) {
        
        UserDTO user = userService.getById(id);
        return ResponseEntity.ok(ApiResponse.success(user));
    }
    
    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<ApiResponse<UserDTO>> create(
            @Valid @RequestBody CreateUserRequest request) {
        
        UserDTO created = userService.create(request);
        
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        
        return ResponseEntity
            .created(location)
            .body(ApiResponse.success(created));
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> update(
            @PathVariable @Positive Long id,
            @Valid @RequestBody UpdateUserRequest request) {
        
        UserDTO updated = userService.update(id, request);
        return ResponseEntity.ok(ApiResponse.success(updated));
    }
    
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> delete(@PathVariable @Positive Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 6.2 Request/Response DTOs

**MUST:**
```java
// Request DTO with validation
@Data
@Builder
public class CreateUserRequest {
    
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;
    
    @NotBlank
    @Size(min = 8, max = 128)
    @Pattern(
        regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).*$",
        message = "Password must contain uppercase, lowercase, digit, and special character"
    )
    private String password;
    
    @NotBlank
    @Size(min = 2, max = 100)
    private String firstName;
    
    @NotBlank
    @Size(min = 2, max = 100)
    private String lastName;
}

// Response DTO (using record)
public record UserDTO(
    Long id,
    String email,
    String firstName,
    String lastName,
    String role,
    Boolean active,
    LocalDateTime createdAt
) {}
```

---

## 7. Exception Handling

### Custom Exceptions

**MUST:**
```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
    
    public ResourceNotFoundException(String resource, String field, Object value) {
        super(String.format("%s not found with %s: %s", 
            resource, field, value));
    }
}

public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}
```

---

## 8. Testing Standards

### 8.1 Unit Tests

**MUST:**
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    @DisplayName("Should create user successfully")
    void shouldCreateUser() {
        // Given
        CreateUserRequest request = CreateUserRequest.builder()
            .email("test@example.com")
            .password("Test@123")
            .firstName("John")
            .lastName("Doe")
            .build();
        
        User user = User.builder()
            .id(1L)
            .email("test@example.com")
            .build();
        
        when(userRepository.existsByEmail(anyString()))
            .thenReturn(false);
        when(passwordEncoder.encode(anyString()))
            .thenReturn("hashedPassword");
        when(userRepository.save(any(User.class)))
            .thenReturn(user);
        
        // When
        UserDTO result = userService.create(request);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getEmail()).isEqualTo("test@example.com");
        
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    @DisplayName("Should throw exception when email exists")
    void shouldThrowWhenEmailExists() {
        // Given
        CreateUserRequest request = CreateUserRequest.builder()
            .email("existing@example.com")
            .build();
        
        when(userRepository.existsByEmail(anyString()))
            .thenReturn(true);
        
        // When & Then
        assertThatThrownBy(() -> userService.create(request))
            .isInstanceOf(DuplicateResourceException.class);
        
        verify(userRepository, never()).save(any());
    }
}
```

### 8.2 Integration Tests

**MUST:**
```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@Testcontainers
@ActiveProfiles("test")
class UserControllerIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:15-alpine")
            .withDatabaseName("testdb");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldCreateUser() {
        // Given
        CreateUserRequest request = CreateUserRequest.builder()
            .email("test@example.com")
            .password("Test@123")
            .firstName("John")
            .lastName("Doe")
            .build();
        
        // When
        ResponseEntity<ApiResponse> response = restTemplate
            .postForEntity("/api/v1/users", request, ApiResponse.class);
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
    }
}
```

---

## 9. Security Best Practices

### 9.1 Password Security

**MUST:**
```java
@Configuration
public class SecurityConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // Strength 12
    }
}

@Service
@RequiredArgsConstructor
public class PasswordService {
    
    private final PasswordEncoder passwordEncoder;
    
    private static final Pattern PASSWORD_PATTERN = Pattern.compile(
        "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).{8,}$"
    );
    
    public String hashPassword(String password) {
        validatePassword(password);
        return passwordEncoder.encode(password);
    }
    
    private void validatePassword(String password) {
        if (!PASSWORD_PATTERN.matcher(password).matches()) {
            throw new InvalidPasswordException(
                "Password must be at least 8 characters with " +
                "uppercase, lowercase, digit, and special character"
            );
        }
    }
}
```

### 9.2 Input Sanitization

**MUST:**
```java
@Component
public class InputSanitizer {
    
    public String sanitizeHtml(String input) {
        if (input == null) return null;
        
        PolicyFactory policy = new HtmlPolicyBuilder()
            .allowElements("p", "br", "b", "i")
            .toFactory();
        
        return policy.sanitize(input);
    }
    
    public String sanitizeFilename(String filename) {
        if (filename == null) return null;
        return filename.replaceAll("[^a-zA-Z0-9._-]", "");
    }
}
```

### 9.3 Logging Security

**MUST:**
```java
@Service
@Slf4j
public class UserService {
    
    public void createUser(CreateUserRequest request) {
        log.info("Creating user: {}", maskEmail(request.getEmail()));
        // NEVER log passwords, tokens, sensitive data
    }
    
    private String maskEmail(String email) {
        if (email == null || !email.contains("@")) return "***";
        String[] parts = email.split("@");
        return parts[0].charAt(0) + "***@" + parts[1];
    }
}
```

**MUST NOT:**
```java
// Bad: Logging sensitive data
log.info("Password: {}", password); // NEVER!
log.debug("Token: {}", jwtToken); // NEVER!
```

---

## 10. Performance Optimization

### 10.1 Caching

**MUST:**
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager manager = new SimpleCacheManager();
        manager.setCaches(Arrays.asList(
            new ConcurrentMapCache("users"),
            caffeineCache("userDetails", 1000, 10)
        ));
        return manager;
    }
    
    private Cache caffeineCache(String name, int maxSize, int minutes) {
        return new CaffeineCache(name, Caffeine.newBuilder()
            .maximumSize(maxSize)
            .expireAfterWrite(minutes, TimeUnit.MINUTES)
            .build());
    }
}

@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public UserDTO getById(Long id) {
        // Cached
        return findAndMap(id);
    }
    
    @CachePut(value = "users", key = "#result.id")
    public UserDTO update(Long id, UpdateUserRequest request) {
        // Updates cache
        return updateAndMap(id, request);
    }
    
    @CacheEvict(value = "users", key = "#id")
    public void delete(Long id) {
        // Removes from cache
        userRepository.deleteById(id);
    }
}
```

### 10.2 Database Optimization

**MUST:**
```yaml
# application.yml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
```

---

## 11. Build & Deployment

### 11.1 Maven (pom.xml)

**MUST:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.2</version>
    </parent>
    
    <groupId>com.company</groupId>
    <artifactId>myapp</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <java.version>17</java.version>
        <lombok.version>1.18.30</lombok.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 11.2 Dockerfile

**MUST:**
```dockerfile
# Multi-stage build
FROM maven:3.9-eclipse-temurin-17-alpine AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests -B

FROM eclipse-temurin:17-jre-alpine
RUN addgroup -g 1001 spring && adduser -S spring -u 1001 -G spring
WORKDIR /app
COPY --from=build --chown=spring:spring /app/target/*.jar app.jar
USER spring:spring
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s \
    CMD wget --spider http://localhost:8080/actuator/health || exit 1
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### 11.3 docker-compose.yml

**MUST:**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/myapp
    depends_on:
      - postgres
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
```

---

## 12. Code Review Checklist

### Before Submitting PR

**Code Quality**
- [ ] Follows naming conventions
- [ ] No hardcoded values
- [ ] No commented code
- [ ] Uses logger (not System.out)
- [ ] Proper error handling
- [ ] No code duplication

**Security**
- [ ] No sensitive data in code
- [ ] Input validation implemented
- [ ] SQL injection prevented
- [ ] Passwords hashed
- [ ] Authentication checked

**Testing**
- [ ] Unit tests written
- [ ] Integration tests for critical paths
- [ ] 80%+ test coverage
- [ ] Edge cases tested

**Performance**
- [ ] No N+1 queries
- [ ] Proper caching
- [ ] Database indexes reviewed
- [ ] Transactions correct

**Documentation**
- [ ] JavaDoc for public APIs
- [ ] README updated
- [ ] API docs updated

---

## Best Practices Summary

### DO's âœ…

1. **Use Java 17+ features** (records, sealed classes, pattern matching)
2. **Constructor injection** over field injection
3. **Optional for nullable returns**
4. **Parameterized queries** (prevent SQL injection)
5. **BCrypt password hashing** (strength 12+)
6. **Validate all inputs**
7. **Write comprehensive tests** (80%+ coverage)
8. **Use caching** strategically
9. **Implement proper logging** (no sensitive data)
10. **Follow layered architecture**

### DON'TS âŒ

1. **Don't use field injection** (@Autowired on fields)
2. **Don't return null** (use Optional)
3. **Don't swallow exceptions**
4. **Don't hardcode secrets**
5. **Don't log passwords/tokens**
6. **Don't trust user input**
7. **Don't create N+1 queries**
8. **Don't use SELECT ***
9. **Don't skip input validation**
10. **Don't mix business logic in controllers**

---

## Commit Message Convention

**MUST:**
```
type(scope): subject

Types: feat, fix, docs, style, refactor, test, chore, perf

Examples:
feat(auth): implement JWT refresh token
fix(user): resolve NPE in email validation
docs(api): update user endpoint docs
refactor(service): extract validation logic
test(auth): add login integration tests
```

---

## Additional Resources

- **Effective Java (3rd Edition)** - Joshua Bloch
- **Clean Code** - Robert C. Martin
- **Spring Boot Documentation**: https://spring.io/projects/spring-boot
- **OWASP Top 10**: https://owasp.org/www-project-top-ten/
- **JUnit 5**: https://junit.org/junit5/docs/current/user-guide/

---

**Document Version**: 1.0  
**Last Updated**: February 2026  
**Maintained By**: Engineering Team

