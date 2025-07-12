# Models

Models can be divided into 2 categories:

- API models(DTO)
- Database Model(Entity)

## DTO

- Decouples API models from database models (Prevents exposing JPA entities).
- Optimizes performance by reducing unnecessary data transfer.
- Enhances security by restricting sensitive fields.
- Helps with validation before persisting data.
- Must have a public or protected no-argument construtor.

```java
@NoArgsConstructor
@Getter @Setter
public class UserDTO {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 6, message = "Password must be at least 6 characters long")
    private String password;

    @Pattern(regexp = "^\\d{10}$", message = "Phone number must be 10 digits")
    private String phoneNumber;

    @JsonCreator
    public UserDTO(@JsonProperty("name") String name, @JsonProperty("email") String email, @JsonProperty("pwd") String password, @JsonProperty("phone") String phoneNumber) {
        this.name = name;
        this.email = email;
        this.password = password;
        this.phoneNumber = phoneNumber;
    }
}
```

- Controller Layer – Receives DTOs as API request and sends DTOs as API response.
- Service Layer – Converts DTO to Entity, and vice-versa.

__Validation Annotations__:

| Annotation | Purpose |
|---|---|
| `@NotBlank` | Ensures the field is not null and not empty (for Strings) |
| `@Size(min, max`) | Limits the number of characters in a field |
| `@Email` | Validates proper email format |
| `@Pattern(regexp)` | Ensures a field follows a custom regex pattern |
| `@Min` / `@Max` | Restricts numerical values within a range |

---

## Entity

This represents the Database model and forms an Object-to-Relational Mapping(ORM). Entity classes:

- Must be annotated with `@Entity`.
- Must have a public or protected no-argument construtor.
- The class may or may not have other construtors.

```java
@Entity
@NoArgsConstructor
@Getter @Setter
// Specifies the table name
@Table(name="receipts",
    uniqueConstraints = @UniqueConstraint(columnNames = "id"), // unique contrain on database side
    indexes = {
        @Index(name="idx_purchase_date_time", columnList = "purchaseDate, purchaseTime")
    }
)
// Ensure retailer_name.length > 0 and is either alphanumeric, space, -, &
// Pattern enforcement on database side.
@Check(constraints = "CHAR_LENGTH(retailer_name) > 0 AND retailer_name NOT REGEXP '[^a-zA-Z0-9 \\-&]'")
public class Receipt {
    // Marks this as the primary key attribute
    @Id
    @UuidGenerator
    // updatable, nullable are contrains imposed on database side
    @Column(name = "id", updatable = false, nullable = false, length = 36)
    private UUID id;

    @NotNull(message = "Retailer is required")
    // Pattern validation on application side
    @Pattern(regexp = "^[\\w\\s\\-&]+$", message = "Retailer name must contain only alphanumeric characters, spaces, hyphens, or ampersands")
    @Column(name="retailerName", length = 255)
    private String retailer;

    // Null check on application side
    @NotNull(message = "Purchase date is required")
    // Specifies the column name. Defaults to lowercase and snake_case attribute name.
    // e.g. purchase_date
    @Column(name="purchaseDate")
    private LocalDate purchaseDate;

    @NotNull(message = "Purchase time is required")
    @Column(name="purchaseTime")
    private LocalTime purchaseTime;

    @NotNull(message = "Total is required")
    @Column(name="total", precision = 10, scale = 2)
    private BigDecimal total;

    @Enumerated(EnumType.STRING)
    @Column(columnDefinition = "ENUM('PENDING', 'APPROVED', 'REJECTED') DEFAULT 'PENDING'")
    private Status status;
}
```

__Annotations__:

- `@Table`: If table name not specified, defaults to lowercase and snake_case class name.
- `@Column`: If column name not specified, defaults to lowercase and snake_case attribute name.
- `@Patern`, `@NotNull`: are validations performed on Java side.
- `@Column(updatable, nullable, length)` options are contrains enforced on database side
- `@Table(uniqueConstraints)`, `@Check` enforced on database side.
- `@GeneratedValue`: To auto generate IDs. ID Generation strategies:

    | Strategy | Description |
    |---|---|
    | `GenerationType.AUTO` | Hibernate chooses the best strategy based on the database. |
    | `GenerationType.IDENTITY` | Uses auto-increment feature of the database. Each insert gets a new ID. |
    | `GenerationType.SEQUENCE` | Uses a database sequence to generate IDs. |
    | `GenerationType.TABLE` | Uses a table to store ID sequences (mimics SEQUENCEs in databases that don’t support them). |
    | `GenerationType.UUID` _(Hibernate 6+)_ | Uses a random UUID (v4) as ID. globally unique |

