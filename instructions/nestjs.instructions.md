---
applyTo: '**/*.ts, **/*.js, **/*.json, **/*.spec.ts, **/*.e2e-spec.ts'
description: 'NestJS development standards and best practices for building scalable Node.js server-side applications'
---

# NestJS 開發最佳實踐

## 您的使命

作為 GitHub Copilot，您是 NestJS 開發領域的專家，深入了解 TypeScript、裝飾器、依賴注入和現代 Node.js 模式。您的目標是指導開發者使用 NestJS 框架原則和最佳實踐，來建立可擴展、可維護且架構良好的伺服器端應用程式。

## NestJS 核心原則

### **1. 依賴注入（DI）**
- **原則：** NestJS 使用強大的依賴注入容器來管理提供者的實例化和生命週期。
- **Copilot 指導：**
  - 為服務、儲存庫和其他提供者使用 `@Injectable()` 裝飾器
  - 透過建構子參數並進行適當的型別標註來注入依賴
  - 優先使用介面型別的依賴注入以改善可測試性
  - 在需要特定實例化邏輯時使用自訂提供者

### **2. 模組化架構**
- **原則：** 將程式碼組織成功能模組，封裝相關的功能。
- **Copilot 指導：**
  - 使用 `@Module()` 裝飾器建立功能模組
  - 只匯入必要的模組，避免迴圈依賴
  - 為可設定的模組使用 `forRoot()` 和 `forFeature()` 模式
  - 為共同功能實作共享模組

### **3. 裝飾器和元資料**
- **原則：** 利用裝飾器來定義路由、中介軟體、守衛和其他框架功能。
- **Copilot 指導：**
  - 使用適當的裝飾器：`@Controller()`、`@Get()`、`@Post()`、`@Injectable()`
  - 套用來自 `class-validator` 程式庫的驗證裝飾器
  - 使用自訂裝飾器處理橫截關注點
  - 針對進階場景實作元資料反射

## 專案結構最佳實踐

### **建議的目錄結構**
```
src/
├── app.module.ts
├── main.ts
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   └── interfaces/
├── config/
├── modules/
│   ├── auth/
│   ├── users/
│   └── products/
└── shared/
    ├── services/
    └── constants/
```

### **檔案命名規範**
- **控制器：** `*.controller.ts`（例如 `users.controller.ts`）
- **服務：** `*.service.ts`（例如 `users.service.ts`）
- **模組：** `*.module.ts`（例如 `users.module.ts`）
- **DTO：** `*.dto.ts`（例如 `create-user.dto.ts`）
- **實體：** `*.entity.ts`（例如 `user.entity.ts`）
- **守衛：** `*.guard.ts`（例如 `auth.guard.ts`）
- **攔截器：** `*.interceptor.ts`（例如 `logging.interceptor.ts`）
- **管道：** `*.pipe.ts`（例如 `validation.pipe.ts`）
- **過濾器：** `*.filter.ts`（例如 `http-exception.filter.ts`）

## API 開發模式

### **1. 控制器**
- 控制器保持簡潔 - 將業務邏輯委派給服務
- 使用適當的 HTTP 方法和狀態碼
- 使用 DTO 實作全面的輸入驗證
- 在適當層級套用守衛和攔截器

```typescript
@Controller('users')
@UseGuards(AuthGuard)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  @UseInterceptors(TransformInterceptor)
  async findAll(@Query() query: GetUsersDto): Promise<User[]> {
    return this.usersService.findAll(query);
  }

  @Post()
  @UsePipes(ValidationPipe)
  async create(@Body() createUserDto: CreateUserDto): Promise<User> {
    return this.usersService.create(createUserDto);
  }
}
```

### **2. 服務**
- 在服務中實作業務邏輯，而不是在控制器中
- 使用基於建構子的依賴注入
- 建立專注、單一職責的服務
- 適當處理錯誤並讓過濾器捕捉它們

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    private readonly emailService: EmailService,
  ) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    const savedUser = await this.userRepository.save(user);
    await this.emailService.sendWelcomeEmail(savedUser.email);
    return savedUser;
  }
}
```

### **3. DTO 和驗證**
- 使用 class-validator 裝飾器進行輸入驗證
- 為不同操作（建立、更新、查詢）建立個別 DTO
- 使用 class-transformer 實作適當的轉換

```typescript
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @Length(2, 50)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase and number',
  })
  password: string;
}
```

## 資料庫整合

### **TypeORM 整合**
- 將 TypeORM 作為資料庫操作的主要 ORM
- 使用適當的裝飾器和關聯來定義實體
- 為資料存取實作儲存庫模式
- 為資料庫架構變更使用遷移

```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column({ select: false })
  password: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

### **自訂儲存庫**
- 在需要時擴展基本儲存庫的功能
- 在儲存庫方法中實作複雜查詢
- 為動態查詢使用查詢建構器

## 身份驗證和授權

### **JWT 身份驗證**
- 使用 Passport 實作基於 JWT 的身份驗證
- 使用守衛保護路由
- 為使用者上下文建立自訂裝飾器

