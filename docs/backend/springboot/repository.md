# Repository Layer(Persistence Layer)

__JPA(Jakarta Persistence API)__

Standard API for Object-to-Relational Mapping (ORM). It's only a specification and defines a set of interfaces. Requires an implementaion to be usable. Implementations:

- Hibernate(Spring Boot default implementaion)
- EclipseLink
- DataNucleus

__Hibernate__

A framework for persisting/saving Java objects in a database.

- handles all of the low-level SQL.
- Minimizes the amount of JDBC code you have to develop.
- Provides Object-to-Relational Mapping (ORM). i.e. defines mapping between Java class and database table.
- Hibernate uses JDBC for all database communications


## Configuration

Include following Dependencies to `pom.xml` or `build.gradle`:

- JDBC Driver: `mysql-connector-j`
- Spring Data(ORM): `spring-boot-starter-data-jpa`

Provide Connection Details:

``` .properties title="application.properties"
# DataSource connection details
spring.datasource.url=jdbc:mysql://localhost:3306/student_tracker
spring.datasource.username=springboot
spring.datasource.password=springboot

# Spring Boot can deduce the JDBC driver class for most databases from the URL.
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# Auto create tables in SQL database
spring.jpa.hibernate.ddl-auto=update
```

You can configure SQL database behavior in Spring Boot using the `spring.jpa.hibernate.ddl-auto` property.

| Option | Description |
|---|---|
| `none` | No action is taken on the database schema. |
| `create` | Drops existing tables (if any) and __creates new tables__ every time the application starts. __Data loss risk.__ |
| `create-drop` | Drops existing tables, __creates new ones__, and then __drops tables on shutdown__. __⚠ Data loss risk.__ |
| `validate` | __Verifies__ that the database schema matches the entity model. __Fails if mismatched.__ |
| `update` | __Modifies the database schema__ to match the entity model. __Does not remove columns or tables.__ |

__Best Practices__:

- __`validate`__ → Use in __production__ to ensure schema correctness.
- __`update`__ → Use in __development__ (but not for major schema changes).
- __`create` / `create-drop`__ → Only for __testing__ (as they remove data).


## Database Interactions

There are 2 ways to interact with the database:

- Using DAO
- Using Spring Data JPA / JpaRepository

??? note "Entity Manager(DAO) v/s JPA Repository"

    __Entity Manager__:

    - Entity Manager provides low-level control over database operations.
    - Complex queries that require advanced features such as native SQL queries or stored procedure calls.

    __JPA Repository__:

    - Provides commonly used CRUD operations out-of-the-box without much code.
    - Provides pagination, sorting.
    - ==Generates queries based on method names.==
    - Can also create custom queries

### DAO(Data Access Object)

- A design pattern to interface with the database.
- Makes use of JPA Entity Manager to perform database operations.
- Uses JPA Query Language(JPQL): ==Query language for retrieving objects. But, makes use of entity name and entity fields==
- This where we define operation specific methods like `save()`, `findById()`, `finaAll()`, `update()` etc...

!!! info "Entity Manager"

    Special JPA healper object. Entity Manager is the main component for saving/retrieving entities. Based on configs Spring Boot will automatically create beans like Datasource, EntityManager, etc.

    Data Source defines database connection details.


``` .java
// Student is the Entity class
public interface StudentDAO {
    void save(Student student);
    Student findById(Integer id);
    List<Student> findAll();
    List<Student> findByLastName(String lastName);
    void update(Student student);
    void delete(Integer id);
}

@Repository
public class StudentDAOImpl implements StudentDAO {
    private EntityManager entityManager;

    @Autowired
    public StudentDAOImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    @Override
    @Transactional
    public void save(Student student) {
        // If StudentId == 0 / null, then save/insert else update
        entityManager.merge(student);
    }

    // @Transactional - not required for read operations
    @Override
    public Student findById(Integer id) {
        return entityManager.find(Student.class, id);
    }

    @Override
    public List<Student> findAll() {
        // Note: All JPQL queries are based on entity name and not table name. similarly field names and not column names

        TypedQuery<Student> query = entityManager.createQuery("FROM Student ORDER BY lastName ASC", Student.class);
        return query.getResultList();
    }

    @Override
    public List<Student> findByLastName(String lastName) {
        TypedQuery<Student> query = entityManager.createQuery("FROM Student WHERE lastName=:lastNameData", Student.class);
        query.setParameter("lastNameData", lastName);
        return query.getResultList();
    }

    @Override
    @Transactional // required as this updates database
    public void update(Student student) {
        // Updates only one student entity
        Student theStudent = entityManager.find(Student.class, student.getId());

        if(theStudent != null) {
            entityManager.merge(student);
        }

        // To update multiple students
        int numRowsUpdated = entityManager.createQuery("UPDATE Student SET lastName='test'").executeUpdate();
    }

    @Override
    @Transactional
    public void delete(Integer studentId) {

        Student student = entityManager.find(Student.class, studentId);

        // Removes one entity
        entityManager.remove(student);

        // Deletes multiple entity
        int numRowsDeleted = entityManager.createQuery("DELETE FROM Student WHERE lastName=:lastNameDate").executeUpdate();
    }
}
```

- `@Repository`: Sub-component of `@Component` annotation
    - Registers DAO implementation
    - Provides translation of any JDBC related exceptions.
- `@Transactional`: Automatically begins and ends a transaction for the JPA code.
- For strict JPQL, the select clause is required
    ``` .java
    TypedQuery<Student> query = entityManager.createQuery("SELECT s FROM Student s WHERE s.lastName=:lastNameData", Student.class);
    ```


### JpaRepository (Spring Data JPA)

- JpaRepository exposes basic CRUD methods
- By extending `JpaRepository` you get `findAll()`, `findById()`, `save()`, `deleteById()` out-off-the-box. [full-list](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)
- [Custom Queries](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html#jpa.query-methods.at-query)

``` .java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
    // Defining custom queries

    // Returns count of employees in a department.
    long countByDepartment(String department);

    // Returns true if an employee with the given email exists.
    boolean existsByEmail(String email);

    // Spring auto-generates SELECT * FROM employees WHERE name = ?.
    List<Employee> findByName(String name);

    // Returns a list of employees in a given department.
    List<Employee> findByDepartment(String department);

    // Uses AND condition in SQL.
    Optional<Employee> findByNameAndDepartment(String name, String department);

    // Enables pagination & sorting with PageRequest.of(page, size, Sort.by("name")).
    Page<Employee> findByDepartment(String department, Pageable pageable);

    // Custom Query. Makes use of entity class name and field name
    @Query("SELECT e FROM Employee e WHERE e.department = :dept")
    List<Employee> findEmployeesByDepartment(@Param("dept") String department);
}
```