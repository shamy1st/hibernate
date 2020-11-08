# Hibernate
* **Hibernate** is a framework for persisting/saving java objects in a database.
  * provide the Object to Relational Mapping (ORM)

* **Config File**: hibernate.cfg.xml (path under /src) [JDBC connction + configurations]

* **Map Class to DB Table**

        @Entity
        @Table(name="Student)
        public class Student {
            ...
        }

* **Map Fields to DB Columns**

        @Entity
        @Table(name="Student)
        public class Student {
            @Id
            @Column(name="id")
            private int id;

            @Column(name="first_name")
            private String firstName;
        }

### Primary Key
* uniquely identifies each row in a table.
* must be unique & not null

        CREATE TABLE student (
            id int(11) NOT NULL AUTO_INCREMENT,
            ...,
            PRIMARY KEY(id)
        )

        @Id
        @GenerateValue(strategy=GenerationType.IDENTITY)
        @Column(name="id")
        private int id;

* **Generation Strategies**

    strategy                | description
    ------------------------|------------------------------------------------------------
    GenerationType.AUTO     | pick an appropriate strategy for the particular DB
    GenerationType.IDENTITY | assign PK using DB identity column
    GenerationType.SEQUENCE | assign PK using DB sequence
    GenerationType.TABLE    | assign PK using an underlying DB table to ensure uniqueness

* **Custom Generation Strategy**
    * create implementation of **org.hibernate.id.IdentifierGenerator**
    * override the method: public Serializable generate(...)
    
* **Starting value of PK**

        ALTER TABLE schema.table_name AUTO_INCREMENT=3000;

* **Reset Auto Increment & Delete all rows**

        TRUNCATE schema.table_name;

### CRUD

* **create**

        session.save(student);

* **read**

        Student student = session.get(Student.class, Student.getId());

* **update**

        Student student = session.get(Student.class, Student.getId());
        student.setFirstName("ahmed");
        session.getTransaction().commit();

* **delete**

        session.delete(student);

* **createQuery**

        List<Student> students = session.createQuery("from Student s where s.firstName='ahmed'").getResultList();

        session.createQuery("update Student set email='any@gmail.com'").executeUpdate();

        session.createQuery("delete from Student where id=2").executeUpdate();

### Entity Lifecycle
![](https://github.com/shamy1st/hibernate/blob/main/entity-lifecycle.png)

operation | description
----------|---------------------------------------------------------------------------------
Detach    | entity is not associated with a Hibernate session
Merge     | if entity is detached from session, then merge will reattach it to session
Persist   | transitions new instances to managed state, next flush/commit will save in DB
Remove    | transitions managed entity to be removed, next flush/commit will delete from DB
Refresh   | Reload/Synch object with data from DB. this prevent stale data.

### Cascade

type    | description
--------|---------------------------------------------------------------------
Persist | if entity is persisted/saved, related entity will also be persisted
Remove  | if entity is removed/deleted, related entity will also be deleted
Refresh | if entity is refreshed, related entity will also be refreshed
Detach  | if entity is detached, related entity will also be detached
Merge   | if entity is merged, related entity will also be merged
All     | all of above cascade types

* **@OneToOne** example Instructor, InstructorDetail
    * **Uni Directional**
    * **Bi Directional**
    
            public class Instructor {
                @OneToOne(cascade=CascadeType.All)
                @JoinColumn(name="instructor_detail_id")
                private InstructorDetail instructorDetail;
                ...
            }
            
            public class InstructorDetail {
                @OneToOne(mappedBy="instructorDetail")
                private Instructor instructor;
                ...
            }
    
   * **Prevent Cascade Delete**

            public class InstructorDetail {
                @OneToOne(mappedBy="instructorDetail", cascade={CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFESH})
                private Instructor instructor;
                ...
            }

* **@OneToMany @ManyToOne** example Course, Instructor (**Bi Directional**)

        public class Course {
            @ManyToOne(cascade={CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFESH})
            @JoinColumn(name="instrcutor_id")
            private Instructor instructor;
            ....
        }

        public class Instructor {
            @OneToMany(mappedBy="instructor", cascade={CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFESH})
            private List<Course> courses;
        }

### Fetch Types

* Two Types
    * **Eager**: retrieve everything
    * **Lazy**: retrieve on request

mapping     | default fetch type
------------|-------------------
@OneToOne   | FetchType.EAGER
@OneToMany  | FetchType.LAZY
@ManyToOne  | FetchType.EAGER
@ManyToMany | FetchType.LAZY

* you can override the FetchType

        @ManyToOne(fetch=FetchType.LAZY)
        @JoinColumn(name="instructor_id")
        private Instructor instructor;

```diff
- Warn: for LAZY you should have open session, if the session closed it will throw exception.
```

* **solution**: use "JOIN FETCH" in the query to load everything at once.

        Query<Instructor> query = session.createQuery("select i from Instructor i" 
            + "JOIN FETCH i.courses where i.id=: instructorId", Instructor.class);

* **@OneToMany** (Uni Directional) example Course, Review

        public class Course {
            @OneToMany(fetch=FetchType.LAZY, cascade=CascadeType.All)
            @JoinColumn(name="course_id")
            private List<Review> reviews;
            ...
        }
        
        public class Review {
            //nothing
            ...
        }

* **@ManyToMany** example Course, Student

        public class Course {
            @ManyToMany
            @JoinTable(name="course_student", joinColumns=@JoinColumn(name="course_id")
                , inverseJoinColumns=@JoinColumn(name="student_id"))
            private List<Student> students;
            ...
        }
        
        public class Student {
            @ManyToMany
            @JoinTable(name="course_student", joinColumns=@JoinColumn(name="student_id")
                , inverseJoinColumns=@JoinColumn(name="course_id"))
            private List<Course> courses;
        }

* **LAZY loading & Prevent cascade delete**

        @ManyToMany(fetch=FetchType.LAZY, cascade={CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFESH})
        @JoinTable(...)

