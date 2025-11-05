### Q) Interservice communication between Microservices using spring boot 

In Spring Boot, several mechanisms are available to achieve interservice communication, depending on the architecture style (synchronous or asynchronous) and reliability needs.

1. Synchronous Communication (Request-Response Style)
     one service directly calls another and waits for the response.
     Simple, blocking, and synchronous.

```
 A. REST Communication using RestTemplate (Legacy Approach)

@Service
public class UserService {

    @Autowired
    private RestTemplate restTemplate;

    public Order getOrderDetails(Long orderId) {
        String url = "http://ORDER-SERVICE/orders/" + orderId;
        return restTemplate.getForObject(url, Order.class);
    }
}
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
B. Reactive Communication using WebClient

Non-blocking, asynchronous, and scalable.
Part of Spring WebFlux.

@Service
public class UserService {

    private final WebClient webClient = WebClient.create("http://ORDER-SERVICE");

    public Mono<Order> getOrderDetails(Long orderId) {
        return webClient.get()
                .uri("/orders/{id}", orderId)
                .retrieve()
                .bodyToMono(Order.class);
    }
}

C. Declarative REST with Spring Cloud OpenFeign
     Simplifies REST communication using interfaces.
     Auto-handles serialization, URL building, and load balancing (with Ribbon/Eureka).

@FeignClient(name = "ORDER-SERVICE")
public interface OrderClient {
    @GetMapping("/orders/{id}")
    Order getOrderById(@PathVariable Long id);
}

@Service
public class UserService {

    @Autowired
    private OrderClient orderClient;

    public Order fetchOrder(Long orderId) {
        return orderClient.getOrderById(orderId);
    }
}

Configuration:
@SpringBootApplication
@EnableFeignClients
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```
     
3. Asynchronous Communication (Event-Driven Style)
   ```
   In asynchronous systems, services communicate via events or messages, improving decoupling and scalability.
   
   A. Using Message Brokers (Kafka, RabbitMQ, etc.)
        Each service publishes or consumes events instead of making direct calls.

   Kafka Example:

     Producer Service:
        @Service
public class OrderEventProducer {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void publishOrderCreatedEvent(String orderJson) {
        kafkaTemplate.send("order-topic", orderJson);
    }
}

Consumer Service:

@Service
@KafkaListener(topics = "order-topic", groupId = "user-service-group")
public class OrderEventConsumer {

    @KafkaHandler
    public void consume(String message) {
        System.out.println("Received order event: " + message);
    }
}

Benefits:
     Loosely coupled microservices.
     Services can continue functioning even if other services are down.
     Better for scalability and eventual consistency.

  3. Service Discovery   

     When services run dynamically (e.g., in containers), static URLs are unreliable.
     ✅ Spring Cloud Netflix Eureka or Consul
     Example:

          Register each microservice with Eureka.

          Use @LoadBalanced RestTemplate or Feign client with the service name (e.g., http://ORDER-SERVICE).

  4. API Gateway
       Often, an API Gateway (like Spring Cloud Gateway) is used:
          Routes requests to the correct service.
          Handles cross-cutting concerns like authentication, rate limiting, and logging.
     
     Example:
          spring:
  cloud:
    gateway:
      routes:
        - id: order_route
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/orders/**

5. Security Between Services
   Use OAuth2 / JWT / mTLS for secure service-to-service communication.   
   ```
| Approach         | Type  | Tool/Library           | Pros                       | Use Case                   |
| ---------------- | ----- | ---------------------- | -------------------------- | -------------------------- |
| `RestTemplate`   | Sync  | Spring Web             | Simple                     | Small apps                 |
| `WebClient`      | Async | Spring WebFlux         | Non-blocking               | Reactive apps              |
| `FeignClient`    | Sync  | Spring Cloud OpenFeign | Declarative                | Cloud-native microservices |
| Kafka/RabbitMQ   | Async | Messaging              | Decoupled                  | Event-driven systems       |
| Eureka + Gateway | Infra | Spring Cloud           | Service discovery, routing | Distributed systems        |
#####
__________________________________________________________________________________________________________________________________
#### Caching Mechanisam available in SpringBoot ####

In Spring Framework (and Spring Boot), caching is a mechanism that helps improve performance by storing method return values (or other frequently used data) so that subsequent calls can be served faster without re-executing the logic or database query.

Spring provides a unified caching abstraction, which means you can use the same annotations and configuration regardless of the actual cache provider (like Ehcache, Redis, Caffeine, etc.).

1. Spring Caching Abstraction
``` @EnableCaching ```

Key Annotations:

<li>
     @EnableCaching — Enables caching in your Spring Boot application.
     @Cacheable — Caches the result of a method execution.
     @CachePut — Updates the cache with the new result of a method.
     @CacheEvict — Removes data from the cache.
     @Caching — Allows multiple caching annotations on a single method.
     @CacheConfig — Shared configuration for cache names or key generators.
</li>

Example:
```
@Cacheable("employees")
     public Employee getEmployeeById(Long id) {
         return employeeRepository.findById(id).orElse(null);
     }
```

2. Common Cache Providers Supported

Spring integrates with multiple caching providers via the abstraction layer.
Here are the most popular ones:

| Cache Provider                  | Description                                                    | Dependency                                                |
| ------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------- |
| **Simple (ConcurrentMapCache)** | Default in-memory cache (good for dev/testing)                 | Included by default                                       |
| **Ehcache**                     | Popular, robust in-memory cache                                | `org.ehcache:ehcache`                                     |
| **Caffeine**                    | High-performance in-memory cache (modern alternative to Guava) | `com.github.ben-manes.caffeine:caffeine`                  |
| **Hazelcast**                   | Distributed in-memory data grid (for clustered caching)        | `com.hazelcast:hazelcast`                                 |
| **Infinispan**                  | Distributed cache by Red Hat                                   | `org.infinispan:infinispan-spring-boot-starter`           |
| **Redis**                       | Distributed cache (in-memory, network accessible)              | `org.springframework.boot:spring-boot-starter-data-redis` |
| **JCache (JSR-107)**            | Standard Java cache API (works with multiple providers)        | `javax.cache:cache-api`                                   |

3. Cache Annotations Summary

| Annotation     | Purpose                                    |
| -------------- | ------------------------------------------ |
| `@Cacheable`   | Skips method execution if data is in cache |
| `@CachePut`    | Always executes method and updates cache   |
| `@CacheEvict`  | Removes cache entries                      |
| `@Caching`     | Combines multiple caching rules            |
| `@CacheConfig` | Common cache settings at class level       |

4. Distributed / External Cache Options

When scalability is required:

<li>
     Redis → Most common choice (fast and distributed)
     Hazelcast / Infinispan → Clustered and JVM-based
     Memcached → Lightweight distributed cache
</li>

Example — Using Redis with Spring Boot

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
Configuration (application.yml):

```
spring:
  cache:
    type: redis
  data:
    redis:
      host: localhost
      port: 6379
```
Usage:

```
@Cacheable(value = "products", key = "#id")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElseThrow();
}
   
```
Summary

| Category         | Examples                                | Scope                  |
| ---------------- | --------------------------------------- | ---------------------- |
| **In-Memory**    | SimpleCache, Caffeine, Ehcache          | Local JVM              |
| **Distributed**  | Redis, Hazelcast, Infinispan, Memcached | Clustered / Multi-node |
| **Standard API** | JCache (JSR-107)                        | Pluggable Providers    |

___________________________________________________________________________________________________________________________________
#### cron scheduler in springboot. ####

<b> Step-by-Step Setup for Cron Scheduler </b>

1. Enable Scheduling
     Add the **@EnableScheduling** annotation to your main Spring Boot application class (or a configuration class).
   ```
     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.scheduling.annotation.EnableScheduling;

     @SpringBootApplication
     @EnableScheduling
     public class SchedulerApplication {
         public static void main(String[] args) {
             SpringApplication.run(SchedulerApplication.class, args);
         }
     }

   ```
            
2. Create a Scheduled Task
     Use the **@Scheduled** annotation on any method inside a **@Component** or **@Service** class.
     ```
          import org.springframework.scheduling.annotation.Scheduled;
          import org.springframework.stereotype.Component;

          @Component
          public class CronJobScheduler {
          
              // Runs every 10 seconds
              @Scheduled(cron = "*/10 * * * * *")
              public void runJob() {
                  System.out.println("Running scheduled job at " + java.time.LocalDateTime.now());
              }
          }
     ```

      
3. Understanding the Cron Expression
   Spring’s cron format has 6 fields (not 7 like Unix) </br>
      ┌───────────── second (0–59) <br/>
      │ ┌───────────── minute (0–59)  <br/>
      │ │ ┌───────────── hour (0–23)  <br/>
      │ │ │ ┌───────────── day of month (1–31)  <br/>
      │ │ │ │ ┌───────────── month (1–12 or JAN–DEC)  <br/>
      │ │ │ │ │ ┌───────────── day of week (0–7 or SUN–SAT)  <br/>
      │ │ │ │ │ │  <br/>
      * * * * * *
 
<br/>

✅ Examples:

     | Expression       | Description              |
     | ---------------- | ------------------------ |
     | `0 0 * * * *`    | Every hour at 00 minutes |
     | `0 0 9 * * *`    | Every day at 9:00 AM     |
     | `0 */5 * * * *`  | Every 5 minutes          |
     | `0 0 0 * * MON`  | Every Monday at midnight |
     | `*/10 * * * * *` | Every 10 seconds         |

   
   
4. Using Externalized Cron from application.properties
     You can define the cron expression in a property file for flexibility:
     ```
     app.cron.expression=*/15 * * * * *
     ``` 
     Then use it in your code:

     ```
      import org.springframework.beans.factory.annotation.Value;
     import org.springframework.scheduling.annotation.Scheduled;
     import org.springframework.stereotype.Component;

     @Component
     public class PropertyBasedScheduler {
     
     @Value("${app.cron.expression}")
     private String cronExpression;
     
     @Scheduled(cron = "${app.cron.expression}")
          public void runTask() {
             System.out.println("Running job as per cron: " + cronExpression);
          }
     }    
     ```
   
   
5. Fixed Delay / Fixed Rate (Alternate Options)
    If you don’t need cron but want fixed intervals:
   ```
     @Scheduled(fixedRate = 5000)  // Runs every 5 seconds (measured from start)
     public void runFixedRateTask() { ... }

     @Scheduled(fixedDelay = 5000) // Runs 5 seconds after previous completion
     public void runFixedDelayTask() { ... }

     @Scheduled(initialDelay = 10000, fixedRate = 5000)
     public void runDelayedTask() { ... }

   ```
     
6. Enable Concurrent or Sequential Execution
     By default, scheduled methods run in a **single thread**.
     To allow concurrent scheduling, configure a **TaskScheduler** bean:

   ```
     import org.springframework.context.annotation.Configuration;
     import org.springframework.scheduling.annotation.EnableScheduling;
     import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
     import org.springframework.context.annotation.Bean;

     @Configuration
     @EnableScheduling
     public class SchedulerConfig {
     
         @Bean
         public ThreadPoolTaskScheduler taskScheduler() {
             ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
             scheduler.setPoolSize(5);
             scheduler.setThreadNamePrefix("cron-task-");
             scheduler.initialize();
             return scheduler;
         }
     }
       
   ```

<br/>

✅ Summary

<br/>

| Type        | Annotation                    | Example                            |
| ----------- | ----------------------------- | ---------------------------------- |
| Cron-based  | `@Scheduled(cron="...")`      | `@Scheduled(cron="0 0/5 * * * *")` |
| Fixed Rate  | `@Scheduled(fixedRate=5000)`  | every 5 seconds                    |
| Fixed Delay | `@Scheduled(fixedDelay=5000)` | after 5 sec of previous run        |

__________________________________________________________________________________________________________________________________

<br/>
####Spring Batching Processing####
<br/>
<p>Spring Batch Processing is a lightweight, comprehensive framework designed for batch processing — i.e., executing a series of jobs or tasks without user interaction, often dealing with large volumes of data efficiently and reliably.</p>

<br/>
🔹 What is Batch Processing ?  <br/>

     Batch processing means executing a sequence of operations on a large dataset, typically: <br/>
          -Reading data from a source (DB, file, queue)
          -Processing/transformation logic
          -Writing the processed data to a target (DB, file, API, etc.) <br/>

     It’s commonly used for: <br/>
          -ETL (Extract, Transform, Load)
          -Report generation
          -Data migration
          -Payroll, billing, or reconciliation jobs <br/> 

🔹 Spring Batch Overview <br/>

     Spring Batch provides: <br/>
          -Transaction management<br/> 
          -Chunk-based processing<br/> 
          -Retry/restart capabilities<br/> 
          -Job scheduling and monitoring<br/> 
          -Scalability (parallel or partitioned steps)<br/> 
<br/>

🔹 Core Components <br/>

     | Component         | Description                                                   |
     | ----------------- | ------------------------------------------------------------- |
     | **Job**           | A container for the entire batch process (one or more steps). |
     | **Step**          | A single phase in a job — e.g., read → process → write.       |
     | **JobInstance**   | Represents a single run of a job with specific parameters.    |
     | **JobExecution**  | Represents a single attempt to run a JobInstance.             |
     | **JobRepository** | Stores job metadata (execution status, parameters, etc.).     |
     | **ItemReader**    | Reads data from a source (file, DB, etc.).                    |
     | **ItemProcessor** | Applies business logic/transformation.                        |
     | **ItemWriter**    | Writes the processed data to a destination.                   |

<br/>

🔹 Chunk-Oriented Processing <br/>
     A key feature of Spring Batch. <br/>
     * Data is processed in chunks (e.g., 100 records at a time). <br/>
     * Each chunk is read–processed–written as a transaction. <br/>

     Example:

     <chunk reader="itemReader" processor="itemProcessor" writer="itemWriter" commit-interval="100"/>

<br/>
🔹 Spring Batch Architecture <br/>

        +----------------------+
        |     JobLauncher      |
        +----------+-----------+
                   |
                   v
        +----------------------+
        |        Job           |
        +----------+-----------+
                   |
                   v
        +----------------------+
        |        Step          |
        +----------+-----------+
                   |
        +---------+---------+
         | Reader  |Processor|Writer|

<br/>

