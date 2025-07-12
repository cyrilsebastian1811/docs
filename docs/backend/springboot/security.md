# Security Layer

- Spring Security defines a framework for security
- Implemented using Servlet Filters in the background.
    - Servlet Filters are used to pre-process / post-process web requests.
    - Servlet Filters can route web requests based on security logic.
- Two methods of securing an app:
    - __Declarative Security__:
        - Define application's security contraints in a configuration class with `@Configuration` annotation.
        - Seperates application code from security code.
    - __Programmatic Security__:
        - Spring Security provides an API for custom application coding.
        - Provides greater customization for specific app requirements.


__Security Concepts__:

- Authentication: Check if ==userid== and ==password== with credentials stored in app / db.
- Autherization: Check if user has an ==authorized role==.


__Pre-requisite__: Add `spring-boot-starter-security` to your project.


==Spring Security automatically secures all endpoints. THe default username is `user` and password can be found in the application logs.== This can be ovverriden:

``` .properties
spring.security.user.name=user
spring.security.user.password=user
```

## Password Storage

User provided passwords are encoded and stored securly in the database. Spring Security’s `PasswordEncoder` interface is used to perform a one-way transformation of a password.

- The general format for a password is: `{id}encodedPassword`
    - `id` is an identifier that is used to look up which `PasswordEncoder` should be used.
    ``` .txt
    {bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
    {noop}password
    {pbkdf2}5d923b44a6d129f3ddf3e3c8d29412723dcbde72445e8ef6bf3b508fbf17fa4ed4d6b99ca763d8dc
    {scrypt}$e0801$8bWJaSu2IKSn9Z9kM+TPXfOc/9bdYSrN1oD9qfVThWEwdRTnO7re7Ei+fUZRJ68k9lTyuTeUp4of4g24hHnazw==$OAOec05+bXxvuu/1qZ6NUR+xQYvYv7BeL1QxwRpY5Pc=
    {sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0
    ```
- `NoOpPasswordEncoder` is the default `PasswordEncoder`.
- Spring Security introduces `DelegatingPasswordEncoder`, which maps encoding id to a specific `PasswordEncoder`. Can be used to define and configure custom `PasswordEncoder`.

    ``` .java
    String idForEncode = "bcrypt";
    Map encoders = new HashMap<>();
    encoders.put("bcrypt", new BCryptPasswordEncoder()); // one-way hashing
    encoders.put("noop", NoOpPasswordEncoder.getInstance());
    encoders.put("pbkdf2", Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_5());
    encoders.put("pbkdf2@SpringSecurity_v5_8", Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_8());
    encoders.put("scrypt", SCryptPasswordEncoder.defaultsForSpringSecurity_v4_1());
    encoders.put("scrypt@SpringSecurity_v5_8", SCryptPasswordEncoder.defaultsForSpringSecurity_v5_8());
    encoders.put("argon2", Argon2PasswordEncoder.defaultsForSpringSecurity_v5_2());
    encoders.put("argon2@SpringSecurity_v5_8", Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8());
    encoders.put("sha256", new StandardPasswordEncoder());

    // idForEncode passed into the constructor determines which PasswordEncoder is used for encoding passwords. i.e. bcrypt
    PasswordEncoder passwordEncoder =
        new DelegatingPasswordEncoder(idForEncode, encoders);
    ```

## Cross-Site Request Forgery(CSRF)

Cross-Site Request Forgery (__CSRF__) is an attack where a malicious website tricks a user’s browser into making an __unauthorized request__ to another site where the user is authenticated.

- __Enabled by default__ for __non-GET requests__.
- Spring Security __generates a unique CSRF token__ for each user session and expects it in __every state-changing request__ (`POST`, `PUT`, `DELETE`, etc.).
- Requires __CSRF token__ in the request header (`X-CSRF-TOKEN`) or request body.
- Used for browser web requests. i.e. traditional web apps with HTML forms to add/modify data.
- __Stateless applications (like REST APIs) usually disable CSRF__.

