# Title
ADR 001: Choosing MongoDB as the Database

## Context
We need to select a database for our application, which requires efficient handling of semi-structured data, scalability, and flexibility for future feature expansions. The system deals with frequently changing data structures and high read/write operations.

## Decision
We have decided to use MongoDB as our database.

## Rationale
Schema Flexibility – MongoDB’s document-based model allows dynamic and evolving data structures, making it suitable for our use case where requirements may change over time.
  
Scalability – MongoDB supports horizontal scaling via sharding, which will help handle growing data loads efficiently.
  
Performance – It provides high-speed read/write operations, suitable for applications requiring frequent updates and fast queries.

Ease of Development – The JSON-like document format makes it easier to integrate with JavaScript/Node.js and other modern web frameworks.

Rich Querying – MongoDB provides powerful query capabilities, including aggregation pipelines, text search, and geospatial queries.

Cloud Support – MongoDB Atlas provides a fully managed cloud database solution, reducing infrastructure overhead.

## Consequences
Pro: MongoDB allows schema-less data storage, making it easy to modify document structures over time.

Con: Lack of strict schemas can lead to inconsistent data if proper validation is not enforced using Mongoose or application logic.

Pro: MongoDB is designed for horizontal scaling, allowing it to handle large amounts of data by distributing it across multiple servers (sharding).

Con: Complex queries (like joins) require manual aggregation instead of simple SQL joins, making development more challenging.

Pro: Proper indexing improves read performance significantly.

Con: Too many indexes slow down write operations because indexes must be updated on every insert/update.


## Sample code
Give some sample code related to this decision.

Set Up MongoDB in Spring Boot

xml:
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>

Connecting to MongoDB

application.properties:
spring.data.mongodb.uri=mongodb://localhost:27017/mydatabase


Defining a Schema with Flexibility

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.util.List;

@Document(collection = "users")
public class User {

    @Id
    private String id;
    private String username;
    private String email;
    private Profile profile;

    public User(String username, String email, Profile profile) {
        this.username = username;
        this.email = email;
        this.profile = profile;
    }

    // Getters and Setters...

    public static class Profile {
        private Integer age;
        private List<String> interests;

        public Profile(Integer age, List<String> interests) {
            this.age = age;
            this.interests = interests;
        }

        // Getters and Setters...
    }
}

Create a Repository (MongoDB CRUD)

import org.springframework.data.mongodb.repository.MongoRepository;

public interface UserRepository extends MongoRepository<User, String> {
    User findByUsername(String username);
}




Handling CRUD Operations

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service

public class UserService {

    @Autowired
    private UserRepository userRepository;

    // Create (Insert a User)
    public void createUser(String username, String email, Integer age, List<String> interests) {
        User.Profile profile = new User.Profile(age, interests);
        User user = new User(username, email, profile);
        userRepository.save(user);
        System.out.println("User Created: " + username);
    }

    // Read (Find a User)
    public User getUserByUsername(String username) {
        return userRepository.findByUsername(username);
    }

    // Update (Modify User Data)
    public void updateUserAge(String username, Integer newAge) {
        User user = userRepository.findByUsername(username);
        if (user != null) {
            user.getProfile().setAge(newAge);
            userRepository.save(user);
            System.out.println("User Updated: " + username);
        }
    }

    // Delete (Remove a User)
    public void deleteUser(String username) {
        User user = userRepository.findByUsername(username);
        if (user != null) {
            userRepository.delete(user);
            System.out.println("User Deleted: " + username);
        }
    }
}