🔹 Example: Java Config <br/>

     ```
     @Configuration
     @EnableBatchProcessing
     public class BatchConfig {
     
         @Bean
         public ItemReader<String> reader() {
             return new FlatFileItemReaderBuilder<String>()
                     .name("stringItemReader")
                     .resource(new ClassPathResource("input.txt"))
                     .lineMapper((line, lineNumber) -> line)
                     .build();
         }
     
         @Bean
         public ItemProcessor<String, String> processor() {
             return item -> item.toUpperCase(); // example transformation
         }
     
         @Bean
         public ItemWriter<String> writer() {
             return items -> items.forEach(System.out::println);
         }
     
         @Bean
         public Step step(StepBuilderFactory stepBuilderFactory, 
                          ItemReader<String> reader,
                          ItemProcessor<String, String> processor,
                          ItemWriter<String> writer) {
             return stepBuilderFactory.get("step1")
                     .<String, String>chunk(5)
                     .reader(reader)
                     .processor(processor)
                     .writer(writer)
                     .build();
         }
     
         @Bean
         public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
             return jobBuilderFactory.get("job1")
                     .start(step)
                     .build();
         }
     }```

     <br/>
     🔹 Advanced Features </br>
          1. ** Job Parameters ** – pass dynamic data into a job (e.g., date or file name).
          2. ** Job Scheduling ** – integrate with Spring Scheduler or Quartz.
          3. ** Error Handling & Retry ** – skip, retry, or rollback failed records
          4. ** Parallel Processing ** – using partitioning, multi-threading, or remote chunking.
          5. ** Integration with Spring Boot ** – auto-configured setup and monitoring via Actuator.
          
</br>
🔹 Spring Boot + Spring Batch </br>

     Spring Boot simplifies configuration with:
          1. Auto-configuration of JobLauncher, JobRepository, etc.
          2. Running jobs automatically on startup (spring.batch.job.enabled=true).
          3 YAML/properties-based customization.

__________________________________________________________________________________________________________________________________
#### configuring Spring Boot to connect to and use two different database servers? ####

Let’s assume we have:
     Database 1: MySQL
     Database 2: PostgreSQL

     We’ll use Spring Data JPA for both.

     1. Add dependencies
     In your pom.xml (Maven) or build.gradle (Gradle):

     ```
     <!-- MySQL and PostgreSQL drivers -->
     
     <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
     </dependency>
     
     <dependency>
         <groupId>org.postgresql</groupId>
         <artifactId>postgresql</artifactId>
     </dependency>
     
     <!-- Spring Data JPA -->
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-jpa</artifactId>
     </dependency> ```


     2. Configure application.properties / application.yml
     
      ``` # MySQL datasource
               spring.datasource.mysql.url=jdbc:mysql://localhost:3306/db1
               spring.datasource.mysql.username=root
               spring.datasource.mysql.password=root
               spring.datasource.mysql.driver-class-name=com.mysql.cj.jdbc.Driver
               spring.jpa.mysql.hibernate.ddl-auto=update
               spring.jpa.mysql.show-sql=true
               
               # PostgreSQL datasource
               spring.datasource.postgres.url=jdbc:postgresql://localhost:5432/db2
               spring.datasource.postgres.username=postgres
               spring.datasource.postgres.password=postgres
               spring.datasource.postgres.driver-class-name=org.postgresql.Driver
               spring.jpa.postgres.hibernate.ddl-auto=update
               spring.jpa.postgres.show-sql=true
          ```
          <br/>
          
3. Create DataSource configuration

    ``` 
     @Configuration
     @EnableTransactionManagement
     @EnableJpaRepositories(
                             basePackages = "com.example.mysql.repository",
                             entityManagerFactoryRef = "mysqlEntityManagerFactory",
                             transactionManagerRef = "mysqlTransactionManager"
                          )
     public class MysqlDataSourceConfig {

         @Bean
         @Primary
         @ConfigurationProperties(
                                      prefix = "spring.datasource.mysql"
                                  )
         public DataSource mysqlDataSource() {
             return DataSourceBuilder.create().build();
         }
     
         @Bean
         @Primary
         public LocalContainerEntityManagerFactoryBean mysqlEntityManagerFactory(
                 EntityManagerFactoryBuilder builder) {
             return builder
                     .dataSource(mysqlDataSource())
                     .packages("com.example.mysql.entity")
                     .persistenceUnit("mysqlPU")
                     .build();
         }
     
         @Bean
         @Primary
         public PlatformTransactionManager mysqlTransactionManager(
                 @Qualifier("mysqlEntityManagerFactory") EntityManagerFactory emf) {
             return new JpaTransactionManager(emf);
         }
     }
    ```

<b>PostGres sql</b>

```
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = "com.example.postgres.repository",
    entityManagerFactoryRef = "postgresEntityManagerFactory",
    transactionManagerRef = "postgresTransactionManager"
)
public class PostgresDataSourceConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.postgres")
    public DataSource postgresDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean postgresEntityManagerFactory(
            EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(postgresDataSource())
                .packages("com.example.postgres.entity")
                .persistenceUnit("postgresPU")
                .build();
    }

    @Bean
    public PlatformTransactionManager postgresTransactionManager(
            @Qualifier("postgresEntityManagerFactory") EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
} ```

4. Define Entities & Repositories
     * MySQL entities in com.example.mysql.entity
     * PostgreSQL entities in com.example.postgres.entity

Repositories in corresponding packages as specified in @EnableJpaRepositories.

``` @Repository
public interface UserRepository extends JpaRepository<User, Long> {
} ```
<br/>

✅ Now your Spring Boot app can interact with both databases simultaneously,
 each with its own entities, repositories, and transactions.<br/>
__________________________________________________________________________________________________________________________________
#### Q) What are the steps to detect and resolve performance issues in a Spring Boot application?

1. Identify the Symptoms
     * Start by understanding how the application is performing:
     * High response times or latency.
     * Increased CPU or memory usage.
     * Slow database queries or high I/O wait.
     * Frequent timeouts or thread pool exhaustion.

2. Enable Monitoring and Metrics
     * Use Spring Boot’s built-in and external monitoring tools:
     * Spring Boot Actuator: Provides health checks, metrics, and endpoints like /actuator/metrics.
     * Micrometer: Integrates with monitoring systems like Prometheus, Grafana.
     * Monitor JVM metrics: CPU, memory, garbage collection, thread counts.

3. Profile the Application
     * Identify where the bottlenecks are:
     * Use Java profilers: VisualVM, YourKit, JProfiler.
     * Analyze heap dumps and thread dumps.
     * Look for slow methods, excessive object creation, or thread contention.

4. Analyze Database Performance
     * Database issues are often the root cause:
     * Enable SQL logging to see slow queries.
     * Use tools like Spring Data JPA statistics or Hibernate metrics.
     * Optimize queries, indexes, and consider caching.

5. Check Application Configuration
     * Review Spring Boot and system configurations:
     * Thread pools (for async operations, web requests) may need tuning.
     * Connection pool settings (HikariCP, Tomcat, etc.).
     * Caching: Use @Cacheable for frequently accessed data.
     * Compression and HTTP keep-alive for web performance.

6. Optimize Code
     * Common code-level improvements:
     * Reduce redundant object creation.
     * Avoid unnecessary blocking operations.
     * Optimize loops, streams, and collection usage.
     * Use asynchronous processing (@Async) where applicable.

7. Test and Benchmark
     * Validate changes with proper testing:
     * Use load testing tools: JMeter, Gatling, or Locust.
     * Measure before-and-after performance metrics.
     * Monitor for regression issues.

8. Implement Continuous Monitoring
     * After fixes:
     * Keep monitoring live application metrics.
     * Set up alerts for CPU, memory, or response time thresholds.
     * Review logs for errors and warnings regularly.

__________________________________________________________________________________________________________________________________
### Q) What are some recommended approaches for versioning REST APIs in Spring Boot?

1. URI Path Versioning (Most Common)

Example:

``` GET /api/v1/customers
    GET /api/v2/customers ```


Implementation:

``` @RestController
@RequestMapping("/api/v1/customers")
public class CustomerControllerV1 {
    @GetMapping
    public String getCustomersV1() { return "Customer data v1"; }
}

@RestController
@RequestMapping("/api/v2/customers")
public class CustomerControllerV2 {
    @GetMapping
    public String getCustomersV2() { return "Customer data v2"; }
} ```


Pros:
     * Simple and intuitive
     * Easy for clients to switch between versions
     * Clear separation of versions
cons:
     * URL clutter     
     * **Might duplicate controllers for each version**


2. Request Parameter Versioning

     Example:
          ``` GET /api/customers?version=1  
               GET /api/customers?version=2 ```

Implementation:


     ``` @RestController
          @RequestMapping("/api/customers")
          public class CustomerController {
              @GetMapping(params = "version=1")
              public String getCustomersV1() { return "Customer data v1"; }
     
              @GetMapping(params = "version=2")
              public String getCustomersV2() { return "Customer data v2"; }
          } ```


Pros:

     * Cleaner URLs
     * Easy for clients to specify version dynamically

Cons:

     * Versioning logic tied to query parameters
     * Can make caching less efficient


3. Header-Based Versioning (a.k.a. Custom Header or Accept Header Versioning)

    Example:

         ``` GET /api/customers  
             Headers: X-API-VERSION=1 ```

    or using Accept Header:


        ``` Accept: application/vnd.example.v1+json ```


     Implementation:


     ``` @RestController
          @RequestMapping("/api/customers")
          public class CustomerHeaderController {
              @GetMapping(headers = "X-API-VERSION=1")
              public String getCustomersV1()
               { return "Customer data v1"; }

              @GetMapping(headers = "X-API-VERSION=2")
              public String getCustomersV2()
                { return "Customer data v2"; }
          } ```   


Pros:

     * Keeps URLs clean
     * Supports content negotiation

Cons:

     * Harder to test/debug in browsers
     * Clients must handle headers properly


4. Media Type (Content Negotiation) Versioning

Example:

     ``` GET /api/customers
          Accept: application/vnd.example.v1+json ```



Implementation

     ``` @RestController
     @RequestMapping("/api/customers")
     public class CustomerMediaTypeController {
         @GetMapping(produces = "application/vnd.example.v1+json")
         public String getCustomersV1() { return "Customer data v1"; }

         @GetMapping(produces = "application/vnd.example.v2+json")
         public String getCustomersV2() { return "Customer data v2"; }
     } ```


Pros:

     * RESTful and aligns with HTTP standards
     * Good for advanced API clients

Cons:

     * Complex setup
     * Difficult for simple clients to use
     
5. Versioning via Subdomain or Hostname (Less Common)
     Example:
        
        ``` https://v1.api.example.com/customers
        https://v2.api.example.com/customers ```  
      
        Pros:

          * Useful for major version upgrades or isolated deployments

     Cons:

          * Requires DNS and infrastructure support
          * Overhead in deployment and maintenance  

   <br/>

   ✅ Best Practice Recommendations

**For internal APIs, use URI path versioning — it’s easiest to maintain.**
For **public or enterprise-grade APIs, use header-based or media type versioning to maintain cleaner URLs**.

Always document versions clearly using **OpenAPI/Swagger**.

Use semantic versioning (e.g., v1.1, v2.0) and **deprecate old versions** with proper notice.

____________________________________________________________________________________________________________________________
### Q) How to secure Spring Boot Actuator endpoints?

To secure Spring Boot Actuator endpoints, you can combine Spring Security with Actuator configuration.
Below are the recommended steps and approaches for protecting sensitive management endpoints like
/actuator/health, /actuator/metrics, etc

1. Add Required Dependencies
     If you don’t already have Spring Security, add it:

     Maven:

     ``` <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
     </dependency> ```


2. Configure Actuator Endpoint Exposure 

application.yml


```
     management:
       endpoints:
         web:
           exposure:
             include: health, info, metrics, env
```


3. Secure Endpoints Using Spring Security
     Approach 1: Basic Authentication (Most Common)
       application.yml

     ```
          spring:
            security:
              user:
                name: admin
                password: secret
          
          management:
            endpoints:
              web:
                exposure:
                  include: "*"
            endpoint:
              health:
                show-details: when_authorized
```   

          Now, only authenticated users can access the endpoints

```
          GET /actuator/metrics
          Authorization: Basic base64(admin:secret)
```
     

     Approach 2: Custom Security Configuration

     Use a dedicated **SecurityFilterChain** to restrict access to Actuator endpoints.

     Example:

      ```
          import org.springframework.boot.actuate.autoconfigure.security.servlet.EndpointRequest;
          import org.springframework.context.annotation.Bean;
          import org.springframework.context.annotation.Configuration;
          import org.springframework.security.config.annotation.web.builders.HttpSecurity;
          import org.springframework.security.web.SecurityFilterChain;
          
          @Configuration
          public class ActuatorSecurityConfig {
          
              @Bean
              public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
                  http
                      .authorizeHttpRequests(authorize -> authorize
                          .requestMatchers(EndpointRequest.to("health", "info")).permitAll()
                          .requestMatchers(EndpointRequest.toAnyEndpoint()).hasRole("ADMIN")
                          .anyRequest().authenticated()
                      )
                      .httpBasic(); // or .formLogin()
                  return http.build();
              }
          }
```


          How this works:

               * /actuator/health and /actuator/info are public.
               * Other Actuator endpoints require authentication with role ADMIN.


     Approach 3: Limit Access to Internal Network

          If Actuator is used for monitoring tools (e.g., Prometheus), you can restrict access by IP.


```
           .authorizeHttpRequests(auth -> auth
          .requestMatchers(EndpointRequest.toAnyEndpoint())
          .hasIpAddress("192.168.1.0/24")
```

     Approach 4: Use Separate Management Port (Optional)

          You can serve Actuator endpoints on a different port for better isolation:

          application.yml
         
```
          management:
            server:
              port: 9090
            endpoints:
              web:
                exposure:
                  include: "*"
```
 
<br/>

✅ Best Practices
     Never expose Actuator endpoints publicly **without authentication**.
     Restrict sensitive endpoints like **/actuator/env**, **/actuator/beans**, **/actuator/configprops**.
     Use **HTTPS** for all Actuator traffic.
     **Integrate with your organization’s centralized monitoring and authentication system (e.g., OAuth2, LDAP).**
____________________________________________________________________________________________________________________________ 
### Q) how to create connetion pool in SpringBoot applicaiton.

In Spring Boot, connection pooling is automatically configured when you include a database dependency (like H2, MySQL, PostgreSQL, etc.). However, you can also explicitly configure it for better performance and control.

#### 1. Understanding Connection Pooling

     A connection pool maintains a cache of database connections that can be reused, avoiding the overhead of creating new connections for each request.
     Spring Boot typically uses HikariCP as the default connection pool.

#### 2. Default Behavior

     When you include a JDBC driver dependency (e.g., spring-boot-starter-data-jpa or spring-boot-starter-jdbc),
     Spring Boot automatically sets up HikariCP.

     Example:

```
          <!-- pom.xml -->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-data-jpa</artifactId>
          </dependency>
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-j</artifactId>
          </dependency>
```
     

#### 3. Basic Configuration (application.properties)

```
          spring.datasource.url=jdbc:mysql://localhost:3306/mydb
          spring.datasource.username=root
          spring.datasource.password=secret
          spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
          
          # HikariCP specific settings
          spring.datasource.hikari.maximum-pool-size=10
          spring.datasource.hikari.minimum-idle=5
          spring.datasource.hikari.idle-timeout=30000
          spring.datasource.hikari.max-lifetime=1800000
          spring.datasource.hikari.connection-timeout=20000
          spring.datasource.hikari.pool-name=MyHikariCP
```