=== "Enable"

    Use this for Stateful web applications, Form-based authentication.

    ``` .java
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig {

        @Bean
        public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
            http
                .authorizeHttpRequests(auth -> auth
                    .anyRequest().authenticated()
                )
                .formLogin(withDefaults()) // CSRF enabled by default
                .csrf(withDefaults()); // Explicitly enabling CSRF (default)

            return http.build();
        }
    }
    ```

=== "Disable"

    Use this REST APIs (stateless, JWT, OAuth2)

    ``` .java
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig {

        @Bean
        public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
            http
                .authorizeHttpRequests(auth -> auth
                    .anyRequest().authenticated()
                )
                .csrf(csrf -> csrf.disable()) // Disables CSRF for APIs
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
            return http.build();
        }
    }
    ```

---

## Implementations

Users, passwords and roles can be defined in several ways. Depending on this security can be implemented in:

- In-memory
- JDBC
- LDAP
- Custom / Pluggable

Our Users for illustration:

| User ID | Password | Roles |
|---|---|---|
| john | test123 | EMPLOYEE |
| mary | test123 | EMPLOYEE, MANAGER |
| susan | test123 | EMPLOYEE, MANAGER, ADMIN |


=== "In-memory Security"

    Configuring Spring Security to use In-memory database and pre-loading 3 users.

    ``` .java
    @Configuration
    public class SecurityConfig {
        
        @Bean
        public InMemoryUserDetailsManager userDetailsManager() {
            // Creating 3 User with the credentials and roles
            UserDetails john = User.builder()
                    .username("john")
                    .password("{noop}test123")
                    .roles("EMPLOYEE")
                    .build();

            UserDetails mary = User.builder()
                    .username("mary")
                    .password("{noop}test123")
                    .roles("EMPLOYEE", "MANAGER")
                    .build();

            UserDetails susan = User.builder()
                    .username("susan")
                    .password("{noop}test123")
                    .roles("EMPLOYEE", "MANAGER", "ADMIN")
                    .build();

            // load these users into the in-memory database
            return new InMemoryUserDetailsManager(john, mary, susan);
        }
    }
    ```

