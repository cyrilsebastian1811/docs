# Controllers, Services, Exception Hadling

## Controller Layer (Presentational Layer)

Controller leverages HTTP methods and response codes to distinguish the type of operations performed and the result of corresponding operations.

??? info "HTTP Methods"

    HTTP methods define the type of action performed on a given resource. Below is a table outlining the common HTTP methods and their purpose.

    | Method | Description |
    |---|---|
    | __GET__ | Read a list of entities or a single entity. It is idempotent and does not change the state of the resource. |
    | __POST__ | Create new entity. It is not idempotent. |
    | __PUT__ | Update an existing entity. It is idempotent. |
    | __PATCH__ | Partially updates an existing resource. |
    | __DELETE__ | Delete and existing entity. It is idempotent. |
    | __HEAD__ | Similar to GET but only retrieves response headers without the body. |
    | __OPTIONS__ | Returns supported HTTP methods for a given resource. |
    | __TRACE__ | Performs a message loop-back test for debugging. |
    | __CONNECT__ | Establishes a tunnel to a server, typically used for secure SSL/TLS connections. |

??? info "HTTP Response Codes"

    HTTP response status codes indicate the result of a client's request to a server. They are grouped into five categories:

    | Status Code | Category         | Description |
    |------------|----------------|-------------|
    | __1xx__    | Informational   | The request was received, continuing process. |
    | __2xx__    | Success         | The request was successfully received, understood, and accepted. |
    | __3xx__    | Redirection     | Further action is needed to complete the request. |
    | __4xx__    | Client Error    | The request contains bad syntax or cannot be fulfilled. |
    | __5xx__    | Server Error    | The server failed to fulfill a valid request. |

    __1xx - Informational__:

    | Code | Meaning |
    |------|---------|
    | __100__ | Continue: The server has received the request headers and the client should proceed to send the request body. |
    | __101__ | Switching Protocols: The server is switching to the protocol requested by the client. |
    | __102__ | Processing: The server has received the request but has not yet completed processing. |

    __2xx - Success__:

    | Code | Meaning |
    |------|---------|
    | __200__ | OK: The request was successful. |
    | __201__ | Created: The request has been fulfilled and resulted in a new resource being created. |
    | __202__ | Accepted: The request has been accepted for processing, but the processing has not been completed. |
    | __204__ | No Content: The server successfully processed the request, but there is no content to return. |

    __3xx - Redirection__:
    
    | Code | Meaning |
    |------|---------|
    | __301__ | Moved Permanently: The resource has been permanently moved to a new URL. |
    | __302__ | Found: The resource is temporarily located at a different URL. |
    | __304__ | Not Modified: The resource has not been modified since the last request. |

    __4xx - Client Errors__:

    | Code | Meaning |
    |------|---------|
    | __400__ | Bad Request: The server cannot process the request due to client error. |
    | __401__ | Unauthorized: Authentication is required and has failed or has not been provided. |
    | __403__ | Forbidden: The request is valid, but the server is refusing to fulfill it. |
    | __404__ | Not Found: The requested resource could not be found. |

    __5xx - Server Errors__:

    | Code | Meaning |
    |------|---------|
    | __500__ | Internal Server Error: A generic error message when the server encounters an unexpected condition. |
    | __502__ | Bad Gateway: The server received an invalid response from the upstream server. |
    | __503__ | Service Unavailable: The server is temporarily unavailable due to overload or maintenance. |
    | __504__ | Gateway Timeout: The server did not receive a timely response from the upstream server. |

    ---

    For a complete list of HTTP response codes, refer to the [MDN HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

??? info "MIME Content Types"

    MIME (Multipurpose Internet Mail Extensions) types define the nature and format of a document or file sent over the internet.

    __Text__:

    | MIME Type        | Description |
    |-----------------|-------------|
    | `text/plain`    | Plain text without formatting |
    | `text/html`     | HTML document |
    | `text/css`      | Cascading Style Sheets (CSS) |
    | `text/javascript` | JavaScript file |

    __Image__:

    | MIME Type         | Description |
    |------------------|-------------|
    | `image/jpeg`    | JPEG image |
    | `image/png`     | PNG image |
    | `image/gif`     | GIF image |
    | `image/svg+xml` | SVG image |
    | `image/webp`    | WebP image |

    __Audio__:

    | MIME Type        | Description |
    |-----------------|-------------|
    | `audio/mpeg`    | MP3 audio file |
    | `audio/ogg`     | Ogg Vorbis audio |
    | `audio/wav`     | WAV audio file |

    __Video__:

    | MIME Type       | Description |
    |---------------|-------------|
    | `video/mp4`   | MP4 video file |
    | `video/ogg`   | Ogg video file |
    | `video/webm`  | WebM video file |

    __Application (Binary Data)__:

    | MIME Type              | Description |
    |-----------------------|-------------|
    | `application/json`    | JSON data |
    | `application/xml`     | XML data |
    | `application/pdf`     | PDF document |
    | `application/zip`     | ZIP archive |
    | `application/x-www-form-urlencoded` | Form data encoded as key-value pairs |

    __Multipart (Used in Forms and Emails)__:

    | MIME Type                   | Description |
    |----------------------------|-------------|
    | `multipart/form-data`      | Used for file uploads in forms |
    | `multipart/alternative`    | Contains multiple versions of the same content (e.g., plain text and HTML emails) |

    ---

    For a complete list of MIME types, refer to [IANA MIME Types](https://www.iana.org/assignments/media-types/media-types.xhtml).

dependency: `spring-boot-starter-web`

``` .java
@RestController
@RequestMapping("/api")
public class EmployeeRestController {
    private final EmployeeServiceImpl employeeService;

    @Autowired
    public EmployeeRestController(EmployeeServiceImpl employeeService) {
        this.employeeService = employeeService;
    }

    // Notice, the response body is a DTO(Data transfer Object)
    @PostMapping("/employees")
    // @Valid ensures that the UserDTO fields are validated.
    public Employee saveEmployee(@Valid @RequestBody EmployeeDTO employeeDTO) {
        Employee newEmployee = employeeService.saveEmployee(employeeDTO);
        return newEmployee;
    }

    @GetMapping("/employees/{employeeId}")
    public Employee findEmployeeById(@PathVariable("employeeId") int employeeId) {
        Employee employee = employeeService.findEmployeeById(employeeId);
        if(employee == null) {
            throw new RuntimeException(String.format("Employee not found with id: %d", employeeId));
        }
        return employee;
    }
}
```

- We use `@RestController` annotation, which is a sub-type of `@Component`
- __DTO__: DTOs are typically used between the Controller and Service layers. [details](./model.md#dto)
- __Jackson Data Binding__: Is the process of converting JSON data to a Java POJO and vice versa. AKA Serialization / Deserialization, Marshalling / Unmarshalling. Jackson calls appropriate getter/setter method for data binding.
- `@RequestBody`: converts JSON to Java POJO
- Mapping Responses can be passed in 2 ways:
    - Plain Java POJO: Jackson converts it to JSON
    - `ResponseEntity<T>`: offers more customizability of response thrown. customize response status, message, etc.


### Spring Data Rest

- Leverages your existing JpaRepository and gives you REST CRUD implementation.
- Minimizes bioler plate code

__Setup__:

- Add `spring-boot-starter-data-rest`, `spring-boot-starter-data-jpa` dependencies
- Define Entity class
- Add JpaRepository corresponding to the entity class

``` .java
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String department;
    private double salary;

    // Getters & Setters
}

@RepositoryRestResource(path = "members") // Customize REST endpoint to /members
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
}
```

__Additional Configurations__

``` .properties
spring.data.rest.base-path=/magic-api
spring.data.rest.default-page-size=20
spring.data.rest.max-page-size=30
```

__Standard CRUD APIs__:

| __Method__ | __Endpoint__ | __Description__ | __Example Request__ |
|---|---|---|---|
| __GET__ | `/employees` | Get all employees (paginated & sorted) | `GET /employees?page=0&size=5&sort=name,asc` |
| __GET__ | `/employees/{id}` | Get a single employee by ID | `GET /employees/1` |
| __POST__ | `/employees` | Create a new employee | `POST /employees` (with JSON body) |
| __PUT__ | `/employees/{id}` | Update an existing employee | `PUT /employees/1` (with JSON body) |
| __PATCH__ | `/employees/{id}` | Partially update an employee | `PATCH /employees/1` (with JSON body) |
| __DELETE__ | `/employees/{id}` | Delete an employee | `DELETE /employees/1` |

__Pagination & Sorting APIs__:

| __Feature__ | __Parameter__ | __Example Request__ | __Description__ |
|---|---|---|---|
| __Get page `N`__ | `page=N` | `/employees?page=1` | Fetches the `N`th page (0-based index) |
| __Set page size__ | `size=X` | `/employees?page=0&size=5` | Limits the number of results per page |
| __Sort ascending__ | `sort=field,asc` | `/employees?sort=name,asc` | Sorts results by field in ascending order |
| __Sort descending__ | `sort=field,desc` | `/employees?sort=salary,desc` | Sorts results by field in descending order |
| __Sort by multiple fields__ | `sort=field1,asc&sort=field2,desc` | `/employees?sort=name,asc&sort=salary,desc` | Sorts by multiple fields |

## Service Layer (Business Logic Layer)

- Service Facade design pattern.
- Intermediate layer for custom business logic.
- Integrate data from multiple sources (DAO/repositories).
- We use `@Service` annotation, which is a sub-type of `@Component`
- ==`@Transactional` annotation is added to service instead of DAO, as service defines transactional boundaries.==


``` .java
public interface EmployeeService {
    Employee saveEmployee(Employee employee);
    Employee findEmployeeById(int employeeId);
    List<Employee> findAllEmployees();
    void deleteEmployee(int employeeId);
}

@Service
public class EmployeeServiceImpl implements EmployeeService {
    private final EmployeeDAO employeeDAO;

    @Autowired
    public EmployeeServiceImpl(EmployeeDAO employeeDAO) {
        this.employeeDAO = employeeDAO;
    }

    @Override
    @Transactional
    public Employee saveEmployee(Employee employee) {
        return employeeDAO.save(employee);
    }

    @Override
    public List<Employee> findAllEmployees() {
        return employeeDAO.findAll();
    }
}
```


## Exception Handling Layer

``` .java title="Exception Handling specific to a Controller"
// Creating the Error Response
public class StudentErrorResponse {
    private int status;
    private String message;
    private long timeStamp;
}


// Creating the Exception type
public class StudentNotFoundException extends RuntimeException {
    public StudentNotFoundException(String message) {
        super(message);
    }

    public StudentNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }

    public StudentNotFoundException(Throwable cause) {
        super(cause);
    }
}


@RestController
@RequestMapping("/api")
public class StudentRestController {

    @ExceptionHandler(StudentNotFoundException.class)
    public ResponseEntity<StudentErrorResponse> handleStudentNotFoundException(StudentNotFoundException exc) {
        StudentErrorResponse error = new StudentErrorResponse(HttpStatus.NOT_FOUND.value(), exc.getMessage(), System.currentTimeMillis());

        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> handleIllegalArgument(IllegalArgumentException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body("Invalid input: " + ex.getMessage());
    }

    @GetMapping("/students/{Id}")
    public Student getStudentById(@PathVariable("Id") int studentId) {
        if(studentId >= students.size() || studentId<0) {
            throw new StudentNotFoundException(String.format("No student with id: %d", studentId));
        }
        return students.get(studentId);
    }
}
```


``` .java title="Global Exception Handling"
@RestControllerAdvice
public class StudentRestExceptionHandler {
    @ExceptionHandler
    public ResponseEntity<StudentErrorResponse> handleStudentNotFoundException(StudentNotFoundException exc) {
        StudentErrorResponse error = new StudentErrorResponse(HttpStatus.NOT_FOUND.value(), exc.getMessage(), System.currentTimeMillis());

        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGenericException(Exception exc) {
        String errorMessage = "Error: " + exc.getMessage();
        return new ResponseEntity<>(errorMessage, HttpStatus.BAD_REQUEST);
    }
}
```

- RestControllerAdvice is similar to an interceptor / filter. It can be used to:
    - Pre-process request to controllers
    - Post-process response to handle exceptions
    - Perfect for global exception handling


## Filter

- Framework Level: Servlet specification (Java EE / Jakarta EE)
- Executes before `DispatcherServlet`
- Scope: Entire web application
- Request Mapping Knowledge: No knowledge of controller or handler
- At the filter stage: There is no knowledge of which controller will handle the request.
- Return Value Control:	Indirect — via `FilterChain.doFilter()`
- Use case: CORS, Security, Encoding

=== "Register via Annotation"

    ```java
    @WebFilter("/*")
    public class LoggingFilter implements Filter {
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
                throws IOException, ServletException {
            System.out.println("[Filter] Request received: " + ((HttpServletRequest) request).getRequestURI());
            chain.doFilter(request, response); // pass to next filter or DispatcherServlet
            System.out.println("[Filter] Response sent");
        }
    }
    ```

=== "Register via Bean"

    ```java
    public class LoggingFilter implements Filter {...}

    @Bean
    public FilterRegistrationBean<LoggingFilter> loggingFilter() {
        FilterRegistrationBean<LoggingFilter> reg = new FilterRegistrationBean<>();
        reg.setFilter(new LoggingFilter());
        reg.addUrlPatterns("/*");
        return reg;
    }
    ```


## Interceptor

- Framework Level: Spring MVC
- Executes: Before and after controller methods
- Scope: Only Spring MVC controller requests.
- Request Mapping Knowledge: Can access handler method details.
- Return Value Control:	Direct — can return false to block flow
- Use case: CORS, Security, Encoding

```java title="Defining Interceptor"
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String auth = request.getHeader("Authorization");
        if (auth == null || !auth.startsWith("Bearer ")) {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            return false; // Stop processing
        }
        return true; // Continue to controller
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println("[Interceptor] Request completed");
    }
}
```
```java title="Registering Interceptor"
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor()).addPathPatterns("/api/**");
    }
}
```