#### 4. Using application.yml (alternative)

```
     spring:
       datasource:
         url: jdbc:mysql://localhost:3306/mydb
         username: root
         password: secret
         driver-class-name: com.mysql.cj.jdbc.Driver
         hikari:
           maximum-pool-size: 10
           minimum-idle: 5
           idle-timeout: 30000
           max-lifetime: 1800000
           connection-timeout: 20000
           pool-name: MyHikariCP
```


#### 5. Custom Configuration via Java Code (Optional)
          If you want to programmatically configure a pool:

```
               import com.zaxxer.hikari.HikariConfig;
               import com.zaxxer.hikari.HikariDataSource;
               import org.springframework.context.annotation.Bean;
               import org.springframework.context.annotation.Configuration;
               
               import javax.sql.DataSource;
               
               @Configuration
               public class DataSourceConfig {
               
                   @Bean
                   public DataSource dataSource() {
                       HikariConfig config = new HikariConfig();
                       config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
                       config.setUsername("root");
                       config.setPassword("secret");
                       config.setMaximumPoolSize(10);
                       config.setMinimumIdle(5);
                       config.setPoolName("CustomHikariPool");
                       return new HikariDataSource(config);
                   }
               }
```


#### 6. Monitoring the Connection Pool

          You can monitor connection pool metrics through:

               1. Spring Boot Actuator (/actuator/metrics/hikaricp.connections.*)
               2. JMX (Java Management Extensions)
               3. Custom logging
          
#### 7. Using Other Connection Pools (Optional)

     If you don’t want to use HikariCP, you can switch to:

          1. Apache Commons DBCP2
          2. Tomcat JDBC Pool


<br/>

✅ Summary </br>

| Feature             | Default Pool                          | Configurable via               |
| ------------------- | ------------------------------------- | ------------------------------ |
| Pool Implementation | HikariCP                              | application.properties / @Bean |
| Auto-configured     | Yes                                   | When JDBC driver present       |
| Key Properties      | maxPoolSize, idleTimeout, maxLifetime | spring.datasource.hikari.*     |

_____________________________________________________________________________________________________________________________
### Q) What strategies we use to optimize Spring boot application.

Here are some key strategies to optimize a Spring Boot application for better performance, scalability, and resource efficiency:

#### 01. Optimize Application Startup

     * **Lazy Initialization:** Enable lazy bean loading (spring.main.lazy-initialization=true) to reduce startup time.
     * **Exclude Unused Auto-configurations**: Use @SpringBootApplication(exclude = {...}) or spring.autoconfigure.exclude to skip unnecessary modules.
     * **Profile-based Configuration**: Load only required beans/configurations using @Profile
     
#### 02. Tune JVM and Memory

     * Configure JVM heap size and garbage collector (GC) for your environment (-Xms, -Xmx, -XX:+UseG1GC, etc.).
     * Use tools like VisualVM, JConsole, or Micrometer metrics to monitor memory usage
          
#### 03. Optimize Database Access

    * Use connection pooling (HikariCP — default in Spring Boot).
    * Optimize JPA/Hibernate:
         * Enable batch fetching and lazy loading.
         * **Avoid N+1** query problems using **@EntityGraph** or JOIN FETCH.
         * Use DTO projections for read-heavy queries
     * Indexing and query tuning: 
          Ensure DB queries are indexed properly and optimized.
     
#### 04. Use Efficient Caching

     * Use Spring Cache abstraction: Cache frequently accessed data (Redis, Ehcache, etc.).
     * Avoid unnecessary recomputation: Cache expensive calculations or DB fetches.
     
#### 05. Optimize REST APIs

     * Enable HTTP/2: Reduces latency for multiple requests.
     * Use content compression: Enable GZIP compression (server.compression.enabled=true).
     * Limit request payload: Validate input and avoid loading large unnecessary data.
     
#### 06. Reduce I/O and Logging Overhead

     I/O operations include disk reads/writes, network calls, database queries, and file access. 
     These are typically slower than in-memory operations. Optimization strategies:

#### 07. Profile and Monitor Regularly
          
     * Spring Boot Actuator: Monitor metrics, health, and traces.
     * Profiling tools: Use VisualVM, YourKit, or JFR to identify bottlenecks.
     * Log efficiently: Avoid excessive logging in production; use appropriate log levels.
          
#### 08. Optimize Deployment

    * Spring Boot layered jars: Use layered jars to speed up container builds.
    * Remove unused dependencies: Reduce application size and startup time.
    * Ahead-of-Time (AOT) compilation: With GraalVM native images for fast startup


#### 09. Implement Asynchronous and Non-blocking Processing 

    * @Async methods: For tasks that don’t need to block the main thread.
    * Messaging queues: Offload heavy processing to Kafka, RabbitMQ, or similar.    
    
      
#### 10. Use Proper Data Serialization


     * Messaging (Kafka, RabbitMQ): Use Protobuf or Avro for better performance than JSON.
     * Combine serialization with compression (GZIP) for large payloads sent over network.
     * Don’t serialize heavy objects like database connections, file handles, or large in-memory caches.
_____________________________________________________________________________________________________________________________
### Q) What are the best practices for managing transactions in a Spring Boot application?

Spring Boot leverages Spring’s powerful transaction management framework, which can be declarative or programmatic.
Following these practices ensures consistency, performance, and maintainability.

#### 01. Prefer Declarative Transaction Management


     * Use <b>@Transactional</b> annotations on service layer methods rather than managing transactions manually.
     * Advantages
          * Less boilerplate code.
          * Easier to maintain and test.
          * Clear separation of business logic from transaction handling.

```
          @Service
          public class UserService {
          
              @Transactional
              public void createUser(User user) {
                  userRepository.save(user);
                  // Additional logic
              }
          }
```


#### 02. Apply Transactions at the Service Layer

     * Avoid annotating repository or DAO methods directly.
     * Service layer ensures that all business logic in a single operation is atomic.
     * Keeps transaction boundaries clear and consistent.


#### 03. Use Proper Propagation Settings

     * Understand and use transaction propagation wisely:
          * REQUIRED (default) – join existing transaction or create new one.
          * REQUIRES_NEW – always start a new transaction, suspending any existing one.
          * MANDATORY – must run within an existing transaction


```
          @Transactional(propagation = Propagation.REQUIRES_NEW)
               public void auditAction(Audit audit) {
               auditRepository.save(audit);
          }
```
     


#### 04. Set Appropriate Isolation Levels

     * Prevent data anomalies by choosing the correct isolation level:
          * READ_COMMITTED – default, prevents dirty reads.
          * REPEATABLE_READ – prevents non-repeatable reads.
          * SERIALIZABLE – strictest, avoids phantom reads but reduces concurrency.
          
     * Don’t use overly strict isolation unless required—it impacts performance.

```
          @Transactional(isolation = Isolation.READ_COMMITTED)
               public void updateAccountBalance(Account account) {
                   // update logic
          }
```

               

#### 05. Handle Exceptions Correctly


     * Only unchecked exceptions (RuntimeException) trigger automatic rollback by default.
     * For checked exceptions, explicitly configure rollback:


```
@Transactional(rollbackFor = Exception.class)
               public void processPayment(Payment payment) throws PaymentException {
                   // business logic
}
```
          
        
#### 06. Keep Transactions Short

     * Long-running transactions can lock resources and degrade performance.
     * Break complex operations into smaller, manageable units when possible.


#### 07. Avoid Transactional Annotations on Private Methods

     * Spring’s proxy-based AOP doesn’t intercept calls to private methods, so @Transactional won’t work there.
     * Always annotate public methods that are called from outside the bean.

#### 08. Consider Read-Only Transactions

     * For methods that only read data, mark them as readOnly = true.
     * Optimizes database access and avoids unnecessary locks.


```
@Transactional(readOnly = true)
          public List<User> getAllUsers() {
              return userRepository.findAll();
          }
```
     


#### 09. Integrate with Proper Database Connection Pooling

     * Use connection pools (HikariCP, default in Spring Boot) to efficiently manage transactional connections.
     * Ensure the transaction manager is properly configured to work with the datasource.

#### 10. Use Programmatic Transactions Only When Necessary

     * Use TransactionTemplate or PlatformTransactionManager for fine-grained control when needed.
     * Useful in special cases like multiple datasources or conditional rollback.

```
     transactionTemplate.execute(status -> {
              // transactional code
              return result;
          });
```

<br/>


✅ Summary


* Prefer declarative transactions (@Transactional) at service layer.
* Use proper propagation, isolation, and rollback settings.
* Keep transactions short and read-only where possible.
* Avoid transactions on private methods.
* Integrate with a connection pool for efficiency.
  
_____________________________________________________________________________________________________________________________

### Q) How do you approach testing in springboot application ?

Testing in a Spring Boot application is a critical part of ensuring the application works correctly and reliably.
The approach generally involves a combination of unit testing,
integration testing, and end-to-end testing. Here’s a structured way to approach it:

#### 01. Define the Scope and Layers

Spring Boot applications typically have multiple layers:
     1. Controller layer: Handles HTTP requests.
     2. Service layer: Contains business logic.
     3. Repository layer: Handles data persistence.
     4. Configuration/Integration layer: External services, messaging, or APIs.

Your testing approach should target each layer appropriately.

#### 02. Unit Testing.
1. Purpose: Test individual components in isolation.
2. Tools: JUnit 5, Mockito, AssertJ.
3. Best Practices:
     * Test service classes without depending on the database.
     * Mock dependencies like repositories or external APIs using @Mock or @MockBean.
     * Focus on small, deterministic tests.

Example:
          ```
               @ExtendWith(MockitoExtension.class)
               class UserServiceTest {

              @Mock
              private UserRepository userRepository;
          
              @InjectMocks
              private UserService userService;
          
              @Test
              void testGetUserById() {
                  User user = new User(1L, "John");
                  when(userRepository.findById(1L)).thenReturn(Optional.of(user));
          
                  User result = userService.getUserById(1L);
          
                  assertEquals("John", result.getName());
              }
          }
```

#### 03. Integration Testing
1. Purpose: Test the interaction between components and with the real database or external systems.
2. Tools: @SpringBootTest, Testcontainers, H2 in-memory database.
3. **Best Practices:**
    * Use @SpringBootTest with webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT for full context loading
    * Use @DataJpaTest for repository testing.
    * Use Testcontainers for integration with external databases or services.
     
Example
     
```
     @SpringBootTest
     @AutoConfigureMockMvc
     class UserControllerIntegrationTest {
     
     @Autowired
     private MockMvc mockMvc;
     
    @Test
    void testGetUserEndpoint() throws Exception {
        mockMvc.perform(get("/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("John"));
     }
}
```
   
#### 04. Controller / Web Layer Testing
1. Purpose: Verify request handling, validation, and response formatting.
2. Tools: MockMvc or WebTestClient (for reactive apps).
3. Best Practices:
     * Test error responses, validation, and edge cases.
     * Use @WebMvcTest to test only the controller layer.          
               
#### 05. End-to-End Testing
1. Purpose: Test the application as a whole in a realistic environment.
2. Tools: Selenium, RestAssured, or Postman.
3. Best Practices:
     * Run against a staging environment or with Docker Compose.
     * Focus on critical user journeys
  
#### 06. Test Data Management
   1. Use in-memory databases (H2) for unit/integration tests.
   2. Seed data for predictable results
   3. Clean up after tests to maintain isolation      

#### 07. Testing Aspects Specific to Spring Boot
1. Configuration testing: @TestConfiguration and property overrides.
2. Profiles: Use @ActiveProfiles("test") for test-specific configurations.
3. Transactional tests: @Transactional ensures rollback after each test.
4. Mocking external services: @MockBean or WireMock.

#### 08. Continuous Integration
1. Run tests automatically on every commit or pull request.
2. Use code coverage tools (JaCoCo) to ensure adequate testing.
3. Keep unit tests fast; integration tests can be slower but should also run reliably.

✅ Summary:

Approach testing in Spring Boot by layering your tests (unit → integration → end-to-end), 
use Spring-specific testing annotations to simplify setup, and ensure repeatable, isolated, 
and fast tests wherever possible.

_____________________________________________________________________________________________________________________________

### Q) How does spring boot simplify data access layer.

Spring Boot simplifies the data access layer by building on top of Spring Data, JPA, JDBC, and related technologies — removing boilerplate configuration and enabling rapid development. Here's how it achieves that:

🔹 1. Auto-Configuration
     * Spring Boot automatically configures the necessary data source, JPA provider (like Hibernate), 
          and transaction manager based on the dependencies on your classpath.
     * For example, if you add spring-boot-starter-data-jpa, Spring Boot auto-configures:
          A DataSource bean (based on application.properties)
          An EntityManagerFactory
          A PlatformTransactionManager
     Result: No need for XML or manual bean definitions.     
          
🔹 2. Spring Data Repositories
     * Spring Data JPA provides Repository interfaces (e.g., CrudRepository, JpaRepository, PagingAndSortingRepository) 
          that offer built-in CRUD and query operations.
          
     * You just define an interface:

           ``public interface EmployeeRepository extends JpaRepository<Employee, Long> {
            List<Employee> findByDepartment(String department);
          }
          ``
          Spring automatically generates the implementation at runtime. 
          Result: No boilerplate DAO implementations.
          
🔹 3. Convention over Configuration
     * By following naming conventions (like findByName, findByEmail), Spring Data derives queries automatically.
     * Developers can focus on the domain logic rather than SQL or query handling.
🔹 4. Integrated Transaction Management
     * The @Transactional annotation works seamlessly with Spring Boot’s auto-configured transaction manager.
     * This ensures declarative transaction boundaries without manual transaction handling.
     
🔹 5. Embedded Database Support
     * For rapid prototyping, Spring Boot supports embedded databases (H2, HSQLDB, Derby) without manual setup.
     * Automatically creates schemas and initializes data using schema.sql or data.sql.     

🔹 6. Database Configuration via Properties
     * Simple configuration in application.properties or application.yml:
     
     ``spring.datasource.url=jdbc:mysql://localhost:3306/mydb
          spring.datasource.username=root
          spring.datasource.password=secret
          spring.jpa.hibernate.ddl-auto=update
          ``
          
     No need for XML or Java-based bean setup.
     
🔹 7. Integration with ORM and SQL Libraries
     * Supports JPA, JDBC, R2DBC, MongoDB, Cassandra, Redis, etc. through specific starters.
     * You can switch databases by changing dependencies and minor configuration.
     
🔹 8. Simplified Exception Translation
     * Spring’s @Repository annotation triggers DataAccessException translation, converting vendor-specific exceptions into consistent, unchecked exceptions.

✅ In short:
Spring Boot simplifies the data access layer by automating configuration, eliminating boilerplate, integrating transaction management,
and providing repository abstractions — letting you focus purely on business logic rather than infrastructure code.

________________________________________________________________________________________________________________________________________
### Q) Explain the concept and purpose of conditional annotations in Spring Boot ?

