# NestJS 安全审计

## 识别特征

```
装饰器: @Module(), @Controller(), @Injectable(), @Get(), @Post()
文件: main.ts, app.module.ts, *.controller.ts, *.service.ts
```

## Guards 遗漏

```typescript
// Controller 无 Guard
@Controller('admin')
export class AdminController {
    @Get('users')
    getUsers() { return this.userService.findAll(); }
    // 无 @UseGuards → 任何人可访问
}

// 个别路由无 Guard
@Controller('api')
@UseGuards(AuthGuard)           // Controller 级 Guard
export class ApiController {
    @Get('public')
    @Public()                    // 但有 @Public() 标记
    getPublic() { ... }

    @Get('secret')
    getSecret() { ... }          // 继承 Controller 级 Guard
}

// 全局 Guard
// main.ts
app.useGlobalGuards(new AuthGuard());
// 个别路由用 @Public() 标记例外
```

## Validation / DTO

```typescript
// 无 Validation
@Post('user')
createUser(@Body() body: any) {  // any → Mass Assignment
    return this.userService.create(body);
}

// DTO + class-validator
export class CreateUserDto {
    @IsString()
    @Length(2, 50)
    name: string;

    @IsEmail()
    email: string;

    // 不包含 isAdmin, role 等敏感字段
}

@Post('user')
createUser(@Body() dto: CreateUserDto) {
    return this.userService.create(dto);
}
```

## JWT / Passport

```typescript
// JWT 配置弱
JwtModule.register({
    secret: '123456',         // 弱密钥
    signOptions: { expiresIn: '7d' }  // 过长
});

// 安全
JwtModule.register({
    secret: process.env.JWT_SECRET,
    signOptions: { expiresIn: '15m' }
});
```

## CORS

```typescript
// main.ts
// 漏洞
app.enableCors();  // 默认允许所有来源

// 安全
app.enableCors({
    origin: ['https://app.com'],
    methods: ['GET', 'POST'],
    credentials: true
});
```

## Swagger

```typescript
// 生产环境必须禁用
// main.ts
if (process.env.NODE_ENV !== 'production') {
    const config = new DocumentBuilder().build();
    const document = SwaggerModule.createDocument(app, config);
    SwaggerModule.setup('api', app, document);
}
```

## 检测清单

- [ ] 全局 Guard 或每个 Controller 有 @UseGuards
- [ ] DTO 使用 class-validator 限制字段
- [ ] JWT secret 强度足够
- [ ] CORS 白名单
- [ ] Swagger 生产禁用
- [ ] Helmet 已配置
- [ ] Rate limiting (throttler) 已启用
