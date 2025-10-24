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

     

     
        
   ```
| Approach         | Type  | Tool/Library           | Pros                       | Use Case                   |
| ---------------- | ----- | ---------------------- | -------------------------- | -------------------------- |
| `RestTemplate`   | Sync  | Spring Web             | Simple                     | Small apps                 |
| `WebClient`      | Async | Spring WebFlux         | Non-blocking               | Reactive apps              |
| `FeignClient`    | Sync  | Spring Cloud OpenFeign | Declarative                | Cloud-native microservices |
| Kafka/RabbitMQ   | Async | Messaging              | Decoupled                  | Event-driven systems       |
| Eureka + Gateway | Infra | Spring Cloud           | Service discovery, routing | Distributed systems        |


#####
  