**_Concept of Conditional Annotations in Spring Boot:_**
Conditional annotations in Spring Boot are used to control when a particular bean or configuration should be loaded into the Spring application context. They allow developers to enable or disable components automatically based on certain conditions, such as the presence of a class, property value, or bean.

Spring Boot’s auto-configuration mechanism relies heavily on these conditional annotations to configure components only when certain prerequisites are met—ensuring that the application context remains lightweight and flexible.


_**Common Conditional Annotations**_
     1. @ConditionalOnProperty
     2. @ConditionalOnClass / @ConditionalOnMissingClass
     3. @ConditionalOnBean / @ConditionalOnMissingBean
     4. @ConditionalOnExpression

_**Purpose of Conditional Annotations**_ 
     **Enable Auto-Configuration:**
          * They allow Spring Boot to dynamically configure components only when needed, supporting the
               “convention over configuration” principle.
               
               ``@Configuration
                    @ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
                    public class FeatureConfig { ... }
                    ``
                    
     **Improve Flexibility:**
          * Developers can define optional features or modules that load conditionally—helpful for modular applications.

          ``@Configuration
          @ConditionalOnClass(name = "com.example.ExternalLib")
          public class ExternalLibConfig { ... }
``

     **Reduce Configuration Conflicts:**
          * Prevents duplicate or conflicting bean definitions by activating configurations only under the right conditions.

          ``@Service
          @ConditionalOnMissingBean(MyService.class)
          public class DefaultMyService implements MyService { ... }
          ``
          
     **Enhance Performance:**
          * Loads only necessary beans, reducing startup time and memory usage.

          Example:

          
``@Configuration
@ConditionalOnExpression("${cache.enabled}==true")
public class CacheConfig { ... }
``

In summary:
     Conditional annotations in Spring Boot make configurations context-aware and adaptive, supporting modular, flexible, and efficient application setup.

_______________________________________________________________________
### Q) explain the role of @EnableAutoConfiguration annotations in a spring boot application. how does Springboot achivev auto-configuration internally ?

The @EnableAutoConfiguration annotation is one of the core features of Spring Boot. It plays a central role in enabling the auto-configuration mechanism, which allows Spring Boot to automatically configure your application based on the dependencies present in the classpath and a few other settings.

🧩 Role of @EnableAutoConfiguration
     @EnableAutoConfiguration tells Spring Boot to automatically configure your application context
     1) It scans the classpath for commonly used dependencies (like Spring MVC, Data JPA, Security, etc.).
     2)Then, it automatically creates and registers beans required for those technologies, so you don’t have to write explicit configuration code.

For example:
     1. If spring-boot-starter-web is on the classpath, it auto-configures an embedded Tomcat server and sets up Spring MVC.
     2. If spring-boot-starter-data-jpa is present, it auto-configures a DataSource, EntityManagerFactory, and TransactionManager.

Typically, you don’t use @EnableAutoConfiguration directly because it is included in:
     ``@SpringBootApplication
     ``
     which is equivalent to:
     ``@Configuration
       @ComponentScan
       @EnableAutoConfiguration
     ``
 ⚙️ How Spring Boot Achieves Auto-Configuration Internally
      Internally, auto-configuration is achieved through several key steps and mechanisms:
          
     1. Spring Factories / AutoConfiguration Imports
          Spring Boot looks for a file named:
           ``META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
``
or in older versions:

``META-INF/spring.factories
``
Sample config

``org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
``

Each listed class is an @Configuration class that defines beans conditionally.

     **2. Conditional Annotations**

     Auto-configuration classes use conditional annotations like:

     * @ConditionalOnClass – activates config only if a specific class is on the classpath
     * @ConditionalOnMissingBean – configures a bean only if it’s not already defined by the user.
     * @ConditionalOnProperty – activates config based on a specific property value in application.properties.

     ``@Configuration
@ConditionalOnClass(DataSource.class)
public class DataSourceAutoConfiguration {
    // Defines a DataSource bean automatically
}
``

These conditions make auto-configuration smart — it only applies when it makes sense.


**3. AutoConfigurationImportSelector**
     When Spring processes @EnableAutoConfiguration, it delegates to the class:

     ``org.springframework.boot.autoconfigure.AutoConfigurationImportSelector
``

This selector reads the auto-configuration entries from the files above and imports them into the application context dynamically.

     **4. Configuration Evaluation Phase**
          During the Spring context startup:
               * The AutoConfigurationImportSelector evaluates each auto-config class.
               * It checks all conditions.
               * Only the matching configurations are loaded.
               You can inspect which configurations were applied or excluded using:
               
``--debug
``
This shows the auto-configuration report in the console.

_______________________________________________________________________

### Q) how can we secure the actuator endpoints ?

     In Spring Boot, Actuator endpoints provide valuable operational information — such as health, metrics, and environment details — which makes securing them crucial to prevent unauthorized access.

     1. Restrict Exposure of Endpoints
           ``management:
  endpoints:
    web:
      exposure:
        include: health, info
``

     2. Use Spring Security for Authentication & Authorization

     ``import org.springframework.boot.actuate.autoconfigure.security.servlet.EndpointRequest;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ActuatorSecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(requests -> requests
                .requestMatchers(EndpointRequest.to("health", "info")).permitAll()
                .requestMatchers(EndpointRequest.toAnyEndpoint()).hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(); // or .formLogin() if needed
        return http.build();
    }
}
``

     3. Customize the Actuator Base Path

     You can change the default /actuator path to make endpoints less predictable:

     ``management:
  endpoints:
    web:
      base-path: /manage
``

     4. Secure Actuator Over HTTPS
     Always use HTTPS for sensitive actuator endpoints to ensure encrypted communication.

     ``server:
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: secret
    key-store-type: PKCS12
``

5. Limit Network Access
     You can restrict actuator endpoints to be accessible only from specific IPs or internal networks using a reverse proxy or firewall rules.

6. Use Management Port Separation
 
   ``management:
  server:
    port: 9001
``

7. Hide Sensitive Details

        ``management:
  endpoint:
    health:
      show-details: when_authorized
``


| Practice                           | Description                         |
| ---------------------------------- | ----------------------------------- |
| **Expose only required endpoints** | Avoid `*` exposure                  |
| **Use Spring Security**            | Protect with roles & authentication |
| **Use HTTPS**                      | Encrypt communication               |
| **Limit access by network**        | Allow only trusted hosts            |
| **Separate management port**       | Isolate operational endpoints       |



   example

   🧩 1. YAML Configuration (application.yml)

        ``server:
  port: 8080

management:
  server:
    port: 9001              # Run actuator endpoints on a separate management port
  endpoints:
    web:
      base-path: /manage    # Custom base path instead of /actuator
      exposure:
        include: health, info, metrics, prometheus  # Expose only selected endpoints
  endpoint:
    health:
      show-details: when_authorized   # Show detailed health info only to authorized users
  security:
    enabled: true

spring:
  security:
    user:
      name: admin           # Default admin username
      password: Admin@123   # Strong password (should be encrypted in production)
      roles: ADMIN
``

🛡️ 2. Java Security Configuration (ActuatorSecurityConfig.java)

``
package com.example.config;

import org.springframework.boot.actuate.autoconfigure.security.servlet.EndpointRequest;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class ActuatorSecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                // Allow health and info without authentication
                .requestMatchers(EndpointRequest.to("health", "info")).permitAll()
                // Restrict all other actuator endpoints to ADMIN role
                .requestMatchers(EndpointRequest.toAnyEndpoint()).hasRole("ADMIN")
                // Require authentication for other app endpoints if needed
                .anyRequest().authenticated()
            )
            .httpBasic()   // Use HTTP Basic Auth for simplicity
            .and()
            .csrf().disable();  // Disable CSRF for actuator API access (safe if using HTTPS)

        return http.build();
    }
}
``
_____________________________________________________________________________
### Q) What strategies would you use to optimize the performance of a spring boot application ? 

     1. Optimize Application Startup and Memory Usage
          * **Lazy Initialization**: Enable spring.main.lazy-initialization=true to delay bean creation until required.
          * **Remove Unused Auto-Configuration**: Use @SpringBootApplication(exclude = { ... }) to disable unused components.
          * **Profile-Based Configuration**: Define environment-specific configurations using @Profile (e.g., dev, prod) to load only necessary beans.
          
     2. Improve Database Performance
          * **Connection Pooling**: Use efficient pools like HikariCP (default in Spring Boot). 
               Tune properties such as maximumPoolSize, connectionTimeout, etc.
          * **Batch Operations**: Use batch inserts/updates for large datasets (spring.jpa.properties.hibernate.jdbc.batch_size).
          * **Pagination**: Use pagination and projections for large data queries instead of fetching all records.
          * **Caching**: Leverage Hibernate 2nd-level cache and query cache with providers like Ehcache or Redis
          * **Indexing**: Add appropriate DB indexes to reduce query execution time.

     3. Optimize Caching Layer
          * Use Spring Cache (@Cacheable, @CacheEvict, @CachePut) to cache frequently accessed data.
          * Choose appropriate caching backends like Redis, Caffeine, or Ehcache
          * Implement cache invalidation policies carefully to avoid stale data.
     4. Optimize I/O and Logging
          * Asynchronous Logging: Use Log4j2 async appenders to reduce logging overhead.
          * Limit Log Levels in Production: Set log level to INFO or WARN to reduce disk I/O.
          * Non-blocking I/O: Use Spring WebFlux for high-concurrency, non-blocking workloads.
    5. Reduce Network Overhead    
         * Connection Reuse: Use persistent HTTP connections or connection pooling for REST calls.
         * Compression: Enable GZIP compression in responses using server.compression.enabled=true
         * Pagination and Filtering: Return only required data via pagination and selective field projection.
    6. Tune JVM and GC Settings     
         * Adjust heap size and garbage collection (GC) parameters using flags like:
         
         ``-Xms512m -Xmx1024m -XX:+UseG1GC
          ``
         * Monitor GC activity using tools like VisualVM, JConsole, or Prometheus + Grafana.

     7. Optimize Thread Management     
          * Tune thread pools for web requests and async tasks:
     
          ``spring.task.execution.pool.core-size: 10
            spring.task.execution.pool.max-size: 50
``
          * Use @Async wisely to parallelize I/O-bound tasks.
          
     8. Profile and Monitor the Application
          * Use Spring Boot Actuator to expose metrics (memory, thread, DB connections).
          * Integrate with Micrometer and monitoring tools like Prometheus, Grafana, or New Relic.
          * Profile performance with JProfiler, YourKit, or VisualVM to identify slow methods or memory leaks.
          
     9. Use Build and Deployment Optimizations
          * Layered JARs: Use Spring Boot’s layered JAR feature for faster Docker builds.
          * Native Images: Use GraalVM native-image for ultra-fast startup and reduced memory usage.
          * CDN and Static Content Optimization: Serve static resources via CDN.

     10. Architecture-Level Optimization      
          * Break monoliths into microservices for scalability.
          * Use message queues (Kafka, RabbitMQ) for asynchronous processing.
          * Apply circuit breakers and rate limiters (e.g., Resilience4j) to handle load gracefully.
 ____________________________________________________________________________

 ### Q) how can we handle multiple beans of the same type ?

      1. Using @Primary
      2. Using @Qualifier
      3. Using @Resource (from JSR-250)  // Not @Repository

____________________________________________________________________________

 ### Q) what are some best practices for managing transactions in Spring Boot application ? 
 
      1. Use Declarative Transaction Management (@Transactional)
           * Prefer declarative over programmatic transaction managemen
           * Apply @Transactional at the service layer, not at the controller or repository level.

           ``@Service
               public class OrderService {
               
                   @Transactional
                   public void placeOrder(Order order) {
                       // business logic: save order, update inventory, process payment
                   }
               }
``     

     2. Keep Transactions Short-Lived
          * Minimize the duration of a transaction — avoid holding database locks for long.
          * Don’t perform slow operations (like network calls or file I/O) inside a transaction.
          
          Example (❌ Bad):

          ``@Transactional
               public void updateOrder() {
                   callExternalAPI();  // Slow call
                   orderRepository.save(order);
               }
``
     
     3. Use Proper Transaction Propagation
         *  Control how transactions behave across multiple service calls using propagation levels.
         

          | Propagation Type     | Behavior                                    |
          | -------------------- | ------------------------------------------- |
          | `REQUIRED` (default) | Joins existing or creates a new transaction |
          | `REQUIRES_NEW`       | Always starts a new transaction             |
          | `NESTED`             | Creates a nested transaction (if supported) |
          | `MANDATORY`          | Requires an existing transaction            |
          | `SUPPORTS`           | Runs within existing transaction if present |
          | `NEVER`              | Fails if a transaction exists               |


     Example:

          ``@Transactional(propagation = Propagation.REQUIRES_NEW)
               public void auditLog() {
               // independent transaction for logging
               }
``

    4. Handle Rollbacks Carefully
         * By default, only unchecked (Runtime) exceptions trigger rollbacks.
         * To rollback for checked exceptions, specify explicitly:

         ``@Transactional(rollbackFor = Exception.class)
          public void processOrder() throws Exception {
              // logic
          }
``

     5. Choose Appropriate Isolation Levels
          * Control how concurrent transactions interact with each other.
          
          
               | Isolation Level    | Prevents                     | Use Case                               |
               | ------------------ | ---------------------------- | -------------------------------------- |
               | `READ_UNCOMMITTED` | —                            | Rarely used                            |
               | `READ_COMMITTED`   | Dirty reads                  | Default in most DBs                    |
               | `REPEATABLE_READ`  | Dirty + non-repeatable reads | Data consistency critical              |
               | `SERIALIZABLE`     | All anomalies                | Highest isolation, slowest performance |

     Example:
     
          ``@Transactional(isolation = Isolation.REPEATABLE_READ)
               public void getAccountDetails() {
                   // ensures repeatable reads
               }
``

   6. Avoid Mixing Transactions and Asynchronous Calls       
        * Asynchronous methods (@Async) run in a different thread, so they don’t share the parent transaction.
        * If async execution is required, handle transaction boundaries separately.

   7. Use Read-Only Transactions for Queries
       * For queries that don’t modify data, use:
       * 
              ``@Transactional(readOnly = true)
                    public List<Customer> findAll() {
                        return customerRepository.findAll();
                    }
`` 

✅ Optimizes performance by disabling dirty checking in Hibernate.

   8. Handle Nested Transactions Properly
      * Use Propagation.NESTED carefully — it relies on DB savepoints.
      * If your DB doesn’t support savepoints, use REQUIRES_NEW instead.     

  9. Monitor and Debug Transactions
       * Enable SQL logging and transaction traces for debugging:

               ``spring.jpa.show-sql=true
                 logging.level.org.springframework.transaction=DEBUG
