# Implementing JWT Authentication in Spring Boot

*Published on February 20, 2023*

## Introduction

Security is a critical aspect of any application, and implementing a robust authentication mechanism is essential. JSON Web Tokens (JWT) have become a popular choice for implementing stateless authentication in modern web applications. This guide will walk you through the process of implementing JWT authentication in a Spring Boot application.

## Table of Contents

1. [Understanding JWT](#understanding-jwt)
2. [Project Setup](#project-setup)
3. [JWT Implementation](#jwt-implementation)
4. [Security Configuration](#security-configuration)
5. [Authentication Controller](#authentication-controller)
6. [Securing Endpoints](#securing-endpoints)
7. [Testing the Implementation](#testing-the-implementation)
8. [Best Practices](#best-practices)
9. [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)

## Understanding JWT

### What is JWT?

JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed.

### JWT Structure

A JWT consists of three parts separated by dots (`.`):

1. **Header**: Contains the type of token and the signing algorithm being used
2. **Payload**: Contains the claims (statements about an entity and additional data)
3. **Signature**: Used to verify that the sender of the JWT is who it says it is and to ensure the message wasn't changed along the way

Example JWT:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### Benefits of JWT

- **Stateless**: The server doesn't need to store session information
- **Portable**: Can be used across different domains
- **Secure**: Digitally signed
- **Extensible**: Can include custom claims

## Project Setup

### Dependencies

Add the following dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- H2 Database (for demo purposes) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- JJWT (Java JWT) -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Lombok (optional, for reducing boilerplate code) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

### Application Properties

Configure your application properties in `src/main/resources/application.properties`:

```properties
# Server port
server.port=8080

# H2 Database Configuration
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# JWT Configuration
jwt.secret=your-secret-key-here-should-be-very-long-and-secure
jwt.expiration=86400000
```

## JWT Implementation

### User Entity and Repository

Create a User entity:

```java
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String username;
    
    @Column(nullable = false)
    private String password;
    
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "role")
    private Set<String> roles = new HashSet<>();
}
```

Create a User repository:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Boolean existsByUsername(String username);
}
```

### JWT Utility Class

Create a JWT utility class to generate and validate tokens:

```java
@Component
public class JwtUtils {
    
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    @Value("${jwt.expiration}")
    private int jwtExpirationMs;
    
    public String generateJwtToken(Authentication authentication) {
        UserDetailsImpl userPrincipal = (UserDetailsImpl) authentication.getPrincipal();
        
        return Jwts.builder()
                .setSubject(userPrincipal.getUsername())
                .setIssuedAt(new Date())
                .setExpiration(new Date((new Date()).getTime() + jwtExpirationMs))
                .signWith(getSigningKey(), SignatureAlgorithm.HS512)
                .compact();
    }
    
    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(jwtSecret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
    
    public String getUsernameFromJwtToken(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
    }
    
    public boolean validateJwtToken(String authToken) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(authToken);
            return true;
        } catch (SecurityException | MalformedJwtException e) {
            logger.error("Invalid JWT signature: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            logger.error("JWT token is expired: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            logger.error("JWT token is unsupported: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            logger.error("JWT claims string is empty: {}", e.getMessage());
        }
        
        return false;
    }
}
```

### User Details Implementation

Create a custom UserDetails implementation:

```java
public class UserDetailsImpl implements UserDetails {
    private static final long serialVersionUID = 1L;
    
    private Long id;
    private String username;
    private String password;
    private Collection<? extends GrantedAuthority> authorities;
    
    public UserDetailsImpl(Long id, String username, String password,
                          Collection<? extends GrantedAuthority> authorities) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.authorities = authorities;
    }
    
    public static UserDetailsImpl build(User user) {
        List<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority(role))
                .collect(Collectors.toList());
        
        return new UserDetailsImpl(
                user.getId(),
                user.getUsername(),
                user.getPassword(),
                authorities);
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }
    
    public Long getId() {
        return id;
    }
    
    @Override
    public String getPassword() {
        return password;
    }
    
    @Override
    public String getUsername() {
        return username;
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    
    @Override
    public boolean isEnabled() {
        return true;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        UserDetailsImpl user = (UserDetailsImpl) o;
        return Objects.equals(id, user.id);
    }
}
```

### User Details Service

Create a custom UserDetailsService:

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
    
    @Autowired
    UserRepository userRepository;
    
    @Override
    @Transactional
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User Not Found with username: " + username));
        
        return UserDetailsImpl.build(user);
    }
}
```

## Security Configuration

### JWT Authentication Filter

Create a JWT authentication filter:

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtUtils jwtUtils;
    
    @Autowired
    private UserDetailsServiceImpl userDetailsService;
    
    private static final Logger logger = LoggerFactory.getLogger(JwtAuthenticationFilter.class);
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        try {
            String jwt = parseJwt(request);
            if (jwt != null && jwtUtils.validateJwtToken(jwt)) {
                String username = jwtUtils.getUsernameFromJwtToken(jwt);
                
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            logger.error("Cannot set user authentication: {}", e);
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String parseJwt(HttpServletRequest request) {
        String headerAuth = request.getHeader("Authorization");
        
        if (StringUtils.hasText(headerAuth) && headerAuth.startsWith("Bearer ")) {
            return headerAuth.substring(7);
        }
        
        return null;
    }
}
```

### Authentication Entry Point

Create a custom authentication entry point:

```java
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {
    
    private static final Logger logger = LoggerFactory.getLogger(JwtAuthenticationEntryPoint.class);
    
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                        AuthenticationException authException) throws IOException, ServletException {
        logger.error("Unauthorized error: {}", authException.getMessage());
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Error: Unauthorized");
    }
}
```

### Web Security Config

Configure Spring Security:

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    UserDetailsServiceImpl userDetailsService;
    
    @Autowired
    private JwtAuthenticationEntryPoint unauthorizedHandler;
    
    @Bean
    public JwtAuthenticationFilter authenticationJwtTokenFilter() {
        return new JwtAuthenticationFilter();
    }
    
    @Override
    public void configure(AuthenticationManagerBuilder authenticationManagerBuilder) throws Exception {
        authenticationManagerBuilder.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
    }
    
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and().csrf().disable()
            .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
            .authorizeRequests()
            .antMatchers("/api/auth/**").permitAll()
            .antMatchers("/api/test/**").permitAll()
            .antMatchers("/h2-console/**").permitAll()
            .anyRequest().authenticated();
        
        // For H2 Console
        http.headers().frameOptions().disable();
        
        http.addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);
    }
}
```

## Authentication Controller

Create DTOs for authentication requests and responses:

```java
@Data
public class LoginRequest {
    @NotBlank
    private String username;
    
    @NotBlank
    private String password;
}

@Data
public class SignupRequest {
    @NotBlank
    @Size(min = 3, max = 20)
    private String username;
    
    @NotBlank
    @Size(min = 6, max = 40)
    private String password;
    
    private Set<String> roles;
}

@Data
@AllArgsConstructor
public class JwtResponse {
    private String token;
    private String type = "Bearer";
    private Long id;
    private String username;
    private List<String> roles;
    
    public JwtResponse(String token, Long id, String username, List<String> roles) {
        this.token = token;
        this.id = id;
        this.username = username;
        this.roles = roles;
    }
}

@Data
@AllArgsConstructor
public class MessageResponse {
    private String message;
}
```

Create the authentication controller:

```java
@CrossOrigin(origins = "*", maxAge = 3600)
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    @Autowired
    AuthenticationManager authenticationManager;
    
    @Autowired
    UserRepository userRepository;
    
    @Autowired
    PasswordEncoder encoder;
    
    @Autowired
    JwtUtils jwtUtils;
    
    @PostMapping("/signin")
    public ResponseEntity<?> authenticateUser(@Valid @RequestBody LoginRequest loginRequest) {
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword()));
        
        SecurityContextHolder.getContext().setAuthentication(authentication);
        String jwt = jwtUtils.generateJwtToken(authentication);
        
        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
        List<String> roles = userDetails.getAuthorities().stream()
                .map(item -> item.getAuthority())
                .collect(Collectors.toList());
        
        return ResponseEntity.ok(new JwtResponse(
                jwt,
                userDetails.getId(),
                userDetails.getUsername(),
                roles));
    }
    
    @PostMapping("/signup")
    public ResponseEntity<?> registerUser(@Valid @RequestBody SignupRequest signUpRequest) {
        if (userRepository.existsByUsername(signUpRequest.getUsername())) {
            return ResponseEntity
                    .badRequest()
                    .body(new MessageResponse("Error: Username is already taken!"));
        }
        
        // Create new user's account
        User user = new User();
        user.setUsername(signUpRequest.getUsername());
        user.setPassword(encoder.encode(signUpRequest.getPassword()));
        
        Set<String> strRoles = signUpRequest.getRoles();
        Set<String> roles = new HashSet<>();
        
        if (strRoles == null) {
            roles.add("ROLE_USER");
        } else {
            strRoles.forEach(role -> {
                switch (role) {
                    case "admin":
                        roles.add("ROLE_ADMIN");
                        break;
                    case "mod":
                        roles.add("ROLE_MODERATOR");
                        break;
                    default:
                        roles.add("ROLE_USER");
                }
            });
        }
        
        user.setRoles(roles);
        userRepository.save(user);
        
        return ResponseEntity.ok(new MessageResponse("User registered successfully!"));
    }
}
```

## Securing Endpoints

Create a test controller with secured endpoints:

```java
@CrossOrigin(origins = "*", maxAge = 3600)
@RestController
@RequestMapping("/api/test")
public class TestController {
    
    @GetMapping("/all")
    public String allAccess() {
        return "Public Content.";
    }
    
    @GetMapping("/user")
    @PreAuthorize("hasRole('ROLE_USER') or hasRole('ROLE_MODERATOR') or hasRole('ROLE_ADMIN')")
    public String userAccess() {
        return "User Content.";
    }
    
    @GetMapping("/mod")
    @PreAuthorize("hasRole('ROLE_MODERATOR')")
    public String moderatorAccess() {
        return "Moderator Board.";
    }
    
    @GetMapping("/admin")
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    public String adminAccess() {
        return "Admin Board.";
    }
}
```

## Testing the Implementation

### Database Initialization

Create a data initializer to add some test users:

```java
@Component
public class DataInitializer implements CommandLineRunner {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    public void run(String... args) throws Exception {
        // Create admin user
        User admin = new User();
        admin.setUsername("admin");
        admin.setPassword(passwordEncoder.encode("admin123"));
        Set<String> adminRoles = new HashSet<>();
        adminRoles.add("ROLE_ADMIN");
        adminRoles.add("ROLE_USER");
        admin.setRoles(adminRoles);
        
        // Create regular user
        User user = new User();
        user.setUsername("user");
        user.setPassword(passwordEncoder.encode("user123"));
        Set<String> userRoles = new HashSet<>();
        userRoles.add("ROLE_USER");
        user.setRoles(userRoles);
        
        // Save users to database if they don't exist
        if (!userRepository.existsByUsername("admin")) {
            userRepository.save(admin);
        }
        
        if (!userRepository.existsByUsername("user")) {
            userRepository.save(user);
        }
    }
}
```

### Testing with Postman or cURL

1. **Register a new user**:

```bash
curl -X POST http://localhost:8080/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","password":"password123","roles":["user"]}'
```

2. **Login to get JWT token**:

```bash
curl -X POST http://localhost:8080/api/auth/signin \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","password":"password123"}'
```

Response:
```json
{
  "token": "eyJhbGciOiJIUzUxMiJ9...",
  "type": "Bearer",
  "id": 3,
  "username": "testuser",
  "roles": ["ROLE_USER"]
}
```

3. **Access protected endpoint**:

```bash
curl -X GET http://localhost:8080/api/test/user \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..."
```

Response:
```
User Content.
```

4. **Access admin endpoint (should fail)**:

```bash
curl -X GET http://localhost:8080/api/test/admin \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..."
```

Response:
```
{
  "timestamp": "2023-02-20T10:15:30.123+00:00",
  "status": 403,
  "error": "Forbidden",
  "message": "Forbidden",
  "path": "/api/test/admin"
}
```

## Best Practices

### 1. Secure Your Secret Key

- Use a strong, randomly generated key
- Store it in environment variables or a secure vault
- Rotate keys periodically

### 2. Set Appropriate Token Expiration

- Short-lived tokens (15-60 minutes) are more secure
- Implement refresh tokens for better user experience

### 3. Include Only Necessary Claims

- Minimize the data stored in the token
- Avoid storing sensitive information

### 4. Use HTTPS

- Always use HTTPS in production
- Secure cookies with the `Secure` and `HttpOnly` flags

### 5. Implement Token Blacklisting

For critical applications, implement a token blacklist to invalidate tokens before they expire:

```java
@Service
public class TokenBlacklistService {
    
    private Set<String> blacklistedTokens = new HashSet<>();
    
    public void blacklist(String token) {
        blacklistedTokens.add(token);
    }
    
    public boolean isBlacklisted(String token) {
        return blacklistedTokens.contains(token);
    }
}
```

Update the JWT filter to check the blacklist:

```java
@Autowired
private TokenBlacklistService tokenBlacklistService;

@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {
    try {
        String jwt = parseJwt(request);
        if (jwt != null && !tokenBlacklistService.isBlacklisted(jwt) && jwtUtils.validateJwtToken(jwt)) {
            // Proceed with authentication
        }
    } catch (Exception e) {
        logger.error("Cannot set user authentication: {}", e);
    }
    
    filterChain.doFilter(request, response);
}
```

### 6. Implement Refresh Tokens

For better user experience, implement refresh tokens:

```java
@PostMapping("/refresh")
public ResponseEntity<?> refreshToken(@Valid @RequestBody TokenRefreshRequest request) {
    String requestRefreshToken = request.getRefreshToken();
    
    return refreshTokenService.findByToken(requestRefreshToken)
            .map(refreshTokenService::verifyExpiration)
            .map(RefreshToken::getUser)
            .map(user -> {
                String token = jwtUtils.generateTokenFromUsername(user.getUsername());
                return ResponseEntity.ok(new TokenRefreshResponse(token, requestRefreshToken));
            })
            .orElseThrow(() -> new TokenRefreshException(requestRefreshToken,
                    "Refresh token is not in database!"));
}
```

## Common Issues and Troubleshooting

### 1. Token Expiration Issues

If tokens expire too quickly, adjust the expiration time in your properties file:

```properties
jwt.expiration=3600000  # 1 hour in milliseconds
```

### 2. CORS Issues

If you're experiencing CORS issues, configure CORS properly:

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:3000")  // Your frontend URL
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true);
    }
}
```

### 3. "Invalid Signature" Error

This usually means the secret key used to verify the token doesn't match the one used to create it. Ensure you're using the same secret key.

### 4. "JWT Token is Expired" Error

This is normal when a token expires. Implement refresh tokens or redirect the user to the login page.

## Conclusion

Implementing JWT authentication in Spring Boot provides a secure, stateless authentication mechanism for your applications. By following the steps and best practices outlined in this guide, you can create a robust authentication system that protects your application's resources while providing a good user experience.

Remember that security is an ongoing process. Stay updated with the latest security practices and regularly review and update your authentication implementation.

## References

- [Spring Security Documentation](https://docs.spring.io/spring-security/reference/index.html)
- [JWT.io](https://jwt.io/)
- [JJWT GitHub Repository](https://github.com/jwtk/jjwt)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Spring Boot Security JWT Tutorial](https://www.bezkoder.com/spring-boot-jwt-authentication/)
