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

