``

   Use tools like Actuator, Micrometer, or p6spy to track transaction metrics.

   10. Consistency in Distributed Systems
       * For microservices, use saga patterns, event-driven transactions, or outbox patterns instead of distributed transactions.
       * Tools like Debezium, Kafka, or Axon Framework can help ensure data consistency.


       ✅ Summary

          | Practice                              | Key Benefit                           |
          | ------------------------------------- | ------------------------------------- |
          | Use `@Transactional` at service layer | Clean and maintainable code           |
          | Keep transactions short               | Avoid locking and contention          |
          | Use correct propagation and isolation | Ensure correct transactional behavior |
          | Explicit rollback rules               | Control rollback precisely            |
          | Use read-only for queries             | Better performance                    |
          | Monitor transactions                  | Easier debugging and optimization     |

   
____________________________________________________________________________
 ### Q) Discuss the use of @SpringBootTest and @MockBean annotations ?

      @SpringBootTest and @MockBean are core testing annotations in Spring Boot, 
           commonly used together to create integration tests that load the Spring context 
           while isolating certain dependencies.

      1. @SpringBootTest
          Purpose: Used to bootstrap the entire Spring ApplicationContext for integration testing
               ✅ Key Features:
                    * Loads the full Spring Boot context, similar to how the application runs in production.
                    * Enables testing of multiple layers — controller, service, and repository.    
                    * Automatically configures environment (e.g., application.properties).
                    * Can start an embedded web server (when testing web apps).
                   
              Common Usage:

                    ``@SpringBootTest
                         class UserServiceIntegrationTest {
                         
                             @Autowired
                             private UserService userService;
                         
                             @Test
                             void testCreateUser() {
                                 User user = new User("John", "john@example.com");
                                 User savedUser = userService.save(user);
                                 assertNotNull(savedUser.getId());
                             }
                         }
                         ``

          ⚙️ Configuration Options:

                    ``@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
                    ``
                         * MOCK: Default. Loads a mock servlet environment without a server.
                         * RANDOM_PORT: Starts a real web server on a random port.
                         * DEFINED_PORT: Uses the port from application.properties.
                         * NONE: No web environment at all.    
                         
                    When to Use:
                         * When you want to test the full application flow (e.g., service → repository → DB).
                         * For verifying configurations, component scanning, or application startup.
                        
      
      2. @MockBean
                    Purpose: Used to mock a specific Spring bean within the application context — useful for isolating the component under test.

                    ✅ Key Features:

                          * Replaces a real bean in the Spring context with a Mockito mock.
                          * Allows you to focus testing on one layer (e.g., testing a controller while mocking the service layer).
                          * Works seamlessly with @SpringBootTest, @WebMvcTest, and other test slices.
                         
                    Common Usage:
                    
                         ``@SpringBootTest
                              class UserControllerTest {
                              
                                  @Autowired
                                  private MockMvc mockMvc;
                              
                                  @MockBean
                                  private UserService userService; // Mocked dependency
                              
                                  @Test
                                  void testGetUser() throws Exception {
                                      when(userService.getUserById(1L)).thenReturn(new User("John", "john@example.com"));
                              
                                      mockMvc.perform(get("/users/1"))
                                             .andExpect(status().isOk())
                                             .andExpect(jsonPath("$.name").value("John"));
                                  }
                              }
                              ``

               When to use
                     * To avoid calling external systems (databases, APIs).
                     * When you want to test a controller or service independently.
                     * To provide controlled behavior for dependencies
                   

       3. Combined Usage Example

                 ``@SpringBootTest
                    class OrderServiceTest {
                    
                        @Autowired
                        private OrderService orderService;
                    
                        @MockBean
                        private PaymentGateway paymentGateway; // External dependency mocked
                    
                        @Test
                        void testPlaceOrder() {
                            when(paymentGateway.charge(any())).thenReturn("TXN123");
                    
                            String transactionId = orderService.placeOrder(new Order());
                            assertEquals("TXN123", transactionId);
                        }
                    }
                    
                 ``
          Here:
                * @SpringBootTest loads the full Spring context.
                * @MockBean replaces the actual PaymentGateway bean, preventing real API calls.
     
          Summary Table
          
               | Annotation        | Purpose                    | Loads Context      | Uses Real Beans | Mock Integration |
               | ----------------- | -------------------------- | ------------------ | --------------- | ---------------- |
               | `@SpringBootTest` | Full integration test      | ✅ Yes              | ✅ Yes           | ❌ No             |
               | `@MockBean`       | Mock specific dependencies | Depends on context | ❌ No            | ✅ Yes            |
          
               
      Best Practice Tips
          * Use @SpringBootTest sparingly — it’s heavy; for lighter tests, prefer @WebMvcTest, @DataJpaTest, etc
          * Use @MockBean to isolate and control dependencies in integration tests.
          * Combine both when you want realistic integration tests without external side effects.
      

____________________________________________________________________________
 ### Q) What advantage does YAML offer over properties files in SpringBoot ? are there limitations when using YAML FOR configuration ?
      
      ✅ Advantages of YAML over .properties files
            1. Hierarchical Structure (Better Readability & Organization)
            2. Support for Lists and Arrays
            3. Improved Maintainability
            4. Cleaner Multi-Profile Configuration
            5. More Readable and Less Redundant
            
     ⚠️ Limitations / Disadvantages of YAML
          1. Indentation Sensitivity
          2. Difficult Debugging for Large Files
          3. Less Friendly for Simple Configurations
          4. Lacks Explicit Key Flattening
          5. Merging Across Files Can Be Confusing
          
          
     
____________________________________________________________________________
 ### Q) Explain how spring boot profiles work.
      1. Defining Profiles
           You can define profiles using the @Profile annotation:
      2. Activating Profiles
           You can activate profiles in several ways
                application.properties:
                     spring.profiles.active=dev
                 application.yml:
                         spring:
                           profiles:
                             active: dev
                 Command line:
                      java -jar app.jar --spring.profiles.active=prod
                 Environment variable:
                      SPRING_PROFILES_ACTIVE=prod
                  Programmatically:
                  
                       ``SpringApplication app = new SpringApplication(MyApplication.class);
                         app.setAdditionalProfiles("test");
                         app.run(args);
                         ``
                         
     3. Profile-Specific Configuration Files
               Spring Boot automatically looks for profile-specific property files:
                    * application.properties → base configuration (common to all)
                    * application-dev.properties → applies when dev profile is active
                    * application-prod.yml → applies when prod profile is active
                    
                    
      4. Multiple Active Profiles 
           spring.profiles.active=dev,logging

     ⚠️ Best Practices
               * Keep environment-specific credentials out of source code; use environment variables or externalized configuration.
               * Use Spring Cloud Config for managing profiles in distributed microservices.

____________________________________________________________________________
 ### Q) What is aspect-oriented programming in the spring framework ? 
 
      Aspect-Oriented Programming (AOP) in the Spring Framework is a programming paradigm that allows you to separate cross-cutting concerns — logic that is common across multiple parts of an application but not central to the business logic.

      🧩 Concept Overview

           In typical applications, certain functionalities such as:
                Logging
                Transaction management
                Security checks
                Performance monitoring
                Exception handling


      ⚙️ Core AOP Concepts in Spring

                    | Term           | Description                                                                                                 |
                    | -------------- | ----------------------------------------------------------------------------------------------------------- |
                    | **Aspect**     | A module that encapsulates a cross-cutting concern (e.g., logging, transaction).                            |
                    | **Join Point** | A point during program execution, such as a method call or exception throw, where an aspect can be applied. |
                    | **Advice**     | The action taken by an aspect at a particular join point (e.g., before a method executes).                  |
                    | **Pointcut**   | An expression that matches join points; determines where advice should be applied.                          |
                    | **Weaving**    | The process of linking aspects with other application types or objects — done at runtime in Spring.         |


     🔧 Types of Advice in Spring AOP


          | Advice Type         | Description                                                                            |
          | ------------------- | -------------------------------------------------------------------------------------- |
          | **@Before**         | Runs before the method execution.                                                      |
          | **@After**          | Runs after the method completes (regardless of outcome).                               |
          | **@AfterReturning** | Runs only if the method returns successfully.                                          |
          | **@AfterThrowing**  | Runs if the method throws an exception.                                                |
          | **@Around**         | Runs before and after the method execution; allows custom control of method execution. |



     🧠 Example: Logging Aspect

          ``
          @Aspect
          @Component
          public class LoggingAspect {
          
              @Before("execution(* com.example.service.*.*(..))")
              public void logBefore(JoinPoint joinPoint) {
                  System.out.println("Executing method: " + joinPoint.getSignature().getName());
              }
          
              @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", returning = "result")
              public void logAfterReturning(JoinPoint joinPoint, Object result) {
                  System.out.println("Method executed successfully: " + joinPoint.getSignature().getName());
                  System.out.println("Returned: " + result);
              }
          }
``

     Here:

          * The aspect intercepts every method call in the com.example.service package.
          * The @Before and @AfterReturning advices log method activity automatically.

     🧵 How Spring Implements AOP
     
          * Spring AOP is proxy-based — it creates a proxy object that wraps the target bean.
          * When a method on the target is invoked, the proxy intercepts it and applies the aspect logic.
          * It uses JDK dynamic proxies (for interfaces) or CGLIB proxies (for classes without interfaces).

      ✅ Benefits
           * Promotes separation of concerns.
           * Reduces code duplication.
           * Simplifies maintenance and testing.
           * Allows declarative application of behaviors.

      ⚠️ Limitations
           * Works only with Spring-managed beans.
           * Cannot apply aspects to private methods.
           * Runtime proxying has a small performance overhead.
           

____________________________________________________________________________
 ### Q) what is spring cloud and how it is useful for building microservices ?

 🔹 Key Features of Spring Cloud

      1. Centralized Configuration Management
           * Managed through Spring Cloud Config Server. 
           * Allows all microservices to read configuration from a central source (e.g., Git repository).
           * Enables dynamic refresh of configurations without restarting services.
      2. Service Discovery
           * Managed using Eureka, Consul, or Zookeeper.
           * Services can automatically register and discover each other dynamically without hardcoding URLs.
      3. Load Balancing
           * Spring Cloud LoadBalancer or Netflix Ribbon provides client-side load balancing among service instances.
           *      
      4. Circuit Breakers / Fault Tolerance
           * Integrated with Resilience4j or Hystrix to handle service failures gracefully and prevent cascading failures.
      5. API Gateway / Routing
           * Spring Cloud Gateway acts as a single entry point for all microservices.
           * Handles routing, rate limiting, authentication, and monitoring.      
      6. Distributed Tracing and Monitoring
           * Spring Cloud Sleuth and Zipkin provide tracing across multiple services to track requests and diagnose latency issues.
      7. Messaging and Event-Driven Communication
           * Spring Cloud Stream supports asynchronous communication using message brokers like Kafka or RabbitMQ


     🔹 How Spring Cloud Helps in Building Microservices

          | Challenge                                         | Spring Cloud Solution                      |
          | ------------------------------------------------- | ------------------------------------------ |
          | Managing configurations across many services      | **Spring Cloud Config Server**             |
          | Locating and connecting microservices dynamically | **Eureka / Consul Discovery**              |
          | Handling load balancing                           |_ **Spring Cloud LoadBalancer** _           |
          | Preventing cascading failures                     | **Resilience4j / Hystrix Circuit Breaker** |
          | Centralized API routing and security              | **Spring Cloud Gateway**                   |
          | Observability (tracing, metrics, logs)            |_ **Sleuth + Zipkin**   _                   |
          | Messaging and event-driven communication          | **Spring Cloud Stream**                    |

____________________________________________________________________________
 ### Q) How does spring boot make the decision on which server to use ? 
  Dependes on starter included in pom.xml file.
____________________________________________________________________________
 ### Q) How to get the list of all the beans in ur spring boot application ? 

 

``import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;

@Component
public class BeanLister implements CommandLineRunner {

    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public void run(String... args) {
        String[] beanNames = applicationContext.getBeanDefinitionNames();
        System.out.println("Total Beans: " + beanNames.length);
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
    }
}
``


____________________________________________________________________________
 ### Q) Explain concept of spring boot embedded servlet containers.
 
🔹 4. Benefits of Embedded Servlet Containers

✅ Simplified Deployment:
No need for an external server — just run the JAR.

✅ Consistent Environment:
Every deployment includes the same server version, avoiding “works on my machine” issues.

✅ Fast Startup:
Bootstraps quickly without complex server startup scripts.

✅ Microservice Friendly:
Ideal for microservice architectures where each service runs independently.

✅ Custom Configuration:
You can customize port, context path, threads, etc., in application.properties:

____________________________________________________________________________
 ### Q) How does Spring Boot make DI esier compared to traditional Spring ? 

           Spring Boot makes Dependency Injection (DI) much easier and faster to use than traditional Spring by automating
      configuration, reducing boilerplate, and using smart defaults. Let’s break it down clearly:

       ⚙️ 2. Auto-Configuration
           Spring Boot introduces @EnableAutoConfiguration, which automatically configures beans based on classpath dependencies and environment.
For example:
               * If spring-boot-starter-data-jpa is on the classpath, Boot auto-configures EntityManager, DataSource, and TransactionManager beans.
               
        🪄 3. Simplified Configuration via Starters

            * Spring Boot Starters (e.g., spring-boot-starter-web, spring-boot-starter-data-jpa) automatically pull in
             required dependencies and configurations — no need to manage individual library versions or bean definitions
             
        🧰 4. Profiles and Configuration Properties
        
             * Spring Boot supports @ConfigurationProperties and profiles to inject configuration values easily from application.yml or        application.properties, making DI for environment-specific setups straightforward.

       In short:      
       
       Spring Boot removes the need for manual bean declarations, configuration files, and repetitive setup, 
       letting you focus on application logic — not wiring dependencies.         
                          
____________________________________________________________________________
 ### Q) How does spring boot simplify the management of application secrets and sensitive conigurations, especially when deployed in different environments ?
 
     1. Externalized Configuration

          Spring Boot follows the “externalized configuration” principle, which means sensitive
          information (like database passwords, API keys, tokens) is not hard-coded or packaged within the application JAR.

          Configuration can be defined in:
               application.properties or application.yml
               Environment variables
               Command-line arguments
               OS-level system properties
               External config files (outside the JAR/WAR)

         ✅ Benefit: Secrets can vary per environment (e.g., dev vs. prod) without modifying or rebuilding the application.      
          
     2. Spring Profiles
          Profiles (spring.profiles.active) allow environment-specific configurations.
          Example:

               ``# application-dev.yml
db.password: dev-password

# application-prod.yml
db.password: ${DB_PASSWORD}
``

            * In production, you can inject the secret via an environment variable instead of storing it in the file.
            * Spring automatically loads the right profile’s configuration based on the active profile.
               
          ✅ Benefit: Each environment has its own isolated configuration file with minimal risk of leakage.

          
     3. Environment Variables & System Properties
          Spring Boot can directly map environment variables to configuration properties.

          Example:
               ``export SPRING_DATASOURCE_PASSWORD=secret123
``

     In your application.yml:

          `` spring:
  datasource:
    password: ${SPRING_DATASOURCE_PASSWORD}
