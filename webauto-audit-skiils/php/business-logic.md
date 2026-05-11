# PHP 业务逻辑漏洞

## 竞争条件

```php
// 扣减无锁
function createOrder($productId, $quantity) {
    $product = Product::find($productId);
    if ($product->stock >= $quantity) {
        $product->stock -= $quantity;
        $product->save();
        return Order::create([...]);
    }
}

// 数据库原子操作
function createOrder($productId, $quantity) {
    $affected = Product::where('id', $productId)
        ->where('stock', '>=', $quantity)
        ->update(['stock' => \DB::raw("stock - $quantity")]);
    if ($affected === 0) throw new \Exception('库存不足');
}

// 悲观锁
\DB::transaction(function () use ($productId, $quantity) {
    $product = Product::lockForUpdate()->find($productId);
    if ($product->stock < $quantity) throw new \Exception('库存不足');
    $product->stock -= $quantity;
    $product->save();
});
```

## 价格操纵

```php
// 价格来自客户端
function createOrder(Request $req) {
    $total = $req->price * $req->quantity;  // 修改 price=0
    Order::create(['total_price' => $total]);
}

// 价格从数据库取
function createOrder(Request $req) {
    $product = Product::findOrFail($req->product_id);
    $total = $product->price * $req->quantity;
    Order::create(['total_price' => $total]);
}
```

## 优惠券滥用

```php
// 无幂等
function applyCoupon($userId, $couponCode) {
    $coupon = Coupon::where('code', $couponCode)->first();
    if (!$coupon->used) {                    // 并发通过
        $coupon->used = true;
        $coupon->user_id = $userId;
        $coupon->save();
    }
}

// 数据库原子操作
function applyCoupon($userId, $couponCode) {
    $affected = Coupon::where('code', $couponCode)
        ->where('used', false)
        ->update(['used' => true, 'user_id' => $userId]);
    if ($affected === 0) throw new \Exception('已使用');
}
```

## OTP/验证码

```php
// 漏洞
$otp = rand(100000, 999999);     // 弱随机
session(['otp' => $otp]);        // 无过期、无尝试次数限制
// 验证码出现在响应中: return ['otp' => $otp];

// 安全
$otp = random_int(100000, 999999);
Cache::put("otp:{$userId}:password_reset", hash('sha256', $otp), now()->addMinutes(5));
// 验证时: 检查尝试次数 + 一次性使用 + 过期时间
```

## 检测要点

- 库存/余额扣减是否使用数据库原子操作或悲观锁
- 价格/金额是否来自服务端数据库
- 优惠券是否有唯一约束
- 验证码是否有过期和尝试限制
- 关键操作是否有幂等Token
