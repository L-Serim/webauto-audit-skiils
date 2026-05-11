# Java IDOR 越权

## 检测

```java
// 直接通过ID查询 — 无归属校验
@GetMapping("/api/order/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderRepository.findById(id).orElseThrow();
}

// UUID 不等于安全
@GetMapping("/api/invoice/{uuid}")
public Invoice getInvoice(@PathVariable String uuid) {
    return invoiceRepository.findByUuid(uuid);
}

// 批量操作
@PostMapping("/api/orders/delete")
public void batchDelete(@RequestBody List<Long> ids) {
    orderRepository.deleteAllById(ids);
}

// 文件下载
@GetMapping("/download/{filename}")
public ResponseEntity<Resource> download(@PathVariable String filename) {
    Path file = storageService.load(filename);
    return ResponseEntity.ok().body(new UrlResource(file.toUri()));
}

// 密码修改 — 可指定目标用户
@PostMapping("/api/user/password")
public void changePassword(@RequestParam Long userId, @RequestParam String newPass) {
    User user = userRepository.findById(userId).orElseThrow();
    user.setPassword(passwordEncoder.encode(newPass));
    userRepository.save(user);
}
```

## 修复

```java
// 资源归属校验
@GetMapping("/api/order/{id}")
public Order getOrder(@PathVariable Long id) {
    Long currentUserId = getCurrentUserId();
    return orderRepository.findByIdAndUserId(id, currentUserId)
        .orElseThrow(() -> new AccessDeniedException("Not your order"));
}

// 批量归属
@PostMapping("/api/orders/delete")
public void batchDelete(@RequestBody List<Long> ids) {
    Long userId = getCurrentUserId();
    for (Long id : ids) {
        Order order = orderRepository.findByIdAndUserId(id, userId)
            .orElseThrow(() -> new AccessDeniedException());
        orderRepository.delete(order);
    }
}

// 密码修改 — 只能修改自己
@PostMapping("/api/user/password")
public void changePassword(@RequestParam String newPass) {
    User user = userRepository.findById(getCurrentUserId()).orElseThrow();
    user.setPassword(passwordEncoder.encode(newPass));
    userRepository.save(user);
}

// @PreAuthorize + 自定义 PermissionEvaluator
@PreAuthorize("hasPermission(#id, 'Order', 'READ')")
@GetMapping("/api/order/{id}")
public Order getOrder(@PathVariable Long id) { ... }
```

## 检测命令

```bash
grep -rn "findById\|getById\|findByUuid" --include="*.java" | grep -v "userId\|currentUser"
grep -rn "@PathVariable\|@RequestParam.*id" --include="*.java"
grep -rn "deleteAllById\|deleteById" --include="*.java"
```