``
     ✅ Benefit: Secrets can be managed securely at the OS or container level, avoiding plaintext in code repositories.
     
     4. Integration with Secret Management Tools
     
          Spring Boot integrates smoothly with secret managers, enabling secure and centralized secret handling.
          Common integrations:
               * Spring Cloud Config Server: Centralized configuration management, can encrypt/decrypt sensitive values.
               * Spring Cloud Vault: Integration with HashiCorp Vault to fetch secrets securely at runtime.
               * Spring Cloud AWS / Azure Key Vault / GCP Secret Manager: Automatically retrieves secrets from cloud providers’ secret stores.

         ✅ Benefit: Secrets are never stored in the app or config files—fetched securely at runtime.      
     
     5. Encryption Support (Spring Cloud Config)
          Sensitive properties can be encrypted using the Config Server:
          Example:
     
               ``my.secret={cipher}AQBsdn93...
               ``
          Spring decrypts these automatically using a server-side key during startup.
          
      ✅ Benefit: Adds security even if configuration files are exposed.         
                    
     6. Docker & Kubernetes Integration
          When deploying to containers or Kubernetes:
               * Secrets are stored as Docker secrets or Kubernetes Secrets.
               * Spring Boot automatically reads them from mounted files or environment variables.

          ✅ Benefit: Secure by design in cloud-native deployments.
____________________________________________________________________________

 ### Q) Explain spring boot's approach to handle asynchronous operations.

          Spring Boot provides built-in support for asynchronous operations through Spring Framework’s @Async mechanism,
     allowing you to execute methods in a non-blocking, parallel, or background fashion without manually managing threads.

          Here’s how Spring Boot handles asynchronous operations in a clean, declarative way:
          
     🧩 1. Enabling Asynchronous Support
     
          To use asynchronous methods, you must enable async processing by annotating a configuration class with:
          

          ``@EnableAsync
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
``

     ⚙️ 2. Defining Asynchronous Methods

          Mark any method that should run asynchronously with @Async.
          Spring will execute such methods in a separate thread, freeing up the main thread to handle other tasks.

          ``@Service
public class EmailService {

    @Async
    public void sendEmail(String to) {
        // Simulate delay
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println("Email sent to: " + to);
    }
}
``
When you call sendEmail(), it returns immediately — the actual execution happens in a background thread.


     🧵 3. Async Return Types
               An @Async method can:
                         Return void
                         Return Future<T>
                         Return CompletableFuture<T> (recommended for better composition and non-blocking chaining)


                         Example:
                         

                         ``@Async
public CompletableFuture<String> fetchData() {
    String data = callExternalApi();
    return CompletableFuture.completedFuture(data);
}
``

          You can then combine multiple async calls:

          ``CompletableFuture<String> f1 = service.fetchData();
CompletableFuture<String> f2 = service.fetchData();
CompletableFuture.allOf(f1, f2).join();
``

     ⚖️ 4. Thread Pool Configuration
          By default, Spring uses a SimpleAsyncTaskExecutor, which creates new threads for each task
          For production use, you can define a custom thread pool to control concurrency and performance:


          ``@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("AsyncExecutor-");
        executor.initialize();
        return executor;
    }
}
``

          🧰 5. Error Handling
               To handle exceptions in async methods returning void, you can implement AsyncUncaughtExceptionHandler:


                ``@Override
public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return (ex, method, params) ->
        System.err.println("Exception in async method: " + method.getName() + " -> " + ex.getMessage());
}
``    

         ⚡ 6. Integration with Web & Reactive Stack

              * For Servlet-based apps, @Async offloads long-running tasks to background threads.
              * For Reactive apps (Spring WebFlux), you typically use reactive types (Mono, Flux) instead of @Async,
                since the reactive stack already provides non-blocking behavior. 

     In essence:
               Spring Boot simplifies asynchronous programming by abstracting thread management. 
               You just annotate methods with @Async, and Spring handles the rest—creating threads, 
               managing execution, and returning results asynchronously.

____________________________________________________________________________
 ### Q) How can you enable and use asynchrounous method in a spring boot app ? 

      To enable and use asynchronous methods in a Spring Boot application, you use Spring’s @Async support.
This allows a method to run in a separate thread, freeing up the main thread to handle other tasks — improving scalability and responsiveness, especially for I/O-bound operations.

____________________________________________________________________________
 ### Q) Describe how you would secure sensitive data in a Spring Boot application that is accessed by multiple users with different roles ? 

      Securing sensitive data in a Spring Boot application accessed by multiple users with different roles involves implementing a layered approach that combines authentication, authorization, encryption, and secure configuration management.
Here’s a detailed breakdown of how you can achieve this:


     1. Implement Authentication and Authorization
     
          ✅ Authentication
               * Use Spring Security to verify user identities.
               * Integrate with a secure identity provider (e.g., OAuth2, OpenID Connect, or LDAP) or 
                    *  manage users with Spring Security’s UserDetailsService.
               * Use JWT tokens or session-based authentication for stateless APIs.
               
          ✅ Authorization   
               * Define role-based access control (RBAC) using annotations:


               `` @PreAuthorize("hasRole('ADMIN')")
                  public void deleteUser(Long id) { ... }
                  ``


               * Secure endpoints in your SecurityFilterChain:

                ``http
    .authorizeHttpRequests()
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
    .anyRequest().authenticated();
                ``

                               
     2. Protect Sensitive Data in Storage
          * Encrypt sensitive fields (like passwords, tokens, or PII) before persisting to the database.
               * Use JPA converters or Spring Security’s BCryptPasswordEncoder for hashing passwords.
               

                   `` @Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
``

          
     3. Secure Data in Transit
          * Always use HTTPS (TLS) for all communication.
          * Enforce HTTPS by redirecting all HTTP traffic:

          
          ``server:
                 ssl:
                   enabled: true
                   ``

                    
     4. Manage Secrets and Configuration Securely

          * Never hardcode secrets (like API keys, DB credentials, or encryption keys) in code or Git.
          * Use one of the following approaches:
               * Spring Cloud Config + Vault for centralized encrypted secret management.
               * Environment variables or Kubernetes secrets for containerized deployments.
               
               * Example using Vault:


                    `` spring.cloud.vault.enabled=true
spring.cloud.vault.token=<vault-token>
spring.cloud.vault.uri=https://vault-server:8200
``
               
          
     5. Implement Data Access Restrictions
          * Apply method-level or data-level security:
               * Use @PostFilter or @PreFilter to filter sensitive data based on user roles.
               
                    * For example

                    
                         `` @PostFilter("filterObject.owner == authentication.name")
public List<Document> getDocuments() { ... }
``
               
     * Use database-level row filtering or multi-tenant access control where applicable.

                    
     6. Logging and Auditing
     
          * Log security-relevant events (logins, failed access attempts, data modifications).
          * Mask sensitive information in logs (like passwords, credit card numbers).
          * Enable Spring Boot Actuator audit events for tracking access.
          
          
     7. Input Validation and Output Encoding
          * Prevent injection attacks by validating user inputs.
          * Use Spring’s @Valid annotation and Bean Validation.
          * Sanitize or encode output to avoid XSS attacks.

          
     8. Regular Security Updates and Testing
          * Keep dependencies up-to-date using tools like OWASP Dependency Check.
          * Use Spring Boot’s Actuator Security to limit exposure.
          * Perform regular penetration testing and code reviews.


     ✅ Example: Summary Architecture

               | Security Layer         | Implementation                    |
               | ---------------------- | --------------------------------- |
               | **Authentication**     | Spring Security (JWT/OAuth2)      |
               | **Authorization**      | Role-based (`@PreAuthorize`)      |
               | **Data Encryption**    | BCrypt + Jasypt/Spring Vault      |
               | **Transport Security** | HTTPS/TLS                         |
               | **Secret Management**  | Spring Vault / Env Vars           |
               | **Auditing**           | Spring Boot Actuator Audit Events |

     
     

________________________________________________________________________________________________________________________________________

### Q) you are creating an endpoint in a Spring boot application that allows users to upload files. Explain how you would handle the file upload and where        you would store the files.

     🧩 1. Enable File Upload Support

          Spring Boot automatically provides multipart file upload support through MultipartResolver.
          To enable it, ensure that you have the following dependency in your pom.xml (for web apps):
     
     
          `` <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
     </dependency> ``
     
     
          By default, Spring Boot enables 
     
          `` spring.servlet.multipart.enabled=true.
          ``
     
          You can customize upload limits in application.properties:
     
          `` spring.servlet.multipart.max-file-size=10MB
          spring.servlet.multipart.max-request-size=10MB
          ``

     ⚙️ 2. Create the Upload Endpoint
     
          Here’s a simple REST controller to handle file uploads:

          `` @RestController
             @RequestMapping("/api/files")             
             public class FileUploadController {          
             private static final String UPLOAD_DIR = "uploads/";
          
              @PostMapping("/upload")
              public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
                  try {
                      // Ensure directory exists
                      File directory = new File(UPLOAD_DIR);
                      if (!directory.exists()) {
                          directory.mkdirs();
                      }
          
                      // Save file to local directory
                      Path path = Paths.get(UPLOAD_DIR + file.getOriginalFilename());
                      Files.copy(file.getInputStream(), path, StandardCopyOption.REPLACE_EXISTING);
          
                      return ResponseEntity.ok("File uploaded successfully: " + file.getOriginalFilename());
                  } catch (IOException e) {
                      return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                              .body("File upload failed: " + e.getMessage());
                  }
              }
          }
          ``
     
     🗂️ 3. Where to Store the File
          There are several options depending on your use case:
          🏠 Local File System
               Pros: Easy to set up and good for small projects or local development.
               Cons: Not suitable for distributed environments (e.g., multiple app servers).
          Example:
               Files stored under /uploads inside your project directory or an external mount point.
               
          ☁️ Cloud Storage
               For scalability and fault-tolerance, use:
                    * Amazon S3
                    * Google Cloud Storage
                    * Azure Blob Storage
               You would then upload the file directly to the cloud storage bucket instead of saving it locally:
                    

          `` amazonS3.putObject(new PutObjectRequest(bucketName, keyName, file.getInputStream(), metadata));
          ``

     🧠 Database Storage (as BLOB)
          Suitable for small files and when you need strong transactional consistency.
          Store file bytes in a BLOB column, with metadata in the same table.

               Example entity:


               ``
               @Entity
               public class FileEntity {
                   @Id @GeneratedValue
                   private Long id;
                   private String filename;
                   private String contentType;
                   @Lob
                   private byte[] data;
               }
               ``

     🧾 4. Security and Best Practices

               * Validate file type and size before saving (file.getContentType(), file.getSize()).
               * Use random or UUID-based file names to avoid collisions.
               * Sanitize file paths to prevent directory traversal attacks.
               * Consider virus scanning or checksum validation for uploaded files.
               * If using public access, serve files through a secure download endpoint rather than exposing the directory.



               ✅ Summary
               

                    | Aspect       | Option                 | Best for                        |
                    | ------------ | ---------------------- | ------------------------------- |
                    | **Storage**  | Local filesystem       | Simple apps or development      |
                    | **Storage**  | Cloud (S3, GCS, Azure) | Production and scalability      |
                    | **Storage**  | Database (BLOB)        | Small files, transactional apps |
                    | **Security** | Validate & sanitize    | Always                          |


        **AWS S3- 1 Spring Boot file upload endpoint that stores files in AWS S3**

          🚀 1. Add AWS SDK Dependency
               In your pom.xml, include the AWS SDK for S3:


             `` <dependency>
                   <groupId>software.amazon.awssdk</groupId>
                   <artifactId>s3</artifactId>
               </dependency>
             ''

          ⚙️ 2. Configure AWS Credentials

             You can provide AWS credentials in several ways:

                  * Environment variables:

                  
                            `` 
                              AWS_ACCESS_KEY_ID=your_access_key
                              AWS_SECRET_ACCESS_KEY=your_secret_key
                              AWS_REGION=ap-south-1
                              ``

                    * Or in application.properties:   
                         
                         
                         `` cloud.aws.region.static=ap-south-1
                         cloud.aws.s3.bucket-name=my-upload-bucket
                         ``

       🧩 3. Create a Configuration Bean for S3 Client         


            `` import org.springframework.context.annotation.Bean;
               import org.springframework.context.annotation.Configuration;
               import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider;
               import software.amazon.awssdk.regions.Region;
               import software.amazon.awssdk.services.s3.S3Client;
               
               @Configuration
               public class S3Config {
               
                   @Bean
                   public S3Client s3Client() {
                       return S3Client.builder()
                               .region(Region.AP_SOUTH_1)  // or from properties
                               .credentialsProvider(DefaultCredentialsProvider.create())
                               .build();
                   }
               }
``

     📤 4. Create the File Upload Service

          `` import org.springframework.beans.factory.annotation.Value;
               import org.springframework.stereotype.Service;
               import org.springframework.web.multipart.MultipartFile;
               import software.amazon.awssdk.core.sync.RequestBody;
               import software.amazon.awssdk.services.s3.S3Client;
               import software.amazon.awssdk.services.s3.model.PutObjectRequest;
               
               import java.io.IOException;
               import java.util.UUID;
               
               @Service
               public class S3FileUploadService {
               
                   private final S3Client s3Client;
               
                   @Value("${cloud.aws.s3.bucket-name}")
                   private String bucketName;
               
                   public S3FileUploadService(S3Client s3Client) {
                       this.s3Client = s3Client;
                   }
               
                   public String uploadFile(MultipartFile file) throws IOException {
                       String fileName = UUID.randomUUID() + "_" + file.getOriginalFilename();
               
                       PutObjectRequest putObjectRequest = PutObjectRequest.builder()
                               .bucket(bucketName)
                               .key(fileName)
                               .contentType(file.getContentType())
                               .build();
               
                       s3Client.putObject(putObjectRequest, RequestBody.fromBytes(file.getBytes()));
               
                       return fileName;
                   }
               }
               ``

          🌐 5. Create the REST Controller

               ``
                    import org.springframework.http.ResponseEntity;
                    import org.springframework.web.bind.annotation.*;
                    import org.springframework.web.multipart.MultipartFile;

                    @RestController
                    @RequestMapping("/api/files")
                    public class FileUploadController {
                    
                        private final S3FileUploadService s3FileUploadService;
                    
                        public FileUploadController(S3FileUploadService s3FileUploadService) {
                            this.s3FileUploadService = s3FileUploadService;
                        }
                    
                        @PostMapping("/upload")
                        public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
                            try {
                                String fileName = s3FileUploadService.uploadFile(file);
                                String fileUrl = "https://my-upload-bucket.s3.amazonaws.com/" + fileName;
                                return ResponseEntity.ok("File uploaded successfully! URL: " + fileUrl);
                            } catch (Exception e) {
                                return ResponseEntity.internalServerError()
                                        .body("File upload failed: " + e.getMessage());
                            }
                        }
                    }