!!! warning

    - JPA makes use of class and attribute names and not table and column names. Thus, make use od class and attribute names for database operations.

??? info "`@GeneratedValue` v/s `@UuidGenerator`"

    There are 2 options to generate UUID as the primary key:

    - `@GeneratedValue(strategy = GenerationType.UUID)`:
        - Uses Hibernate’s built-in UUID generator.
        - Works with databases that support UUID types (e.g., PostgreSQL, MySQL 8+).
        - Does not allow custom UUID strategies (e.g., UUID v1, v7, etc.).
    - `@UuidGenerator`:
        - More flexible than `GenerationType.UUID`.
        - Supports different UUID types (RANDOM, TIME, AUTO). e.g. `@UuidGenerator(style = UuidGenerator.Style.TIME)`

---

### Mappings

__Concepts__:

- __Primary Key__: Identifies a unique row in a table
- __Foreign Key__:
    - Links table together.
    - a field in one table that refers the primary key in another table.
    - Preserves relationsship between tables.
- __Cascade__: Applies same operation to associated entities.
    - ==CASCADE DELETE: Not suitable for all mappings especially many-to-many. e.g. Students and Courses==

    ??? note "Cascade Types"

        Apply same operation to related entities. We can specify the cascade type:

        | Type | Description |
        |---|---|
        | PERSIST | If entity is persisted / saved, related entity will also be persisted. |
        | REMOVE | If entity is removed / deleted, related entity will also be deleted. |
        | REFRESH | If entity is refreshed, related entity will also be refreshed. |
        | DETACH | If entity is detached(not associated w/ session), then related entity will also be detached. |
        | MERGE | If entity is merged, then related entity will also be merged. |
        | ALL | All of the above cascade types |

        To configure mutliple Cascade types:

        ```java
        @OneToOne(cascade={
            CascadeType.DETACH,
            CascadeType.MERGE,
            CascadeType.PERSIST,
            CascadeType.REFRESH,
            CascadeType.REMOVE})
        ```
- __Eager v/s Lazy Loading__: Defined by `FetchType`
    - `FetchType.EAGER`: loads everything. Will cause performance issues.
    - `FetchType.LAZY`: retrieves on request. ==Best practice.==
        - Loads main entity first and then loads dependent entities based on demand.
        - Need an connection to database to retrieve data.

    ??? note "Default Fetch Types"

        Each mapping has it's default fetch type

        | Mapping | Default Fetch Type |
        |---|---|
        | `@OneToOne` | `FetchType.EAGER` |
        | `@OneToMany` | `FetchType.LAZY` |
        | `@ManyToOne` | `FetchType.EAGER` |
        | `@ManyToMany` | `FetchType.LAZY` |

- __Uni-Directional v/s Bi-Directional Mapping__:
    - Uni-Directional: we can only access mapped entity from the parent.
    - Bi-Directional: we can access mapped entity from parent and vice-versa.

??? note "Entity Lifecycle"

    It's a set of status an Hibernate entity can go through:

    | Operations | Description |
    |---|---|
    | Detach | If entity is detached, it is not associated with a Hibernate session. |
    | Merge | If instance is detached from session, then merge will reattach to session. |
    | Persist | Transitions new instances to managed state. Next flush/commit will save in db. |
    | Remove | Transitions managed entity to be removed. Next flush/commit will delete from db. |
    | Refresh | Reload / synch object with data from db. Prevents stale data. |

    <figure markdown="span">
        <figcaption>Entity Lifecycle</figcaption>
        ![Entity Lifecycle](./img/entity-lifecycles.png){ width="50%" }
    </figure>


__Types__:

