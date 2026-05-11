# Java 业务逻辑漏洞

## 竞争条件

```java
// 无锁扣减
@Transactional
public Order createOrder(Long productId, int quantity) {
    Product product = productRepository.findById(productId).orElseThrow();
    if (product.getStock() >= quantity) {
        product.setStock(product.getStock() - quantity);
        productRepository.save(product);
        return orderRepository.save(new Order(productId, quantity));
    }
    // 并发请求同时通过检查 → 超卖
}

// 悲观锁
@Transactional
public Order createOrder(Long productId, int quantity) {
    Product product = productRepository.findByIdWithLock(productId);  // SELECT ... FOR UPDATE
    if (product.getStock() < quantity) throw new InsufficientStockException();
    product.setStock(product.getStock() - quantity);
    return orderRepository.save(new Order(productId, quantity));
}

// 乐观锁
@Entity
public class Product {
    @Version
    private Long version;
    private int stock;
}
// 更新时JPA自动检查version → 并发冲突时抛OptimisticLockException
```

## 价格操纵

```java
// 价格来自客户端
@PostMapping("/order/create")
public Order create(@RequestBody OrderRequest req) {
    order.setTotalPrice(req.getPrice() * req.getQuantity());  // 修改 price=0
    return orderRepository.save(order);
}

// 价格从数据库取
@PostMapping("/order/create")
public Order create(@RequestBody OrderRequest req) {
    Product product = productRepository.findById(req.getProductId()).orElseThrow();
    BigDecimal total = product.getPrice().multiply(BigDecimal.valueOf(req.getQuantity()));
    order.setTotalPrice(total);
    return orderRepository.save(order);
}
```

## 状态机

```java
// 状态可跳跃
@PostMapping("/order/{id}/status")
public Order updateStatus(@PathVariable Long id, @RequestParam String status) {
    Order order = orderRepository.findById(id).orElseThrow();
    order.setStatus(status);  // PENDING → DELIVERED
    return orderRepository.save(order);
}

// 状态机约束
public enum OrderStatus {
    PENDING { OrderStatus next() { return PAID; } },
    PAID { OrderStatus next() { return SHIPPED; } },
    SHIPPED { OrderStatus next() { return DELIVERED; } },
    DELIVERED { OrderStatus next() { return this; } };
    abstract OrderStatus next();
}
```

## OTP/密码重置

```java
// Token可爆破
String token = String.valueOf(new Random().nextInt(999999));  // 4-6位数字
// 无尝试次数限制 → 可爆破

// 安全
String token = UUID.randomUUID().toString();  // 128位随机
tokenRepository.save(new ResetToken(token, userId, Instant.now().plus(5, ChronoUnit.MINUTES)));
// 验证时: 绑定userId + 检查过期 + 一次性使用 + 尝试次数限制
```
