# Experiment-9



// SpringDIExample.java
package com.example.springdi;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.*;

class Course {
    public String getCourseName() {
        return "Computer Science";
    }
}

class Student {
    private Course course;

    public Student(Course course) {
        this.course = course;
    }

    public void displayInfo() {
        System.out.println("Student is enrolled in: " + course.getCourseName());
    }
}

@Configuration
class AppConfig {

    @Bean
    public Course course() {
        return new Course();
    }

    @Bean
    public Student student() {
        return new Student(course());
    }
}

public class SpringDIExample {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        Student student = context.getBean(Student.class);
        student.displayInfo();
    }
}







// HibernateCRUDExample.java
package com.example.hibernatedemo;

import javax.persistence.*;
import org.hibernate.*;
import org.hibernate.cfg.Configuration;

@Entity
@Table(name = "student")
class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name")
    private String name;

    @Column(name = "marks")
    private float marks;

    public Student() {}

    public Student(String name, float marks) {
        this.name = name;
        this.marks = marks;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public float getMarks() { return marks; }

    public void setName(String name) { this.name = name; }
    public void setMarks(float marks) { this.marks = marks; }
}

public class HibernateCRUDExample {

    private static SessionFactory factory = new Configuration().configure().buildSessionFactory();

    public static void main(String[] args) {
        HibernateCRUDExample example = new HibernateCRUDExample();

        // CREATE
        Student s1 = new Student("Amit Sharma", 88.5f);
        example.addStudent(s1);

        // READ
        Student student = example.getStudent(1);
        System.out.println("Retrieved: " + student.getName() + " - " + student.getMarks());

        // UPDATE
        student.setMarks(92.0f);
        example.updateStudent(student);

        // DELETE
        example.deleteStudent(1);

        factory.close();
    }

    public void addStudent(Student student) {
        Session session = factory.openSession();
        Transaction tx = session.beginTransaction();
        session.save(student);
        tx.commit();
        session.close();
    }

    public Student getStudent(int id) {
        Session session = factory.openSession();
        Student student = session.get(Student.class, id);
        session.close();
        return student;
    }

    public void updateStudent(Student student) {
        Session session = factory.openSession();
        Transaction tx = session.beginTransaction();
        session.update(student);
        tx.commit();
        session.close();
    }

    public void deleteStudent(int id) {
        Session session = factory.openSession();
        Transaction tx = session.beginTransaction();
        Student student = session.get(Student.class, id);
        if (student != null) {
            session.delete(student);
        }
        tx.commit();
        session.close();
    }
}









// SpringHibernateTransactionExample.java
package com.example.bankingsystem;

import javax.persistence.*;
import org.hibernate.*;
import org.hibernate.cfg.Configuration;
import org.springframework.context.annotation.*;
import org.springframework.stereotype.*;
import org.springframework.transaction.annotation.*;

@Entity
@Table(name = "account")
class Account {
    @Id
    @Column(name = "account_id")
    private int accountId;

    @Column(name = "balance")
    private double balance;

    public Account() {}

    public Account(int accountId, double balance) {
        this.accountId = accountId;
        this.balance = balance;
    }

    public int getAccountId() { return accountId; }
    public double getBalance() { return balance; }

    public void setBalance(double balance) { this.balance = balance; }
}

@Repository
class AccountDAO {

    private static SessionFactory factory = new Configuration().configure().buildSessionFactory();

    public Account getAccount(int id) {
        Session session = factory.openSession();
        Account account = session.get(Account.class, id);
        session.close();
        return account;
    }

    public void updateAccount(Account account) {
        Session session = factory.openSession();
        Transaction tx = session.beginTransaction();
        session.update(account);
        tx.commit();
        session.close();
    }
}

@Service
class BankingService {

    private final AccountDAO accountDAO;

    public BankingService(AccountDAO accountDAO) {
        this.accountDAO = accountDAO;
    }

    @Transactional
    public void transferFunds(int fromAccountId, int toAccountId, double amount) {
        Account from = accountDAO.getAccount(fromAccountId);
        Account to = accountDAO.getAccount(toAccountId);

        from.setBalance(from.getBalance() - amount);
        to.setBalance(to.getBalance() + amount);

        accountDAO.updateAccount(from);
        accountDAO.updateAccount(to);
    }
}

@Configuration
@EnableTransactionManagement
class AppConfig {

    @Bean
    public AccountDAO accountDAO() {
        return new AccountDAO();
    }

    @Bean
    public BankingService bankingService() {
        return new BankingService(accountDAO());
    }
}

public class SpringHibernateTransactionExample {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        BankingService service = context.getBean(BankingService.class);

        service.transferFunds(1, 2, 1000.0);
        System.out.println("Transaction Successful!");
    }
}