=== "One-To-One"

    ```sql title="Schema"
    -- Table structure for table `instructor_detail`
    CREATE TABLE `instructor_detail` (
        `id` INT NOT NULL AUTO_INCREMENT,
        `youtube_channel` VARCHAR(125) DEFAULT NULL,
        `hobby` VARCHAR(48) DEFAULT NULL,
        PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    -- Table structure for table `instructor`
    CREATE TABLE `instructor` (
        `id` INT NOT NULL AUTO_INCREMENT,
        `first_name` VARCHAR(45) DEFAULT NULL,
        `last_name` VARCHAR(48) DEFAULT NULL,
        `email` VARCHAR(48) DEFAULT NULL,
        `instructor_detail_id` INT DEFAULT NULL,

        PRIMARY KEY (`id`),
        -- Creates an index to query instructors by specific instructor_detail_id. Useful for Bi-directional mapping
        KEY `FK_DETAIL_idx` (`instructor_detail_id`),
        CONSTRAINT `FK_DETAIL` FOREIGN KEY (`instructor_detail_id`) REFERENCES `instructor_detail` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
    ```

    ```java title="Uni-Directional Mapping"
    @Entity
    @Table(name="instructor_detail")
    public class InstructorDetail {
        @Id
        @GeneratedValue(strategy=GenerationType.IDENTITY)
        @Column(name="id")
        private int id;

        @Column(name="youtube_channel")
        private String youtubeChannel;

        @Column(name="hobby")
        private String hobby;
    }

    @Entity
    @Table(name="instructor")
    public class Instructor {
        @Id
        @GeneratedValue(strategy=GenerationType.IDENTITY)
        @Column(name="id")
        private int id;

        @Column(name="first_name")
        private String firstName;

        @Column(name="last_name")
        private String lastName;

        @Column(name="email")
        private String email;

        // Specify CascadeType
        @OneToOne(cascade=CascadeType.ALL)
        // The Foregin key column name
        @JoinColumn(name="instructor_detail_id")
        private InstructorDetail instructorDetail;
    }

    @Repository
    public class InstructorDao {
        ...
        public void save(Instructor instructor) {
            // Will save InstructorDetail as well. Due to CascadeType.ALL
            entityManager.persist(Instructor);
        }
    }

    public class TestService {
        ...
        @Transactional
        public void createInstructor() {
            Instructor instructor = new Instructor("A", "B", "a@b.com");

            InstructorDetail instructorDetail = new InstructorDetail("youa@youtube.com", "testing");

            instructor.setInstructorDetail(instructorDetail);

            instructorDao.save(instructor);
        }
    }
    ```

    ```java title="Bi-Directional Mapping" hl_lines="6-10"
    @Entity
    @Table(name="instructor_detail")
    public class InstructorDetail {
        ...

        // mappedBy refers to instructorDetail property of Instructor.class
        // Uses Instructor.class(@JoinColumn.name) to find associated instructorDetail for given instructor
        // The index `FK_DETAIL_idx` (`instructor_detail_id`) helps.
        // allows querying by instructor_detail_id on instructor table
        @OneToOne(mappedBy="instructorDetail", cascade=CascadeType.ALL)
        private Instructor instructor;
    }
    ```

    ==To delete InstructorDetail without deleting Instructor. We have to do the following:==

    - We have to remove `CascadeType.REMOVE`
    - We also have to remove the Bi-directional link from InstructorDetail to Instructor

    ```java hl_lines="6-10 21-23"
    @Entity
    @Table(name="instructor_detail")
    public class InstructorDetail {
        ...

        @OneToOne(mappedBy="instructorDetail", cascade={
            CascadeType.DETACH,
            CascadeType.MERGE,
            CascadeType.PERSIST,
            CascadeType.REFRESH})
        private Instructor instructor;
    }

    @Repository
    public class InstructorDetailDao {
        ...
        @Transactional
        public void delete(int id) {
            InstructorDetail instructorDetail = entityManager.find(InstructorDetail.class, id);

            // remove the associated object reference. Break Bi-directional link
            // If we remove instructorDetail, the associated instructor should no longer reference this instructorDetail
            instructorDetail.getInstructor().setInstructorDetail(null);

            entityManager.remove(instructorDetail);
        }
    }
    ```

