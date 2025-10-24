### Interservice communication between Microservices using spring boot ###
##### In Spring Boot, several mechanisms are available to achieve interservice communication, depending on the architecture style (synchronous or asynchronous) and reliability needs.
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
#### Q) What are some recommended approaches for versioning REST APIs in Spring Boot?

1. URI Path Versioning (Most Common)

Example:

```
GET /api/v1/customers
GET /api/v2/customers
```
Implementation:

```
@RestController
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
}
```

Pros:
     * imple and intuitivev
     * Easy for clients to switch between versions
     * Clear separation of versions
cons:
     * URL clutter     
     * Might duplicate controllers for each version


2. Request Parameter Versioning

     Example:
          ```
          GET /api/customers?version=1  
          GET /api/customers?version=2
          ```
     Implementation:
          ```
               @RestController
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
             Headers: X-API-VERSION=1
         ```

    or using Accept Header:
        ```
               Accept: application/vnd.example.v1+json
     ```

     Implementation:
     ```
        @RestController
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

     ```
     GET /api/customers
     Accept: application/vnd.example.v1+json
     ```
Implementat
```
     @RestController
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
        ```
        https://v1.api.example.com/customers
        https://v2.api.example.com/customers
   ```     
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



   