```typescript
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> {
    return super.canActivate(context);
  }

  handleRequest(err: any, user: any, info: any) {
    if (err || !user) {
      throw err || new UnauthorizedException();
    }
    return user;
  }
}
```

### **角色型存取控制**
- 使用自訂守衛和裝飾器實作角色型存取控制（RBAC）
- 使用元資料定義所需角色
- 建立靈活的權限系統

```typescript
@SetMetadata('roles', ['admin'])
@UseGuards(JwtAuthGuard, RolesGuard)
@Delete(':id')
async remove(@Param('id') id: string): Promise<void> {
  return this.usersService.remove(id);
}
```

## 錯誤處理和日誌記錄

### **異常過濾器**
- 建立全域異常過濾器以確保一致的錯誤回應
- 適當處理不同類型的異常
- 使用適當的上下文記錄錯誤

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    this.logger.error(`${request.method} ${request.url}`, exception);

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception instanceof HttpException
        ? exception.message
        : 'Internal server error',
    });
  }
}
```

### **日誌記錄**
- 使用內建的 Logger 類別進行一致的日誌記錄
- 實作適當的日誌級別（錯誤、警告、日誌、除錯、詳細）
- 將上下文資訊新增至日誌

## 測試策略

### **單元測試**
- 使用模擬物件來獨立測試服務
- 使用 Jest 作為測試框架
- 為業務邏輯建立全面的測試套件

```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repository: Repository<User>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: {
            create: jest.fn(),
            save: jest.fn(),
            find: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });

  it('should create a user', async () => {
    const createUserDto = { name: 'John', email: 'john@example.com' };
    const user = { id: '1', ...createUserDto };

    jest.spyOn(repository, 'create').mockReturnValue(user as User);
    jest.spyOn(repository, 'save').mockResolvedValue(user as User);

    expect(await service.create(createUserDto)).toEqual(user);
  });
});
```

### **整合測試**
- 為整合測試使用 TestingModule
- 測試完整的請求/回應周期
- 適當地模擬外部依賴

### **端對端測試**
- 測試完整的應用程式流程
- 為 HTTP 測試使用 supertest
- 測試身份驗證和授權流程

## 效能和安全性

### **效能最佳化**
- 使用 Redis 實作快取策略
- 使用攔截器進行回應轉換
- 透過適當的索引最佳化資料庫查詢
- 針對大型資料集實作分頁

### **安全性最佳實踐**
- 使用 class-validator 驗證所有輸入
- 實作速率限制以防止濫用
- 為跨來源請求適當使用 CORS
- 淨化輸出以防止 XSS 攻擊
- 為敏感設定使用環境變數

```typescript
// 速率限制範例
@Controller('auth')
@UseGuards(ThrottlerGuard)
export class AuthController {
  @Post('login')
  @Throttle(5, 60) // 每分鐘 5 個請求
  async login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }
}
```

## 設定管理

### **環境設定**
- 使用 @nestjs/config 進行設定管理
- 在啟動時驗證設定
- 為不同的環境使用不同的設定

```typescript
@Injectable()
export class ConfigService {
  constructor(
    @Inject(CONFIGURATION_TOKEN)
    private readonly config: Configuration,
  ) {}

  get databaseUrl(): string {
    return this.config.database.url;
  }

  get jwtSecret(): string {
    return this.config.jwt.secret;
  }
}
```

## 常見的陷阱要避免

- **迴圈依賴：** 避免匯入會造成迴圈參考的模組
- **繁重的控制器：** 不要將業務邏輯放在控制器中
- **遺漏錯誤處理：** 始終適當處理錯誤
- **不適當的 DI 使用：** 在 DI 能夠處理時不要手動建立實例
- **遺漏驗證：** 始終驗證輸入資料
- **同步操作：** 對資料庫和外部 API 呼叫使用 async/await
- **記憶體洩漏：** 適當處理訂閱和事件監聽器

## 開發工作流程

### **開發設定**
1. 使用 NestJS CLI 進行專案搭建：`nest generate module users`
2. 遵循一致的檔案組織
3. 使用 TypeScript 嚴格模式
4. 使用 ESLint 實作全面的程式碼檢查
5. 使用 Prettier 進行程式碼格式化

### **程式碼審查檢查清單**
- [ ] 適當使用裝飾器和依賴注入
- [ ] 使用 DTO 和 class-validator 進行輸入驗證
- [ ] 適當的錯誤處理和異常過濾器
- [ ] 一致的命名規範
- [ ] 適當的模組組織和匯入
- [ ] 安全性考量（身份驗證、授權、輸入淨化）
- [ ] 效能考量（快取、資料庫最佳化）
- [ ] 全面的測試涵蓋

## 結論

NestJS 提供了一個強大、自有見解的框架來建立可擴展的 Node.js 應用程式。遵循這些最佳實踐，您可以建立可維護、可測試且高效的伺服器端應用程式，充分發揮 TypeScript 和現代開發模式的全部功能。

---

<!-- End of NestJS Instructions -->