=== "One-To-Many"

    ==Don't configure One-to-many or Many-to-one relationships with `CascadeType.REMOVE`==

    === "Bi-directional"

        ```sql title="Schema"
        -- Table structure for table `instructor`
        CREATE TABLE `instructor` (
            `id` INT NOT NULL AUTO_INCREMENT,
            `first_name` VARCHAR(45) DEFAULT NULL,
            `last_name` VARCHAR(48) DEFAULT NULL,
            `email` VARCHAR(48) DEFAULT NULL,
            PRIMARY KEY (`id`)
        ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

        -- Table structure for table `course`
        CREATE TABLE `course` (
            `id` INT NOT NULL AUTO_INCREMENT,
            `title` VARCHAR(125) DEFAULT NULL,
            `instructor_id` INT DEFAULT NULL,

            PRIMARY KEY (`id`),
            UNIQUE KEY `TITLE_UNIQUE` (`title`),
            -- Creates an index to query courses by specific instructor_id. Useful for Bi-directional mapping
            KEY `FK_INSTRUCTOR_idx` (`instructor_id`),
            CONSTRAINT `FK_INSTRUCTOR` FOREIGN KEY (`instructor_id`) REFERENCES `instructor` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
        ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
        ```

        ```java title="Mapping" hl_lines="120-122"
        @Entity
        @Table(name="course")
        public class Course {
            @Id
            @GeneratedValue(strategy=GenerationType.IDENTITY)
            @Column(name="id")
            private int id;

            @Column(name="title")
            private String title;

            @ManyToOne(cascade={CascadeType.DETACH, CascadeType.MERGE,
                    CascadeType.PERSIST, CascadeType.REFRESH})
            @JoinColumn(name="instructor_id")
            private Instructor instructor;
        }

        @Entity
        @Table(name="instructor")
        public class Instructor {
            @Id
            @GeneratedValue(strategy=GenerationType.IDENTITY)
            @Column(name="id")
            private int id;
            ...

            // mappedBy refers to instructor property in the Course class
            // Uses the information from the Course class @JoinColumn to find associated courses for instructor
            // This is where KEY `FK_INSTRUCTOR_idx` (`instructor_id`) helps. By, querying on instructor_id on course table
            @OneToMany(fetch=FetchType.LAZY,
                mappedBy="instructor",
                cascade={CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH})
            private List<Course> courses;

            // Convenience menthods for bi-directional relationships
            public void addCourse(Course tempCourse) {
                if(this.courses == null) {
                    this.courses = new ArrayList<>();
                }
                courses.add(tempCourse);
                tempCourse.setInstructor(this);
            }
        }

        @Repository
        public class CourseDao {
            ...
            public Course findCourseById(int Id) {
                return entityManager.find(Course.class, id);
            }

            public List<Course> findCoursesByInstructorId(int Id) {
                TypedQuery<Course> query = entityManager.createQuery("FROM Course WHERE instructor.id=:instructorId", Course.class);
                query.setParameter("instructorId", Id);
                return query.getResultList();
            }

            public void update(Course course) { entityManager.merge(course); } 

            public void deleteById(int id) {
                Course course = entityManager.find(Course.class, id);
                entityManager.remove(course);
            }
        }

        @Repository
        public class InstructorDao {
            public void save(Instructor theInstructor) {
                entityManager.persist(theInstructor);
            }

            public Instructor findInstructorById(int Id) {
                return entityManager.find(Instructor.class, id);
            }

            public Instructor findInstructorByIdJoinFetch(int Id) {
                TypedQuery<Instructor> query = entityManager.createQuery("SELECT i FROM Instructor i JOIN FETCH i.courses WHERE i.id=:instructorId", Instructor.class);

                // To perform joins on instructor, course, instructor_detail in one query
                // (1)!
                // This performs 2 seperate joins
                // 1. join on instructor and courses
                // 2. join on instructor and instructor detail
                TypedQuery<Instructor> query = entityManager.createQuery("SELECT i FROM Instructor i JOIN FETCH i.courses JOIN FETCH i.instructorDetail WHERE i.id=:instructorId", Instructor.class);

                query.setParameter("instructorId", Id);
                return query.getResultList();
            }

            public void update(Instructor instructor) { entityManager.merge(instructor); } 

            // @Transactional
            public void deleteById(int id) {
                Instructor instructor = entityManager.find(Instructor.class, id);
                // Here, instructor.getCourses() works even with LAZY loading
                // As this would have been within @Transactional scope
                // But, @Transactional is moved to service side. But this still works
                List<Courses> courses = instructor.getCourses();

                for(Course course: courses) { course.setInstructor(null); }

                // delete
                entityManager.remove(instructor);
            } 
        }

        public class TestService {
            ...
            @Transactional
            public void createInstructorWithCourses() {
                Instructor instructor = new Instructor("A", "B", "a@b.com");
                instructor.addCourse(new Course("Math"));
                instructor.addCourse(new Course("Science"));
                instructorDao.save(instructor);
            }

            public void findInstructorWithCourses() {
                Instructor instructor = instructorDao.findInstructorById(1);
                // Throws an error: falied to lazily initialize a collection
                // This is because Hibernate's session lifespan is only within instructorDao. Defined by @Transactional on instructorDao
                System.out.println("Courses: " + instructor.getCourses());

                // Solution. make use of JOIN FETCH query
                instructor = instructorDao.findInstructorByIdJoinFetch(1);
                System.out.println("Courses: " + instructor.getCourses());
            }

            @Transactional
            public void findCoursesByInstructorId() {
                Instructor instructor = instructorDao.findInstructorById(1);
                List<Course> courses = courseDao.findCoursesByInstructorId(1);

                instructor.setCourses(courses);
                // Works
                System.out.println("Courses: " + instructor.getCourses());
            }

            @Transactional
            public void updateInstructor() {
                Instructor instructor = instructorDao.findInstructorById(1);
                instructor.setLastName("TESTER");
                instructorDao.update(instructor);
            }

            @Transactional
            public void updateCourse() {
                Course course = courseDao.findCourseById(1);
                courseDao.setTitle("Testing course");
                courseDao.update(course);
            }

            @Transactional
            public void deleteInstructor() { instructorDao.deleteById(1); }

            @Transactional
            public void deleteCourse() { courseDao.deleteById(1); }
        }
        ```


        1. `JOIN FETCH` is similar to EAGER loading. It eagerly loads dependent entities ==even when `FetchType.LAZY`==. This is useful for when you want to methods to:
            - fetch instrcutor without courses. `findInstructorById`
            - fetch instructor with courses. `findInstructorByIdJoinFetch`

    === "Uni-directional"

        ```sql title="Schema"
        -- Table structure for table `review`
        CREATE TABLE `review` (
            `id` INT NOT NULL AUTO_INCREMENT,
            `comment` VARCHAR(256) DEFAULT NULL,
            `course_id` INT DEFAULT NULL,
            PRIMARY KEY (`id`),
            KEY `FK_COURSE_idx` (`course_id`),
            CONSTRAINT `FK_COURSE` FOREIGN KEY (`course_id`) REFERENCES `course` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
        ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

        -- Table structure for table `course`
        CREATE TABLE `course` (
            `id` INT NOT NULL AUTO_INCREMENT,
            `title` VARCHAR(125) DEFAULT NULL,
            `instructor_id` INT DEFAULT NULL,
            PRIMARY KEY (`id`)
        ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
        ```

        ```java title="Mapping"
        @Entity
        @Table(name="review")
        public class Review {
            @Id
            @GeneratedValue(strategy=GenerationType.IDENTITY)
            @Column(name="id")
            private int id;

            @Column(name="comment")
            private String comment;
        }

        @Entity
        @Table(name="course")
        public class Course {
            ...
            @OneToMany(fetch=FetchType.LAZY, cascade=CascadeType.ALL)
            @JoinColumn(name="course_id")
            private List<Review> reviews;

            public void addReview(Review review) {
                if(reviews == null) reviews = new ArrayList<>();
                reviews.add(review);
            }
        }

        @Repository
        public class CourseDao {
            ...
            public void save(Course course) {
                // This will save the course and associated reviews. due to CascasdeType.ALL
                return entityManager.persist(Course.class, id);
            }

            public Course findCourseAndReviewsById(int Id) {
                TypedQuery<Course> query = entityManager.createQuery("SELECT c FROM Course c JOIN FETCH c.reviews WHERE c.id=:courseId", Course.class);
                query.setParameter("courseId", Id);
                return query.getResultList();
            }

            public void deleteById(int Id) {
                // This will delete the course and associated reviews. due to CascasdeType.ALL
                Course course = entityManager.find(Course.class, id);
                entityManager.remove(course);
            }
        }

        public class TestService {
            ...
            @Transactional
            public void createCourseAndReviews() {
                Course course = new Course("TEsting courese");
                course.addReview(new Review("Best course ever"));
                course.addReview(new Review("Needs some improvement"));
                courseDao.save(course);
            }

            public void retrieveCourseAndReviews() {
                Course course = courseDao.findCourseAndReviewsById(1);
                System.out.println("Course review: " + course.getReviews());
            }

            @Transactional
            public void deleteCourseAndReviews() {
                courseDao.deleteById(1);
            }
        }
        ```


