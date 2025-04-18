(현재 버전 문제가 생겨 프로젝트 수정 중입니다.)  

# PeakCache

PeakCache는 Spring Boot 애플리케이션을 위한 고성능 캐싱 라이브러리입니다. 
로컬 메모리(Caffeine)와 Redis를 함께 사용하여 높은 성능과 분산 캐싱 기능을 제공합니다.
피크 타임에 밀려오는 대규모 트래픽을 처리할 용도로 만들었습니다.

## 주요 기능

- **다중 계층 캐싱**: L1 캐시(로컬 메모리)와 L2 캐시(Redis)를 통합 관리
- **메모리 최적화**: 선택적 필드 캐싱 및 데이터 압축 기능
- **Spring Boot 자동 구성**: 최소한의 설정으로 쉽게 통합
- **애노테이션 기반 캐싱**: `@Cacheable`, `@CachePut`, `@CacheEvict` 지원
- **장애 대응**: L2 캐시(Redis) 장애 시 L1 캐시로 폴백 기능

## 요구사항

- Java 21 이상
- Spring Boot 3.4.3 이상
- Redis 서버

## 설치 방법

### Gradle

```gradle
implementation 'com.peakcache:peakcache:0.1.0'
```

### Maven

```xml
<dependency>
    <groupId>com.peakcache</groupId>
    <artifactId>peakcache</artifactId>
    <version>0.1.0</version>
</dependency>
```

## 기본 설정

`application.yml` 에서 기본 설정을 구성 예시입니다.

```yaml
peakcache:
  enabled: true
  enable-aop: true
  local:
    maximum-size: 10000
    expire-after-write-seconds: 300
    expire-after-access-seconds: 600
  redis:
    expiration-seconds: 3600
    key-prefix: "app"
    fallback-enabled: true
  memory:
    compression-enabled: true
    selective-field-enabled: true
    default-cache-fields:
      - id
      - name
      - price
```

## 사용 예제

### 애노테이션 기반 캐싱

```java
@Service
public class ProductService {
    
    @Cacheable(keyPrefix = "product", expiration = 3600)
    public Product getProduct(Long productId) {
        // DB에서 상품 조회
        return productRepository.findById(productId).orElseThrow();
    }
    
    @CachePut(key = "'product:' + #product.id", expiration = 3600)
    public Product updateProduct(Product product) {
        // 상품 업데이트
        return productRepository.save(product);
    }
    
    @CacheEvict(key = "'product:' + #productId")
    public void deleteProduct(Long productId) {
        // 상품 삭제
        productRepository.deleteById(productId);
    }
}
```

### 프로그래밍 방식 캐싱

```java
@Service
public class UserService {
    
    private final CacheManager cacheManager;
    
    public UserService(CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }
    
    public User getUser(Long userId) {
        String key = "user:" + userId;
        
        // 캐시에서 조회
        User user = cacheManager.get(key);
        if (user != null) {
            return user;
        }
        
        // DB에서 조회
        user = userRepository.findById(userId).orElseThrow();
        
        // 캐시에 저장
        cacheManager.put(key, user, 1, TimeUnit.HOURS);
        
        return user;
    }
}
```

## 성능 최적화

### 선택적 필드 캐싱

```java
@Cacheable(keyPrefix = "product", fields = {"id", "name", "price"})
public Product getProduct(Long productId) {
    return productRepository.findById(productId).orElseThrow();
}
```

## License

MIT License