``

          🧾 6. Security and Best Practices

               ✅ Use IAM Roles instead of static keys when running on AWS.
               ✅ Validate file types and sizes before uploading.
               ✅ Use UUIDs or timestamp-based naming to avoid collisions.
               ✅ Restrict S3 bucket permissions — only allow putObject and getObject where needed.
               ✅ Consider S3 pre-signed URLs for large uploads from the browser directly to S3 (bypassing backend).
               
               
     ✅ Summary


     | Step | Description                       |
     | ---- | --------------------------------- |
     | 1️⃣  | Add AWS SDK dependency            |
     | 2️⃣  | Configure credentials & region    |
     | 3️⃣  | Create `S3Client` bean            |
     | 4️⃣  | Implement upload logic in service |
     | 5️⃣  | Expose REST endpoint              |
     | 6️⃣  | Apply security & validation       |


     **AWS S3 1 S3 End**

    -  **AWS S3 2 S3 Explain how to secure Amazon S3 pre-signed URL uploads using AWS Identity and Access Management (IAM), with an example.**
    
          1️⃣ What Are Pre-Signed URLs?
               A pre-signed URL is a temporary, secure link that allows a client (e.g., a web or mobile app) to upload or download an                object directly to or from Amazon S3, without requiring AWS credentials on the client side.

               _he URL is generated and signed by your backend server using valid AWS credentials._

               This allows direct client-to-S3 uploads, offloading file transfer from your server while keeping control and security centralized.
                   
               
          2️⃣ Why Use IAM for Security
               AWS Identity and Access Management (IAM) is used to:
                    * Control who can generate pre-signed URLs.
                    * Limit what actions can be performed (e.g., only upload, no delete).
                    * Enforce fine-grained permissions on specific S3 buckets or prefixes.
                    
               You use IAM roles and policies to ensure that only authorized backend components 
               (like your Spring Boot app) can create pre-signed URLs, while the public client can only
               use the URL to upload a file.
                    
               
          3️⃣ IAM Policy Example
                    Here’s a secure IAM policy that allows your backend service to generate upload URLs (PUT)
               and optionally read (GET) objects:


                    `` 
                         {
                           "Version": "2012-10-17",
                           "Statement": [
                             {
                               "Effect": "Allow",
                               "Action": ["s3:PutObject", "s3:GetObject"],
                               "Resource": "arn:aws:s3:::my-secure-upload-bucket/*"
                             }
                           ]
                         }
``


               ➡️ Attach this policy to the IAM role of your backend application (e.g., EC2, ECS, or Lambda).
                    Your frontend or clients never get access to AWS credentials directly.
               
                    
               
          4️⃣ Backend Flow (Spring Boot Example)
               Step 1: Backend generates a pre-signed URL
               The backend uses the AWS SDK and IAM credentials to create a temporary signed URL:    


`` 
               @Service
               public class S3PresignedUrlService {
               
                   private final S3Presigner presigner;
                   private final String bucketName = "my-secure-upload-bucket";
               
                   public S3PresignedUrlService(S3Presigner presigner) {
                       this.presigner = presigner;
                   }
               
                   public String generatePresignedUploadUrl(String fileName, String contentType) {
                       String key = "uploads/" + UUID.randomUUID() + "_" + fileName;
               
                       PutObjectRequest putObjectRequest = PutObjectRequest.builder()
                               .bucket(bucketName)
                               .key(key)
                               .contentType(contentType)
                               .build();
               
                       PresignedPutObjectRequest presignedRequest = presigner.presignPutObject(p -> p
                               .putObjectRequest(putObjectRequest)
                               .signatureDuration(Duration.ofMinutes(5))
                       );
               
                       return presignedRequest.url().toString();
                   }
               }
``


               Step 2: Client uses the URL to upload directly to S3
                         Once the backend provides the signed URL, the client can upload the file using HTTP PUT:


`` 
               const file = document.querySelector('#fileInput').files[0];
               const response = await fetch(`/api/files/presigned-upload?fileName=${file.name}&contentType=${file.type}`);
               const uploadUrl = await response.text();
               
               // Upload directly to S3
               await fetch(uploadUrl, {
                 method: 'PUT',
                 headers: { 'Content-Type': file.type },
                 body: file
               });

``
                    
               
          5️⃣ Security Best Practices

          | Control                         | Description                                                                          |
          | ------------------------------- | ------------------------------------------------------------------------------------ |
          | **IAM Roles (not static keys)** | Use IAM roles attached to EC2/ECS/Lambda instead of hardcoded access keys.           |
          | **Scoped Permissions**          | Allow only required actions (`PutObject`, `GetObject`) for a specific bucket/prefix. |
          | **Short Expiration**            | Set signed URL expiry (e.g., 5–10 minutes) to limit misuse.                          |
          | **File Validation**             | Backend should validate file size, type, and user permissions before generating URL. |
          | **CORS Configuration**          | In S3, restrict origins and methods allowed for uploads.                             |
          | **Unique Keys**                 | Use UUIDs or user IDs to avoid filename collisions.                                  |


             Example S3 CORS Policy:

                 `` [
                      {
                        "AllowedHeaders": ["*"],
                        "AllowedMethods": ["PUT", "GET"],
                        "AllowedOrigins": ["https://your-frontend.com"],
                        "ExposeHeaders": ["ETag"]
                      }
                    ]
                    ``
          ✅ Summary

               | Step | Description                                                  |
               | ---- | ------------------------------------------------------------ |
               | 1    | Backend uses IAM role credentials to generate pre-signed URL |
               | 2    | URL allows secure, time-limited PUT to S3                    |
               | 3    | Client uploads directly to S3                                |
               | 4    | IAM and validation ensure access control and data protection |

              🧾 In short:  
                   To secure S3 pre-signed URL uploads, use AWS IAM roles and fine-grained S3 policies so only authorized backend services can generate URLs. The client never gets AWS credentials and can upload only to a specific bucket, file path, and time window defined in the signed request.
                               
       **AWS S3 2 S3 End**                
         
    -**AWS S3 3 S3  Download pre-signed URL**_  

          🛠️ Step 1: Configure AWS S3 in Spring Boot
          
               In application.properties:


               `` aws.access.key.id=YOUR_AWS_ACCESS_KEY
                    aws.secret.access.key=YOUR_AWS_SECRET_KEY
                    aws.s3.bucket.name=my-bucket
                    aws.region=us-west-2
                    jwt.secret.key=your_jwt_secret_key
                    ``

               
               
          🧠 Step 2: Create the S3 Service
          
               This service generates pre-signed URLs for secure, temporary access to S3 files.

               
               ``
               import com.amazonaws.HttpMethod;
                    import com.amazonaws.auth.AWSStaticCredentialsProvider;
                    import com.amazonaws.auth.BasicAWSCredentials;
                    import com.amazonaws.regions.Regions;
                    import com.amazonaws.services.s3.AmazonS3;
                    import com.amazonaws.services.s3.AmazonS3ClientBuilder;
                    import com.amazonaws.services.s3.model.GeneratePresignedUrlRequest;
                    import org.springframework.beans.factory.annotation.Value;
                    import org.springframework.stereotype.Service;
                    
                    import java.net.URL;
                    import java.util.Date;
                    
                    @Service
                    public class S3Service {
                    
                        private final AmazonS3 s3Client;
                        private final String bucketName;
                    
                        public S3Service(@Value("${aws.access.key.id}") String accessKey,
                                         @Value("${aws.secret.access.key}") String secretKey,
                                         @Value("${aws.s3.bucket.name}") String bucketName,
                                         @Value("${aws.region}") String region) {
                    
                            this.bucketName = bucketName;
                    
                            BasicAWSCredentials creds = new BasicAWSCredentials(accessKey, secretKey);
                            this.s3Client = AmazonS3ClientBuilder.standard()
                                    .withCredentials(new AWSStaticCredentialsProvider(creds))
                                    .withRegion(Regions.fromName(region))
                                    .build();
                        }
                    
                        public String generatePresignedDownloadUrl(String fileName) {
                            Date expiration = new Date(System.currentTimeMillis() + 1000 * 60 * 5); // 5 minutes validity
                            GeneratePresignedUrlRequest request = new GeneratePresignedUrlRequest(bucketName, fileName)
                                    .withMethod(HttpMethod.GET)
                                    .withExpiration(expiration);
                    
                            URL url = s3Client.generatePresignedUrl(request);
                            return url.toString();
                        }
                    }