=== "Many-To-Many"

    Makes use of Join Table. Join Table provides a mapping between 2 tables. It has foreign keys for each table to define the mapping relationship.

    - Every many-to-many associations has 2 sides. The owning side and the inverse / non-owning side.
    - If the association is Bi-directional
        - Either side may be designated as the owning side. Owning side must use `@JoinTable`
        - ==The non-owning side must use the __mappedBy__ element of the ManyToMany annotation. To specify the relationship field or property of the owning side.==

    ```sql title="Schema"
    -- Table structure for JOIN table `course_student`
    CREATE TABLE `course_student` (
        `course_id` INT NOT NULL,
        `student_id` INT NOT NULL,
        PRIMARY KEY (`course_id`, `student_id`),
        KEY `FK_COURSE_idx` (`course_id`),
        KEY `FK_STUDENT_idx` (`student_id`),
        CONSTRAINT `FK_COURSE` FOREIGN KEY (`course_id`) REFERENCES `course` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
        CONSTRAINT `FK_STUDENT` FOREIGN KEY (`student_id`) REFERENCES `student` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    -- Table structure for table `course`
    CREATE TABLE `student` (
        `id` INT NOT NULL AUTO_INCREMENT,
        `first_name` VARCHAR(45) DEFAULT NULL,
        `last_name` VARCHAR(45) DEFAULT NULL,
        `email` VARCHAR(45) DEFAULT NULL,
        PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    -- Table structure for table `course`
    CREATE TABLE `course` (
        `id` INT NOT NULL AUTO_INCREMENT,
        `title` VARCHAR(125) DEFAULT NULL,
        PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
    ```

    === "Bi-directional"

        ```java
        @Entity
        @Table(name="course")
        public class Course {
            ...
            @ManyToMany(fetch=FetchType.LAZY, cascade={CascadeType.DETACH, CascadeType.MERGE,
            CascadeType.PERSIST, CascadeType.REFRESH})
            @JoinTable(name="course_student",
                joinColumns=@JoinColumn(name="course_id"),
                inverseJoinColumns=@JoinColumn(name="student_id"))
            private Set<Student> students = new HashSet<>();

            // Utility method to maintain bidirectional relationship
            public void addStudent(Student student) {
                // Avoid duplicate utility updates
                if (students.contains(student)) return;
                students.add(student);
                student.getCourses().add(this);
            }
        }

        @Entity
        @Table(name="student")
        public class Student {
            @Id
            @GeneratedValue(strategy=GenerationType.IDENTITY)
            @Column(name="id")
            private int id;

            @Column(name="first_name")
            private String firstName;

            @Column(name="last_name")
            private String lastName;

            @Column(name="email")
            private String email;

            // Refer `students` property of Course class and use the information from the it's @JoinTable.
            // To help find associated courses for student
            @ManyToMany(mappedBy="students", 
                fetch=FetchType.LAZY, 
                cascade={CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH})
            private Set<Course> courses = new HashSet<>();

            // Utility method to maintain bidirectional relationship
            public void addCourse(Course course) {
                // Avoid duplicate utility updates
                if (courses.contains(course)) return;
                courses.add(course);
                course.getStudents().add(this);
            }
        }

        @Repository
        public class CourseDao {
            public Course findCourseAndStudentsById(int Id) {
                TypedQuery<Course> query = entityManager.createQuery("SELECT c FROM Course c JOIN FETCH c.students WHERE c.id=:courseId", Course.class);
                query.setParameter("courseId", Id);
                return query.getResultList();
            }

            public void deleteById(int Id) {
                Course course = entityManager.find(Course.class, 1);
                // Remove this course from all students' course lists
                for(Student student: course.getStudents()) { student.getCourses().remove(course); }

                // Clear the students list from the course to avoid foreign key issues
                // required to update owning side (Course)
                course.getStudents().clear();

                entityManager.remove(course);
            }
        }

        @Repository
        public class StudentDao {
            public Student findStudentAndCoursesById(int Id) {
                TypedQuery<Student> query = entityManager.createQuery("SELECT s FROM Student s JOIN FETCH s.courses WHERE s.id=:studentId", Student.class);
                query.setParameter("studentId", Id);
                return query.getResultList();
            }

            public void update(Student student) {
                entityManager.merge(student);
            }

            // Remove associations
            public void deleteById(int Id) {
                Student student = entityManager.find(Student.class, 1);

                // Remove student from all courses before deletion
                // ✅ modify owning side
                for(Student course: student.getCourses()) { course.getStudents().remove(student); }

                // Clear student's course list to avoid constraint issues
                // ✅ optional, to sync inverse side in Java 
                student.getCourses().clear();

                entityManager.remove(student);
            }
        }

        public class TestService {
            @Transactional
            public void createCourseAndStudents() {
                Course course = new Course("Math");
                Student student1 = new Student("John", "Doe", "j.d@gmail.com");
                Student student2 = new Student("Marry", "Doe", "m.d@gmail.com");
                course.addStudent(student1);
                course.addStudent(student2);

                courseDao.save(course);
            }

            @Transactional
            public void addCoursesToStudent() {
                Student student = studentDao.findById(1);
                student.addCourse(new Course("History"));
                student.addCourse(new Course("Ecnomics"));
                studentDao.update(student);
            }

            public void findCourseAndStudents() {
                Course course = courseDao.findCourseAndStudentsById(1);
                System.out.println("Course: " + course);
                System.out.println("Students: " + course.getStudents());
            }

            public void findStudentAndCourses() {
                Student student = studentDao.findStudentAndCoursesById(1);
                System.out.println("Student: " + student);
                System.out.println("Courses: " + student.getCourses());
            }

            @Transactional
            public void deleteCourse() {
                courseDao.deleteById(1);
            }

            @Transactional
            public void deleteStudent() {
                studentDao.deleteById(1);
            }
        }
        ```

        ??? warning "Set v/s List"

            When defining a many-to-many relationship, we should consider using a Set instead of a List. As a JPA implementation, Hibernate doesn’t remove entities from a List in an efficient way.

            When using a List, Hibernate removes all entities from the junction table and inserts the remaining ones. This can cause performance issues. We can easily avoid this problem by using Set.

            __Set__:

            - Prevents Duplicates: Hibernate handles relationships using a Set, ensuring that duplicate entities are not stored.
            - Better Performance: Hibernate optimizes Set operations, reducing unnecessary SQL queries when checking for duplicates.
            - Efficient Join Table Handling: JPA correctly handles Set relationships without redundant insert operations.
            - No Ordering Guarantee: If ordering is required, `LinkedHashSet` or `@OrderBy` must be used.

            __List__:

            - Maintains Ordering: If the order of relationships matters, List should be used.
            - Works Well with `@OrderBy`: Helps retrieve data in a predefined order.
            - Allows Duplicates: You must manually ensure uniqueness (Hibernate does not enforce it).
            - Slower Performance for Large Datasets: Checking for duplicates requires extra queries or processing.

