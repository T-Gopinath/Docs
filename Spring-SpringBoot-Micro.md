### Interservice communication between Microservices using spring boot ###
##### In Spring Boot, several mechanisms are available to achieve interservice communication, depending on the architecture style (synchronous or asynchronous) and reliability needs.
1. Synchronous Communication (Request-Response Style)
     one service directly calls another and waits for the response.
     Simple, blocking, and synchronous.

```
<b> A. REST Communication using RestTemplate (Legacy Approach)</b>

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
```
     
3. Asynchronous Communication (Event-Driven Style)


#####
  