``

               
          🔒 Step 3: Secure the API with JWT

               Here’s a simplified JWT authentication filter (like the upload example):

               
               `` 
                    import io.jsonwebtoken.Claims;
                    import io.jsonwebtoken.Jwts;
                    import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
                    import org.springframework.security.core.context.SecurityContextHolder;
                    import org.springframework.web.filter.OncePerRequestFilter;
                    
                    import javax.servlet.FilterChain;
                    import javax.servlet.ServletException;
                    import javax.servlet.http.HttpServletRequest;
                    import javax.servlet.http.HttpServletResponse;
                    import java.io.IOException;
                    
                    public class JwtAuthenticationFilter extends OncePerRequestFilter {
                    
                        private final String secretKey = "your_jwt_secret_key";
                    
                        @Override
                        protected void doFilterInternal(HttpServletRequest request, 
                                                           HttpServletResponse response, 
                                                           FilterChain filterChain) throws ServletException, IOException {
                    
                            String header = request.getHeader("Authorization");
                            if (header != null && header.startsWith("Bearer ")) {
                                String token = header.substring(7);
                    
                                try {
                                    Claims claims = Jwts.parser()
                                            .setSigningKey(secretKey)
                                            .parseClaimsJws(token)
                                            .getBody();
                    
                                    String username = claims.getSubject();
                                    if (username != null) {
                                        SecurityContextHolder.getContext().setAuthentication(
                                                new UsernamePasswordAuthenticationToken(username, null, null));
                                    }
                                } catch (Exception ignored) {
                                }
                            }
                    
                            filterChain.doFilter(request, response);
                        }
                    }
       
               ``
                    
                    
          ⚙️ Step 4: Spring Security Configuration
          

                    ` import org.springframework.context.annotation.Configuration;
                    import org.springframework.security.config.annotation.web.builders.HttpSecurity;
                    import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
                    import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
                    
                    @Configuration
                    @EnableWebSecurity
                    public class SecurityConfig extends                          org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter {
                    
                        @Override
                        protected void configure(HttpSecurity http) throws Exception {
                            http.csrf().disable()
                                    .authorizeRequests()
                                    .antMatchers("/api/files/download/**").authenticated()
                                    .anyRequest().permitAll()
                                    .and()
                                    .addFilterBefore(new JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
                        }
                    }
                    `


          📁 Step 5: Create File Download Controller
               This endpoint returns a pre-signed URL for the requested file.


               ` import org.springframework.beans.factory.annotation.Autowired;
               import org.springframework.http.ResponseEntity;
               import org.springframework.web.bind.annotation.*;
               
               @RestController
               @RequestMapping("/api/files")
               public class FileDownloadController {
               
                   @Autowired
                   private S3Service s3Service;
               
                   @GetMapping("/download/{fileName}")
                   public ResponseEntity<String> getDownloadUrl(@PathVariable String fileName) {
                       String presignedUrl = s3Service.generatePresignedDownloadUrl(fileName);
                       return ResponseEntity.ok(presignedUrl);
                   }
               }
               `

                                  
          ⚛️ Step 6: React.js Client
               Here’s how your React.js client can securely download the file:

               ` import React, { useState } from 'react';
               import axios from 'axios';
               
               function FileDownloader() {
                   const [fileName, setFileName] = useState('');
                   const [downloadUrl, setDownloadUrl] = useState('');
                   const [error, setError] = useState('');
                   const token = localStorage.getItem('token'); // JWT token stored after login
               
                   const handleDownload = async () => {
                       try {
                           const response = await axios.get(`http://localhost:8080/api/files/download/${fileName}`, {
                               headers: {
                                   Authorization: `Bearer ${token}`
                               }
                           });
                           setDownloadUrl(response.data);
                           window.open(response.data, "_blank"); // Automatically trigger download
                       } catch (err) {
                           setError('Failed to get download URL');
                       }
                   };
               
                   return (
                       <div>
                           <input
                               type="text"
                               placeholder="Enter file name"
                               value={fileName}
                               onChange={(e) => setFileName(e.target.value)}
                           />
                           <button onClick={handleDownload}>Download File</button>
                           {error && <p style={{ color: 'red' }}>{error}</p>}
                           {downloadUrl && <p>Download link generated!</p>}
                       </div>
                   );
               }
               
               export default FileDownloader;
               `

                    
          ✅ Flow Summary     
               1 React calls /api/files/download/{fileName} with JWT in headers.
               2 Spring Boot validates JWT, generates a temporary pre-signed S3 URL, and returns it.
               3 React opens the URL → AWS S3 serves the file directly. 
               
          🧾 Advantages
               
         
       **AWS S3 2 S3 Download** 
            ✅ Secure: JWT ensures only authorized users can generate URLs.
            ✅ Efficient: Files are downloaded directly from S3 (not proxied).
            ✅ Scalable: Works for large files without stressing your backend.
            ✅ Temporary Access: Pre-signed URLs expire automatically.
            
     

     
____________________________________________________________________________
 ### Q) After successful registration, your spring boot application needs to send a welcome email to the user. Describe how would you send the emails to the registered users.
 
     To send a welcome email after a successful registration in a Spring Boot application, you can use Spring’s email support with JavaMailSender (part of the spring-boot-starter-mail dependency).

     ✅ Step 1: Add Dependencies

                    In your pom.xml, add the following dependencies:


                    `` <dependencies>
                             <!-- Spring Boot starter for sending emails -->
                             <dependency>
                                 <groupId>org.springframework.boot</groupId>
                                 <artifactId>spring-boot-starter-mail</artifactId>
                             </dependency>
                             
                             <!-- Optional: for sending HTML emails -->
                             <dependency>
                                 <groupId>org.springframework.boot</groupId>
                                 <artifactId>spring-boot-starter-thymeleaf</artifactId>
                             </dependency>
                         </dependencies>
                         ``

               
     ✅ Step 2: Configure Mail Properties

          In your application.properties (or application.yml), configure the SMTP server details:


          `` spring.mail.host=smtp.gmail.com
               spring.mail.port=587
               spring.mail.username=your_email@gmail.com
               spring.mail.password=your_app_password
               spring.mail.properties.mail.smtp.auth=true
               spring.mail.properties.mail.smtp.starttls.enable=true
               ``


               💡 Tip: If you’re using Gmail, make sure to generate an App Password (not your Gmail password) and enable 2FA.
               
          
     ✅ Step 3: Create an Email Service

          Create a EmailService class that uses JavaMailSender to send emails.
          

          `` 
          import org.springframework.beans.factory.annotation.Autowired;
          import org.springframework.mail.SimpleMailMessage;
          import org.springframework.mail.javamail.JavaMailSender;
          import org.springframework.stereotype.Service;
          
          @Service
          public class EmailService {
          
              @Autowired
              private JavaMailSender mailSender;
          
              public void sendWelcomeEmail(String toEmail, String userName) {
                  String subject = "Welcome to Our Platform!";
                  String body = "Hi " + userName + ",\n\n" +
                                "Welcome to our application! We're excited to have you on board.\n\n" +
                                "Best regards,\nThe Team";
          
                  SimpleMailMessage message = new SimpleMailMessage();
                  message.setFrom("your_email@gmail.com");
                  message.setTo(toEmail);
                  message.setSubject(subject);
                  message.setText(body);
          
                  mailSender.send(message);
              }
          }
          ``

          
          
     ✅ Step 4: Call the Email Service After Registration

          In your user registration service, send the welcome email after saving the user


          ``
               import org.springframework.beans.factory.annotation.Autowired;
               import org.springframework.stereotype.Service;
               
               @Service
               public class UserService {
               
                   @Autowired
                   private UserRepository userRepository;
               
                   @Autowired
                   private EmailService emailService;
               
                   public void registerUser(User user) {
                       // 1. Save user to database
                       userRepository.save(user);
               
                       // 2. Send welcome email
                       emailService.sendWelcomeEmail(user.getEmail(), user.getName());
                   }
               }

          ``
     
     ✅ Step 5: (Optional) Send HTML Emails

          If you want rich HTML emails, use MimeMessageHelper:


          ``
          import jakarta.mail.MessagingException;
          import jakarta.mail.internet.MimeMessage;
          import org.springframework.mail.javamail.MimeMessageHelper;
          
          public void sendHtmlWelcomeEmail(String toEmail, String userName) throws MessagingException {
              MimeMessage mimeMessage = mailSender.createMimeMessage();
              MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
          
              helper.setFrom("your_email@gmail.com");
              helper.setTo(toEmail);
              helper.setSubject("Welcome to Our Platform!");
              helper.setText("<h1>Welcome, " + userName + "!</h1><p>We're glad to have you with us.</p>", true);
          
              mailSender.send(mimeMessage);
          }
          ``


          
     ✅ Step 6: Testing

          You can test the functionality with a simple controller endpoint:


               ``
               import org.springframework.web.bind.annotation.*;

               @RestController
               @RequestMapping("/api/users")
               public class UserController {
               
                   @Autowired
                   private UserService userService;
               
                   @PostMapping("/register")
                   public String registerUser(@RequestBody User user) {
                       userService.registerUser(user);
                       return "User registered and welcome email sent!";
                   }
               }
               ``

          
     💡 Best Practices
          * Use asynchronous sending with @Async to avoid blocking registration response.
          * Use a templating engine (Thymeleaf or FreeMarker) for beautiful email layouts.
          * Use transactional events to ensure the email is sent only after successful registration.
          
     
____________________________________________________________________________
 ### Q) What is spring boot CLI and how to execute the Spring Boot project using boot CLI ? 

             🧩 What is Spring Boot CLI?
                 Spring Boot CLI (Command Line Interface) is a lightweight tool that allows you to:
                      * Quickly create, run, and test Spring Boot applications using simple Groovy scripts or Maven/Gradle projects.
                      * Avoid writing boilerplate Java code — you can write concise Groovy-based Spring Boot apps that auto-configure                               themselves.
                      * Run and manage Spring Boot projects directly from the terminal, without needing an IDE.

           🧱 Option 2: Manual installation
                     * https://spring.io/tools Then unzip and add the bin/ directory to your system PATH.
                       Check installation:    spring --version     console output Spring Boot v3.4.0
                       
           🧩 4. Running an Existing Spring Boot Project (Maven/Gradle)
           
                spring run src/main/java/com/example/DemoApplication.java or simpley mvn spring-boot:run
                

           
____________________________________________________________________________
 ### Q) HOW IS SPRING Security Implemented In a Spring Boot Application ? 

     Spring Security is implemented in a Spring Boot application to handle authentication, authorization, and protection against common security threats (like CSRF, XSS, session fixation, etc.).

     🧩 1. Add Spring Security Dependency


          `` 
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-security</artifactId>
          </dependency>
          ``


          ✅ This automatically activates Spring Security’s auto-configuration, which by default:
               Secures all endpoints
               Provides a default login page
               Uses an in-memory user (user) with a generated password (shown in the console)
               
          
     🔐 2. Create a Security Configuration Class

          In modern Spring Boot (v3.x+), you use the SecurityFilterChain bean instead of extending 
               deprecated WebSecurityConfigurerAdapter.     
          
          ``
               @Configuration
               @EnableWebSecurity
               public class SecurityConfig {
               
                   @Bean
                   public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
                       http
                           // Define authorization rules
                           .authorizeHttpRequests(auth -> auth
                               .requestMatchers("/public/**").permitAll()   // accessible to all
                               .requestMatchers("/admin/**").hasRole("ADMIN") // only for ADMIN role
                               .anyRequest().authenticated()   // everything else requires login
                           )
                           // Configure form login
                           .formLogin(form -> form
                               .loginPage("/login")   // custom login page
                               .permitAll()
                           )
                           // Configure logout
                           .logout(logout -> logout
                               .logoutUrl("/logout")
                               .permitAll()
                           );
                       return http.build();
                   }
               
                   // Define a user for testing (in-memory)
                   @Bean
                   public UserDetailsService userDetailsService() {
                       UserDetails user = User
                           .withUsername("user")
                           .password(passwordEncoder().encode("password"))
                           .roles("USER")
                           .build();
               
                       UserDetails admin = User
                           .withUsername("admin")
                           .password(passwordEncoder().encode("admin123"))
                           .roles("ADMIN")
                           .build();
               
                       return new InMemoryUserDetailsManager(user, admin);
                   }
               
                   // Password encoder
                   @Bean
                   public PasswordEncoder passwordEncoder() {
                       return new BCryptPasswordEncoder();
                   }
               }
               `
          

          
     🧠 3. Authentication vs Authorization
          Spring Security manages both via:
          * AuthenticationManager
          * UserDetailsService
          * GrantedAuthority (for roles/permissions)

          
               
     ⚙️ 4. Common Authentication Approaches
          Spring Security supports multiple authentication mechanisms:


          | Mechanism                   | Description                                              |
          | --------------------------- | -------------------------------------------------------- |
          | **Form Login**              | Traditional login with username & password form          |
          | **HTTP Basic Auth**         | Simple header-based authentication (often used for APIs) |
          | **JWT Tokens**              | For stateless authentication in REST APIs                |
          | **OAuth2 / OpenID Connect** | For third-party login (e.g., Google, GitHub)             |
          | **LDAP Authentication**     | For enterprise environments                              |

          
     🪶 5. Securing a REST API (Stateless Example with JWT)
          For REST APIs, you typically disable sessions and CSRF:


          ``
               @Bean
               public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
                   http
                       .csrf(csrf -> csrf.disable())   // disable CSRF for APIs
                       .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                       .authorizeHttpRequests(auth -> auth
                           .requestMatchers("/auth/**").permitAll()
                           .anyRequest().authenticated()
                       )
                       .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
                   return http.build();
               }
               `

              
     🧰 6. Additional Security Features
          Spring Security automatically provides:
          
          🔒 CSRF Protection (enabled by default for form logins)
          🧩 Password Encryption using BCryptPasswordEncoder
          🧱 CORS Configuration for cross-origin requests
          🕵️ Method-Level Security using annotations like below:


                         
               `` @PreAuthorize("hasRole('ADMIN')")
                  @PostAuthorize("returnObject.user == authentication.name")          
               ``

               
               Enable it using:
                    
                `` @EnableMethodSecurity
                ``

     🧾 Summary


          | Step | Description                                            |
          | ---- | ------------------------------------------------------ |
          | 1    | Add `spring-boot-starter-security`                     |
          | 2    | Create `SecurityConfig` with `SecurityFilterChain`     |
          | 3    | Define authentication rules (in-memory, DB, JWT, etc.) |
          | 4    | Configure authorization for endpoints                  |
          | 5    | Enable encryption (`BCryptPasswordEncoder`)            |
          | 6    | Test secured endpoints                                 |

     
____________________________________________________________________________
 ### Q) how to disable a specific Auto-Configuration ? 
 
          In Spring Boot, auto-configuration is a key feature that automatically configures beans based on the classpath and environment. However, sometimes you may need to disable a specific auto-configuration because it conflicts with your setup or you want to manually configure something.

          Here are 4 common ways to disable specific auto-configurations:
          
          ✅ 1. Using @SpringBootApplication(exclude = …)
                    This is the most common and direct way.
                    

                    ``
                         import org.springframework.boot.autoconfigure.SpringBootApplication;
                         import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

                         @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
                         public class MyApplication {
                             public static void main(String[] args) {
                                 SpringApplication.run(MyApplication.class, args);
                             }
                         }
                         ``

                      🟢 When to use:
                           When you know exactly which auto-configuration class to exclude and want to apply it globally.   
                    
          ✅ 2. Using @EnableAutoConfiguration(exclude = …)
                    This works the same as above but is more explicit.


                         `` 
                         import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
                         import org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration;
                         import org.springframework.context.annotation.Configuration;
                         
                         @Configuration
                         @EnableAutoConfiguration(exclude = { HibernateJpaAutoConfiguration.class })
                         public class AppConfig { }
                         ``

                              🟢 When to use:
                                        When you don’t use @SpringBootApplication (e.g., in a library or a non-main config class).

               
          ✅ 3. Using spring.autoconfigure.exclude in application.properties or application.yml
                    This method lets you disable auto-configurations without touching code.


                    `` spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
                    ``

                    🟢 When to use:
                              If you want to disable auto-configurations based on environment or profile without recompiling code.
                                        
               
          ✅ 4. Programmatically via SpringApplicationBuilder     

               You can disable it when creating the SpringApplication.


                    `` 
                   new SpringApplicationBuilder(MyApplication.class)
                  .web(WebApplicationType.SERVLET)
                  .properties("spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration")
                  .run(args);
                  ``

                  
               🟢 When to use:
                         In advanced or programmatic startup scenarios.
                    


      💡 How to know which auto-configuration to exclu
          If you’re unsure which class to exclude:
               Run your app with --debug flag or set
                    debug = true
               Spring boot will log "AUTO-CONFIGURATION REPORT", showing which auto configurations were applied or not.

          
       🧠 Example use case
            If you want to use your own database configuration instead of Spring Boot’s:


            ``
                 @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
                    public class CustomDbApp { ... }
            ``

          
____________________________________________________________________________
 ### Q) explain the difference between cache eviction and cache expiration.

____________________________________________________________________________
 ### Q) If you had to scale a Spring Boot application to handle high traffic, what strategies would you use ? 

____________________________________________________________________________
 ### Q) Describe how to implement security in a microservices architecture using spring boot and spring security.

____________________________________________________________________________
 ### Q) In Spring boot how is session management configured and handled, especially in distributed systems. ? 

____________________________________________________________________________
 ### Q) Imagine you are designing a spring boot application that interfaces with multiple external APIs . How would you handle API rate limits and failures ? 

____________________________________________________________________________
 ### Q) Imagine you are designing a Spring Boot application that interfaces with multiple external APIs. How would you handle API rate limits and failures ?

____________________________________________________________________________
 ### Q) How you would manage externalized configuration and secure sensitive configuration properties in a microservices architecture ? 

____________________________________________________________________________
 ### Q) how does spring boot support internationalization (i18n) ? 

____________________________________________________________________________
 ### Q) What is spring boot DevTools used for ?

____________________________________________________________________________
 ### Q) How can you mock external services in a SpringBoot test ? 
 

____________________________________________________________________________
 ### Q) how do you mock microservices during testing ?

____________________________________________________________________________
 ### Q) Explain the process of creating a Docker image for a Spring Boot application.

____________________________________________________________________________
 ### Q) Discuss the configuration of Spring security to address common security concerns.

____________________________________________________________________________
 ### Q) Discuss how would you secure a Spring Boot application using JSON Web Token (JWT) ? 

____________________________________________________________________________
 ### Q)  How can Spring Boot applications be made more resilient to failures, especially in Microservices architectures ? 

____________________________________________________________________________
 ### Q) Explain the conversion fo business logic into serverless functions with Spring Cloud Functions.

____________________________________________________________________________
 ### Q) How can spring cloud gateway be configured for routing, security and monitoring ? 

____________________________________________________________________________
 ### Q) how would you manage and monitor asynchronous tasks in spring boot application, ensuring that you can track task progress and handle failures ?

____________________________________________________________________________
 ### Q) You application needs to process notifications asynchronously using a message queue. Explain how you would setup integration and send message from your spring boot application.

____________________________________________________________________________
 ### Q) You need to secure a spring boot app, to ensure that only authenticated users can access certain endpoints. Describe how you would configure spring security to set up a basic for-based authentication.

____________________________________________________________________________
 ### Q) How to tell an Auto-Configuration to Back Away When a Bean Exists ? 

____________________________________________________________________________
 ### Q) How to deploy spring boot web applications as jar and war files ? 

____________________________________________________________________________
 ### Q) What Does It Mean Spring Boot Supports Relaxed Binding ? 
     
____________________________________________________________________________
 ### Q) Discuss the integration of Spring Boot applications with CI/CD pipelines.

____________________________________________________________________________
 ### Q) How to resolve whitelabel error page in the spring boot applications ?

____________________________________________________________________________
 ### Q) how can we implement pagination in springboot application ?

____________________________________________________________________________
 ### Q) how to handle 404 error in spring boot application ? 

____________________________________________________________________________
 ### Q) How can spring boot be used to implement event-driven architectures ?

____________________________________________________________________________
 ### Q) Discuss the integration and use of distributed tracing in spring boot applications for monitoring and troubleshooting.
     - Spring cloud sleut
     - Zipkin
____________________________________________________________________________
 ### Q) Your app need to store and retrieve files from a cloud storage service. Describe how you would integrate this functionality into a Spring Boot App ?

____________________________________________________________________________
 ### Q) To protect ur application from abuse and ensure fair usage, you decide to implement rate limiting on ur API endpoints. Describe a simple approach to achieve this in Spring Boot.

____________________________________________________________________________
 ### Q) Your are tasked with building a non-blocking , reactive REST API that can handle a high volume of concurrent requests efficiently. Describe how would use spring webFlux to achieve this ?

____________________________________________________________________________
 ### Q) Can you explain the Blue-Green deployment strategy ?

____________________________________________________________________________
 ### Q) How do you optimize memory management when designing Java application ? 

____________________________________________________________________________
 ### Q)  What are the ways to adjust JVM memory settings during runtime in a Java application?

____________________________________________________________________________
 ### Q) How to apply Testcontainers in TDD and BDD test development?

____________________________________________________________________________
 ### Q) 

____________________________________________________________________________
 ### Q) 

____________________________________________________________________________
 ### Q) 

____________________________________________________________________________
 ### Q) 

____________________________________________________________________________