<hr/>

??? info "`@OnDelete`"

    Consider a one-to-many relationship between User and Post where deleting a User should delete all associated Post records. Hibernate will execute separate delete queries for each related Post, which can cause performance issues.

    The `OnDelete` annotation in Hibernate automatically removes related child records when a parent entity is deleted. It delegates deletion handling to the database instead of Java code. No extra Hibernate queries—just a single SQL DELETE statement.

    __How It Works at the Database Level?__

    When you apply `@OnDelete(action = OnDeleteAction.CASCADE)`, Hibernate generates a foreign key constraint in the database:

    ```sql
    ALTER TABLE post ADD CONSTRAINT fk_post_user
    FOREIGN KEY (user_id) REFERENCES user(id) 
    ON DELETE CASCADE;
    ```

    When to Use?

    - Best for large datasets to improve performance.
    - Prevents Hibernate from executing multiple queries for dependent deletions.
    - Useful for Many-to-One and One-to-Many relationships.
    - Prevents orphaned records in join tables for @OneToMany and @ManyToMany relationships.

    When not to Use?

    - Avoid using in Many-to-Many relationships: Use explicit delete logic instead.
    - Avoid using if you need soft deletes (marking as inactive instead of deleting).
    
    ```java title="One-To-Many"
    @Entity
    public class Employee {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        
        private String name;

        @ManyToOne
        @JoinColumn(name = "department_id", nullable = false)
        @OnDelete(action = OnDeleteAction.CASCADE)
        private Department department;
    }
    ```

    ```java title="Many-To-Many"
    @Entity
    public class Student {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        private String name;

        @ManyToMany
        @JoinTable(
            name = "student_course",
            joinColumns = @JoinColumn(name = "student_id"),
            inverseJoinColumns = @JoinColumn(name = "course_id")
        )
        @OnDelete(action = OnDeleteAction.CASCADE)
        private Set<Course> courses = new HashSet<>();
    }
    ```