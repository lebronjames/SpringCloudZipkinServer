SpringCloud服务追踪组件zipkin，Spring Cloud Sleuth集成了zipkin组件。
Spring Cloud Sleuth 主要功能就是在分布式系统中提供追踪解决方案
一、构建zipkin-server
1) pom.xml引入依赖spring-boot-starter、spring-boot-starter-web、
spring-boot-starter-test、zipkin-server、zipkin-autoconfigure-ui。
2)程序入口类, 加上注解@EnableZipkinServer，开启ZipkinServer的功能
3)配置文件application.yml
server:
  port: 9411
  
#表示zipkin数据存储方式是mysql
zipkin:
  storage:
    type: mysql
#数据库脚本创建地址，当有多个是可使用[x]表示集合第几个元素
spring:
  application:
    name: zipkin-server
#zipkin数据保存到数据库中需要进行如下配置
#表示当前程序不使用sleuth
  slenth:
    enable: false
    sampler:
      percentage: 1
#spring boot数据源配置
  datasource:
    url: jdbc:mysql://10.5.2.242:3306/zipkin?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
    initialize: true
    continue-on-error: true
4) 初始化sql脚本zipkin.sql导入到mysql数据库。
 
二、构建zipkin-client
1) pom.xml引入依赖spring-boot-starter-web、spring-cloud-starter-zipkin、spring-boot-starter-test。
2) 配置文件application.yml
server:
  port: 8988
spring:
  application:
    name: zipkinClient1
  zipkin:
    base-url: http://localhost:9411
3) 程序主类配置
@SpringBootApplication
@RestController
public class ServiceHiApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceHiApplication.class, args);
    }

    private static final Logger LOG = Logger.getLogger(ServiceHiApplication.class.getName());


    @Autowired
    private RestTemplate restTemplate;

    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    @RequestMapping("/hi")
    public String callHome(){
        LOG.log(Level.INFO, "calling trace service-hi  ");
        return restTemplate.getForObject("http://localhost:8989/miya", String.class);
    }
    @RequestMapping("/info")
    public String info(){
        LOG.log(Level.INFO, "calling trace service-hi ");

        return "i'm service-hi";

    }

    @Bean
    public AlwaysSampler defaultSampler(){
        return new AlwaysSampler();
    }
}

三、访问地址
http://localhost:9411/
http://localhost:8989/miya
http://localhost:8988/hi