=== "JDBC Security"

    - Spring security can read user account information from database.
    - Three ways to configure it:

    === "Predefined Schema"

        Use schema defined by Spring Security.

        ``` .sql title="Schema"
        -- Users Table
        CREATE TABLE `users` (
            `username` VARCHAR(50) NOT NULL,
            `password` VARCHAR(50) NOT NULL,
            `enabled` TINYINT(1) NOT NULL,
            PRIMARY KEY (`username`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

        -- Inserting Users
        INSERT INTO `users` VALUES
        ('john', '{noop}test123', 1),
        ('mary', '{noop}test123', 1),
        ('susan', '{noop}test123', 1);

        -- Authorities Table
        CREATE TABLE `authorities` (
            `username` VARCHAR(50) NOT NULL,
            `authority` VARCHAR(50) NOT NULL,
            UNIQUE KEY `authorities_idx_1` (`username`, `authority`),
            CONSTRAINT `authorities_ibfk_1` FOREIGN KEY (`username`) REFERENCES `users` (`username`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

        -- Inserting Roles
        INSERT INTO `authorities` VALUES
        ('john', 'ROLE_EMPLOYEE'),
        ('mary', 'ROLE_EMPLOYEE'),
        ('mary', 'ROLE_MANAGER'),
        ('susan', 'ROLE_EMPLOYEE'),
        ('susan', 'ROLE_MANAGER'),
        ('susan', 'ROLE_ADMIN');
        ```

        ``` .java title="JDBC Config"
        @Configuration
        public class SecurityConfig {
            @Bean
            public UserDetailsManager userDetailsManager(DataSource dataSource) {
                // DataSource holds JDBC connection details
                // This dataSource is autoconfigured by spring boot
                return new JdbcUserDetailsManager(dataSource);
            }
        }
        ```

        ??? warning

            Need for unique key constrain based `username`, `authority` on the `authorities` table:

            - Enforces Uniqueness: Ensures that a user cannot have the same authority assigned more than once. For example, a user "john" cannot have multiple duplicate entries of "ROLE_USER".

                ``` .sql
                INSERT INTO authorities (username, authority) VALUES ('john', 'ROLE_USER');
                -- Duplicate entry
                INSERT INTO authorities (username, authority) VALUES ('john', 'ROLE_USER');
                ```

            - Speeds Up Queries: Improves the performance of lookups, particularly for authentication-related queries.
            
                ``` .sql
                SELECT authority FROM authorities WHERE username = 'john';
                ```

    === "Custom Schema(SQL queries)"

        - Spring security allows for custom schemas.
        - We must define how to query for users and roles. ==We make use of SQL queries==
            - Query to find user by username.
            - Query to find authorities/roles by username.

        ``` .sql title="Schema"
        -- Users Table
        CREATE TABLE `members` (
            `user_id` VARCHAR(50) NOT NULL,
            `pwd` VARCHAR(68) NOT NULL,
            `active` TINYINT(1) NOT NULL,
            PRIMARY KEY (`user_id`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

        -- Authorities/Roles Table
        CREATE TABLE `roles` (
            `user_id` VARCHAR(50) NOT NULL,
            `role` VARCHAR(50) NOT NULL,
            UNIQUE KEY `roles_idx_1` (`user_id`, `role`),
            CONSTRAINT `roles_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `members` (`user_id`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
        ```

        ``` .java title="JDBC Config with SQL queries"
        @Configuration
        public class SecurityConfig {
            @Bean
            public UserDetailsManager userDetailsManager(DataSource dataSource) {
                JdbcUserDetailsManager jdbcUserDetailsManager = new JdbcUserDetailsManager(dataSource);

                // Defines query to retrieve a user by username. ? -> is a placeholder for values from login
                jdbcUserDetailsManager.setUsersByUsernameQuery("SELECT user_id, pwd, active FROM members WHERE user_id=?");

                // Define query to retrieve the authority/roles by username
                jdbcUserDetailsManager.setAuthoritiesByUsernameQuery("SELECT user_id, role FROM roles WHERE user_id=?");

                return jdbcUserDetailsManager;
            }
        }
        ```

    === "Custom Schema(Hibernate/JPA)"

        Here we mkae use of Hibernate/JPA to define how to load users.

        <figure markdown="span">
            <figcaption>User and Roles mapping</figcaption>
            ![User and Roles mapping](./img/user-roles-mapping.png){ width="50%" }
        </figure>

        ??? info "Schema"

            ``` .sql
            -- You cannot drop a table if another table references it. Thus
            -- Disables foreign key validation.
            SET foreign_key_checks = 0;
            DROP TABLE IF EXISTS `user`;
            DROP TABLE IF EXISTS `role`;

            -- Re-enables foreign key validation
            SET foreign_key_checks = 1;

            -- Table structure for table `user`
            CREATE TABLE `user` (
            `id` INT NOT NULL AUTO_INCREMENT,
            `username` VARCHAR(50) NOT NULL,
            `password` CHAR(80) NOT NULL,
            `enabled` TINYINT NOT NULL,  
            PRIMARY KEY (`id`)
            ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

            -- Table structure for table `role`
            CREATE TABLE `role` (
            `id` INT NOT NULL AUTO_INCREMENT,
            `name` VARCHAR(50) DEFAULT NULL,
            PRIMARY KEY (`id`)
            ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

            -- Table structure for table `users_roles`
            CREATE TABLE `users_roles` (
            `user_id` INT NOT NULL,
            `role_id` INT NOT NULL,
            -- Composite Primary Key
            PRIMARY KEY (`user_id`,`role_id`),
            -- Creates an index on role_id
            -- Speeds up queries that fetch all users with a specific role.
            KEY `FK_ROLE_idx` (`role_id`),
            -- Creates an index on user_id
            -- Speeds up queries that fetch all roles for a specific user.
            KEY `FK_USER_idx` (`user_id`),
            -- ON DELETE NO ACTION: Prevents deletion of a user if they have associated roles.
            -- ON UPDATE NO ACTION: Prevents updates to user.id if referenced.
            CONSTRAINT `FK_USER` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
            CONSTRAINT `FK_ROLE` FOREIGN KEY (`role_id`) REFERENCES `role` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
            ``` 

        ``` .java
        @Entity
        @Table(name = "user")
        public class AuthUser {
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            @Column(name = "id")
            private Long id;

            @Column(name = "username")
            private String userName;

            @Column(name = "password")
            private String password;

            @ManyToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
            @JoinTable(name = "users_roles",
                    joinColumns = @JoinColumn(name = "user_id"),
                    inverseJoinColumns = @JoinColumn(name = "role_id"))
            private Collection<Role> roles;
        }

        @Entity
        @Table(name = "role")
        public class Role {
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            @Column(name = "id")
            private Long id;

            @Column(name = "name")
            private String name;
        }

        @Service
        public class AuthUserService implements UserDetailsService {
            // Must implement AuthUserDao. Not provided by spring
            private AuthUserDao authUserDao;

            @Autowired
            public AuthUserService(AuthUserDao authUserDao) { this.authUserDao = authUserDao; }

            @Override
            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
                User user = authUserDao.findByUserName(username);   // includes roles
                if (user == null) {
                    throw new UsernameNotFoundException("Invalid username or password");
                }

                // Converting custom User to userdetails.User
                return new org.springframework.security.core.userdetails.User(
                    user.getUserName(),
                    user.getPassword(),
                    mapRolesToAuthorities(user.getRoles()));
            }

            private Collection<? extends GrantedAuthority> mapRolesToAuthorities(Collection<Role> roles) {
                return roles
                    .stream()
                    .map(role -> new SimpleGrantedAuthority(role.getName()))
                    .collect(Collectors.toList());
            }
        }

        @Configuration
        public class SecurityConfig {
            @Bean
            public PasswordEncoder passwordEncoder() {
                return new BCryptPasswordEncoder();
            }

            // @Autowired not specified to inject UserService
            // @Bean handles this automatically
            @Bean
            public DaoAuthenticationProvider authenticationProvider(AuthUserService authUserService) {
                DaoAuthenticationProvider auth  = new DaoAuthenticationProvider();
                // Set the custom user details service
                auth.setUserDetailsService(authUserService);
                // Set the password encoder - bcrypt
                auth.setPasswordEncoder(passwordEncoder());
                return auth;
            }
        }
        ```

- ==Internally Spring Security uses `ROLE_` prefix for role entries.==

---

??? infor "Bcrypt encoding"

    - format: `{bcrypt}encoedPassword`
    - Password column must be at least 68 chars wide. `{bcrypt}` - 8 chars, `encoedPassword` - 60 chars
    
    ``` .sql title="password column contraint"
    `password` VARCHAR(68) NOT NULL,
    ```

    ``` .java
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    ```

__Restricting access to endpoints__:

``` .java title="Role-Based Access Control(RBAC)"
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    // Authorization
    http.authorizeHttpRequests(configurer ->
            configurer
                    .requestMatchers(HttpMethod.GET, "/api/employees").hasRole("EMPLOYEE")
                    .requestMatchers(HttpMethod.GET, "/api/employees/__").hasRole("EMPLOYEE")
                    .requestMatchers(HttpMethod.POST, "/api/employees").hasRole("MANAGER")
                    .requestMatchers(HttpMethod.PUT, "/api/employees").hasRole("MANAGER")
                    .requestMatchers(HttpMethod.PUT, "/api/employees/**").hasRole("MANAGER")
                    .requestMatchers(HttpMethod.DELETE, "/api/employees/__").hasRole("ADMIN")
    );

    // Using Basic Auth
    http.httpBasic(Customizer.withDefaults());

    // Disable CSRF
    http.csrf(csrf -> csrf.disable());

    return http.build();
}
```


## References

- [Spring Security Docs](https://docs.spring.io/spring-security/reference/)
- [Filters](https://docs.spring.io/spring-framework/reference/web/webmvc/filters.html)