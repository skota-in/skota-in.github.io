# Hibernate Annotations Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of Hibernate annotations, which are used to map Java objects to database tables and define relationships between entities. The guide covers all major annotations with practical examples and best practices.

## Table of Contents

1. [Entity Mapping Annotations](#entity-mapping-annotations)
2. [Primary Key Annotations](#primary-key-annotations)
3. [Column Mapping Annotations](#column-mapping-annotations)
4. [Relationship Annotations](#relationship-annotations)
5. [Inheritance Annotations](#inheritance-annotations)
6. [Query Annotations](#query-annotations)
7. [Validation Annotations](#validation-annotations)
8. [Cache Annotations](#cache-annotations)
9. [Best Practices](#best-practices)
10. [References](#references)

## Entity Mapping Annotations

### 1. @Entity
Marks a class as a JPA entity.

```java
@Entity
public class User {
    // Entity properties
}
```

### 2. @Table
Specifies the table name and schema.

```java
@Entity
@Table(name = "users", schema = "public")
public class User {
    // Entity properties
}
```

### 3. @Embeddable
Marks a class as embeddable.

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String zipCode;
}
```

### 4. @Embedded
Specifies an embedded object.

```java
@Entity
public class User {
    @Embedded
    private Address address;
}
```

## Primary Key Annotations

### 1. @Id
Marks a field as the primary key.

```java
@Entity
public class User {
    @Id
    private Long id;
}
```

### 2. @GeneratedValue
Specifies the generation strategy for primary keys.

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

### 3. @SequenceGenerator
Defines a sequence generator.

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
    @SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)
    private Long id;
}
```

### 4. @TableGenerator
Defines a table generator.

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "user_table")
    @TableGenerator(name = "user_table", table = "id_generator", pkColumnName = "gen_name", valueColumnName = "gen_value")
    private Long id;
}
```

## Column Mapping Annotations

### 1. @Column
Specifies column properties.

```java
@Entity
public class User {
    @Column(name = "user_name", nullable = false, length = 50)
    private String username;
}
```

### 2. @Temporal
Specifies temporal type for date/time fields.

```java
@Entity
public class User {
    @Temporal(TemporalType.DATE)
    private Date birthDate;
    
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastLogin;
}
```

### 3. @Enumerated
Maps enum values.

```java
@Entity
public class User {
    @Enumerated(EnumType.STRING)
    private UserStatus status;
}
```

### 4. @Lob
Maps large objects.

```java
@Entity
public class User {
    @Lob
    private String bio;
    
    @Lob
    private byte[] profilePicture;
}
```

## Relationship Annotations

### 1. @OneToOne
Defines a one-to-one relationship.

```java
@Entity
public class User {
    @OneToOne
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}
```

### 2. @OneToMany
Defines a one-to-many relationship.

```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders;
}
```

### 3. @ManyToOne
Defines a many-to-one relationship.

```java
@Entity
public class Order {
    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

### 4. @ManyToMany
Defines a many-to-many relationship.

```java
@Entity
public class User {
    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles;
}
```

## Inheritance Annotations

### 1. @Inheritance
Specifies inheritance strategy.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "user_type")
public abstract class User {
    // Common properties
}
```

### 2. @DiscriminatorValue
Specifies discriminator value.

```java
@Entity
@DiscriminatorValue("ADMIN")
public class AdminUser extends User {
    // Admin specific properties
}
```

### 3. @MappedSuperclass
Marks a class as a mapped superclass.

```java
@MappedSuperclass
public abstract class BaseEntity {
    @Id
    @GeneratedValue
    private Long id;
    
    @Version
    private Long version;
}
```

## Query Annotations

### 1. @NamedQuery
Defines a named query.

```java
@Entity
@NamedQuery(
    name = "User.findByStatus",
    query = "SELECT u FROM User u WHERE u.status = :status"
)
public class User {
    // Entity properties
}
```

### 2. @NamedQueries
Defines multiple named queries.

```java
@Entity
@NamedQueries({
    @NamedQuery(name = "User.findByStatus", query = "SELECT u FROM User u WHERE u.status = :status"),
    @NamedQuery(name = "User.findByEmail", query = "SELECT u FROM User u WHERE u.email = :email")
})
public class User {
    // Entity properties
}
```

### 3. @NamedNativeQuery
Defines a native SQL query.

```java
@Entity
@NamedNativeQuery(
    name = "User.findByStatusNative",
    query = "SELECT * FROM users WHERE status = :status",
    resultClass = User.class
)
public class User {
    // Entity properties
}
```

## Validation Annotations

### 1. @NotNull
Ensures a field is not null.

```java
@Entity
public class User {
    @NotNull
    private String username;
}
```

### 2. @Size
Specifies size constraints.

```java
@Entity
public class User {
    @Size(min = 3, max = 50)
    private String username;
}
```

### 3. @Email
Validates email format.

```java
@Entity
public class User {
    @Email
    private String email;
}
```

### 4. @Pattern
Validates against a regex pattern.

```java
@Entity
public class User {
    @Pattern(regexp = "^[A-Za-z0-9]+$")
    private String username;
}
```

## Cache Annotations

### 1. @Cacheable
Enables caching for an entity.

```java
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {
    // Entity properties
}
```

### 2. @Cache
Specifies cache strategy.

```java
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE, region = "userCache")
public class User {
    // Entity properties
}
```

## Best Practices

1. **Entity Design**:
   - Use meaningful entity and table names
   - Implement proper primary key strategy
   - Consider inheritance strategies carefully
   - Use appropriate relationship types

2. **Performance**:
   - Use lazy loading for relationships
   - Implement proper caching strategies
   - Optimize queries
   - Use appropriate fetch strategies

3. **Validation**:
   - Implement proper validation constraints
   - Use appropriate data types
   - Consider database constraints
   - Validate business rules

4. **Maintenance**:
   - Document complex relationships
   - Use consistent naming conventions
   - Keep annotations organized
   - Follow naming standards

## References

- [Hibernate ORM Documentation](https://hibernate.org/orm/documentation/)
- [JPA Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate Validator Documentation](https://hibernate.org/validator/documentation/)
- [Hibernate Cache Documentation](https://hibernate.org/orm/documentation/caching/) 