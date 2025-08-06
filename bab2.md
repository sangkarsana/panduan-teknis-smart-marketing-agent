# BAB 2
# ARSITEKTUR SISTEM

## 2.1 Overview Arsitektur

Smart Marketing Agent dibangun dengan prinsip modern web architecture yang mengutamakan scalability, maintainability, dan performance. Sistem ini mengadopsi serverless architecture dengan Next.js sebagai backbone, memungkinkan deployment yang efisien dan biaya operasional yang optimal untuk mendukung mitra UMKM.

### Mengapa Arsitektur Ini Dipilih?

**1. Cost-Effective untuk UMKM**
Dengan serverless architecture, biaya operasional hanya muncul saat sistem digunakan. Tidak ada biaya idle server, sangat cocok untuk UMKM dengan traffic yang fluktuatif.

**2. Scalable by Design**
Sistem dapat menangani 10 atau 10.000 mitra UMKM tanpa perlu redesign architecture. Auto-scaling handle traffic spike saat periode promo atau hari raya.

**3. Developer Friendly**
Tech stack yang populer memudahkan maintenance dan onboarding developer baru. Dokumentasi lengkap dan community support yang besar.

**4. Future Proof**
Arsitektur modular memungkinkan penambahan fitur baru tanpa mengganggu existing system. Integration dengan AI dan third-party services sangat mudah.

## 2.2 Tech Stack Decision

### Core Framework: Next.js 14
```typescript
// Why Next.js?
- Full-stack framework dengan App Router
- Built-in optimization (Image, Font, Script)
- Server Components untuk performance
- API Routes untuk backend logic
- Excellent SEO capabilities
- Strong TypeScript support
```

### Styling: Tailwind CSS + Radix UI
```css
/* Clean, consistent, dan responsive design */
- Utility-first CSS untuk rapid development
- Radix UI untuk accessible components
- Dark mode support out of the box
- Mobile-first approach
```

### Database: PostgreSQL dengan Supabase
```sql
-- Why Supabase?
- PostgreSQL power dengan REST API
- Real-time subscriptions
- Built-in authentication
- Row Level Security (RLS)
- Automatic backups
- Generous free tier untuk starting
```

### AI Integration: OpenAI GPT-4
```javascript
// Smart content generation
- Latest GPT-4 model untuk quality content
- Structured output dengan JSON mode
- Custom training untuk context batik
- Token optimization untuk cost control
```

### Additional Services
- **Vercel**: Hosting dengan global CDN
- **Resend**: Transactional email
- **Fonnte**: WhatsApp API untuk Indonesia
- **Sentry**: Error tracking dan monitoring

## 2.3 System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐  │
│  │              │     │              │     │              │  │
│  │  Web Browser │     │ Mobile Browser│     │  PWA App     │  │
│  │              │     │              │     │              │  │
│  └──────┬───────┘     └──────┬───────┘     └──────┬───────┘  │
│         │                    │                    │           │
│         └────────────────────┴────────────────────┘           │
│                              │                                 │
│                              ▼                                 │
│                   ┌──────────────────┐                       │
│                   │                  │                       │
│                   │  Vercel CDN      │                       │
│                   │  (Global Edge)   │                       │
│                   │                  │                       │
│                   └──────────────────┘                       │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                     APPLICATION LAYER                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                   Next.js 14 App                     │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │                                                     │  │
│  │  ┌───────────┐  ┌────────────┐  ┌──────────────┐ │  │
│  │  │  Pages &  │  │   API      │  │  Server      │ │  │
│  │  │  Layouts  │  │  Routes    │  │ Components   │ │  │
│  │  └───────────┘  └────────────┘  └──────────────┘ │  │
│  │                                                     │  │
│  │  ┌───────────────────────────────────────────────┐ │  │
│  │  │            Middleware Layer                   │ │  │
│  │  │  - Authentication  - Rate Limiting           │ │  │
│  │  │  - Authorization   - Request Validation      │ │  │
│  │  └───────────────────────────────────────────────┘ │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      SERVICE LAYER                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │
│  │   Auth      │  │   User      │  │   Customer      │   │
│  │  Service    │  │  Service    │  │   Service       │   │
│  └─────────────┘  └─────────────┘  └─────────────────┘   │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │
│  │Transaction  │  │    RFM      │  │    Content      │   │
│  │  Service    │  │  Analysis   │  │   Generation    │   │
│  └─────────────┘  └─────────────┘  └─────────────────┘   │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │
│  │Notification │  │  Report     │  │    Cache        │   │
│  │  Service    │  │  Service    │  │   Service       │   │
│  └─────────────┘  └─────────────┘  └─────────────────┘   │
│                                                             │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      DATA & EXTERNAL LAYER                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐  ┌─────────────┐  ┌──────────────┐  │
│  │   PostgreSQL    │  │  OpenAI     │  │   Resend     │  │
│  │   (Supabase)    │  │    API      │  │   Email      │  │
│  └─────────────────┘  └─────────────┘  └──────────────┘  │
│                                                             │
│  ┌─────────────────┐  ┌─────────────┐  ┌──────────────┐  │
│  │  File Storage   │  │   Fonnte    │  │   Sentry     │  │
│  │   (Supabase)    │  │  WhatsApp   │  │  Monitoring  │  │
│  └─────────────────┘  └─────────────┘  └──────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 2.4 Database Design

### Core Philosophy
Database design mengikuti prinsip normalization untuk data integrity, namun dengan strategic denormalization untuk query performance. Setiap tabel dirancang dengan future scaling in mind.

### Entity Relationship Diagram

```sql
-- Users Table (Mitra UMKM)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT UNIQUE NOT NULL,
    full_name TEXT NOT NULL,
    business_name TEXT NOT NULL,
    phone TEXT,
    role TEXT CHECK (role IN ('super_admin', 'umkm_owner')),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Customers Table
CREATE TABLE customers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    email TEXT,
    phone TEXT,
    address TEXT,
    city TEXT,
    tags TEXT[], -- Array untuk flexible tagging
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Index untuk performance
    INDEX idx_customers_user_id (user_id),
    INDEX idx_customers_phone (phone)
);

-- Transactions Table
CREATE TABLE transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    customer_id UUID REFERENCES customers(id) ON DELETE CASCADE,
    transaction_date DATE NOT NULL,
    amount DECIMAL(15,2) NOT NULL,
    products JSONB, -- Flexible product storage
    notes TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Index untuk RFM calculation
    INDEX idx_transactions_customer_date (customer_id, transaction_date DESC),
    INDEX idx_transactions_user_id (user_id)
);

-- RFM Analysis Results (Cached)
CREATE TABLE rfm_analysis (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    customer_id UUID REFERENCES customers(id) ON DELETE CASCADE,
    recency_score INTEGER CHECK (recency_score BETWEEN 1 AND 5),
    frequency_score INTEGER CHECK (frequency_score BETWEEN 1 AND 5),
    monetary_score INTEGER CHECK (monetary_score BETWEEN 1 AND 5),
    segment TEXT NOT NULL,
    analysis_date DATE NOT NULL,
    metrics JSONB, -- Store detailed metrics
    
    -- Unique constraint untuk prevent duplicate
    UNIQUE(user_id, customer_id, analysis_date),
    INDEX idx_rfm_segment (user_id, segment)
);

-- Generated Content Cache
CREATE TABLE content_cache (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    segment TEXT NOT NULL,
    content_type TEXT NOT NULL,
    prompt_hash TEXT NOT NULL,
    content TEXT NOT NULL,
    metadata JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ,
    
    -- Unique constraint untuk cache key
    UNIQUE(user_id, segment, content_type, prompt_hash)
);
```

### Row Level Security (RLS)

```sql
-- Enable RLS
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE transactions ENABLE ROW LEVEL SECURITY;
ALTER TABLE rfm_analysis ENABLE ROW LEVEL SECURITY;

-- Policies untuk UMKM Owner
CREATE POLICY "Users can view own customers" ON customers
    FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can manage own transactions" ON transactions
    FOR ALL USING (auth.uid() = user_id);

-- Super Admin dapat akses semua data
CREATE POLICY "Super admin full access" ON customers
    FOR ALL USING (
        EXISTS (
            SELECT 1 FROM users 
            WHERE id = auth.uid() 
            AND role = 'super_admin'
        )
    );
```

## 2.5 API Architecture

### RESTful API Design

API dirancang mengikuti REST principles dengan consistent naming convention dan predictable behavior.

```typescript
// API Route Structure
/api/
├── auth/
│   ├── login
│   ├── logout
│   ├── refresh
│   └── profile
├── customers/
│   ├── GET    /api/customers          // List with pagination
│   ├── POST   /api/customers          // Create new
│   ├── GET    /api/customers/:id      // Get detail
│   ├── PUT    /api/customers/:id      // Update
│   ├── DELETE /api/customers/:id      // Delete
│   └── POST   /api/customers/import   // Bulk import
├── transactions/
│   ├── GET    /api/transactions       // List with filters
│   ├── POST   /api/transactions       // Create new
│   └── POST   /api/transactions/bulk  // Bulk create
├── rfm/
│   ├── GET    /api/rfm/analysis       // Get latest analysis
│   ├── POST   /api/rfm/calculate      // Trigger calculation
│   └── GET    /api/rfm/segments/:name // Get segment details
└── content/
    ├── POST   /api/content/generate   // Generate new content
    └── GET    /api/content/templates  // Get saved templates
```

### Request/Response Format

```typescript
// Standard Response Format
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  meta?: {
    page?: number;
    limit?: number;
    total?: number;
  };
}

// Example Success Response
{
  "success": true,
  "data": {
    "customers": [...],
    "segments": {
      "champions": 15,
      "loyal_customers": 28,
      "at_risk": 12
    }
  },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 55
  }
}

// Example Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Data pelanggan tidak valid",
    "details": {
      "phone": "Format nomor telepon tidak sesuai"
    }
  }
}
```

### API Security Layers

```typescript
// Middleware Stack
export async function middleware(request: NextRequest) {
  // 1. CORS Headers
  const response = addCorsHeaders(request);
  
  // 2. Rate Limiting
  const rateLimitResult = await checkRateLimit(request);
  if (!rateLimitResult.allowed) {
    return new Response('Too Many Requests', { status: 429 });
  }
  
  // 3. Authentication
  const token = await verifyAuth(request);
  if (!token) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  // 4. Authorization
  const hasPermission = await checkPermission(token, request);
  if (!hasPermission) {
    return new Response('Forbidden', { status: 403 });
  }
  
  // 5. Request Validation
  const isValid = await validateRequest(request);
  if (!isValid) {
    return new Response('Bad Request', { status: 400 });
  }
  
  return response;
}
```

## 2.6 Service Layer Architecture

### Service Pattern Implementation

```typescript
// Base Service Class
abstract class BaseService {
  protected supabase: SupabaseClient;
  
  constructor() {
    this.supabase = createServerClient();
  }
  
  protected async executeQuery<T>(
    query: PostgrestQueryBuilder<T>
  ): Promise<T[]> {
    const { data, error } = await query;
    if (error) throw new DatabaseError(error.message);
    return data;
  }
  
  protected handleError(error: unknown): never {
    if (error instanceof AppError) throw error;
    throw new UnexpectedError('Internal server error');
  }
}

// Customer Service Example
export class CustomerService extends BaseService {
  async findAll(userId: string, filters?: CustomerFilters) {
    try {
      let query = this.supabase
        .from('customers')
        .select('*')
        .eq('user_id', userId);
        
      if (filters?.search) {
        query = query.or(`name.ilike.%${filters.search}%,phone.ilike.%${filters.search}%`);
      }
      
      if (filters?.tags?.length) {
        query = query.contains('tags', filters.tags);
      }
      
      return await this.executeQuery(query);
    } catch (error) {
      this.handleError(error);
    }
  }
  
  async create(userId: string, data: CreateCustomerDto) {
    try {
      // Validation
      const validated = customerSchema.parse(data);
      
      // Check duplicate
      const existing = await this.findByPhone(userId, validated.phone);
      if (existing) {
        throw new ValidationError('Customer with this phone already exists');
      }
      
      // Insert
      const { data: customer } = await this.supabase
        .from('customers')
        .insert({ ...validated, user_id: userId })
        .select()
        .single();
        
      return customer;
    } catch (error) {
      this.handleError(error);
    }
  }
}
```

### RFM Analysis Service

```typescript
export class RFMAnalysisService extends BaseService {
  private readonly RECENCY_BINS = [30, 60, 90, 180]; // days
  private readonly FREQUENCY_BINS = [1, 2, 5, 10]; // transactions
  private readonly MONETARY_BINS = [100000, 500000, 1000000, 5000000]; // IDR
  
  async calculateRFM(userId: string): Promise<RFMResult[]> {
    // Get all transactions for user
    const transactions = await this.getTransactions(userId);
    
    // Group by customer
    const customerTransactions = this.groupByCustomer(transactions);
    
    // Calculate RFM for each customer
    const rfmResults = [];
    for (const [customerId, txns] of customerTransactions) {
      const rfm = this.calculateCustomerRFM(customerId, txns);
      const segment = this.determineSegment(rfm);
      
      rfmResults.push({
        customerId,
        ...rfm,
        segment
      });
    }
    
    // Cache results
    await this.cacheResults(userId, rfmResults);
    
    return rfmResults;
  }
  
  private calculateCustomerRFM(customerId: string, transactions: Transaction[]) {
    const now = new Date();
    const sortedTxns = transactions.sort((a, b) => 
      b.transaction_date.getTime() - a.transaction_date.getTime()
    );
    
    // Recency: Days since last transaction
    const lastTxn = sortedTxns[0];
    const daysSinceLastTxn = Math.floor(
      (now.getTime() - lastTxn.transaction_date.getTime()) / (1000 * 60 * 60 * 24)
    );
    
    // Frequency: Total number of transactions
    const frequency = transactions.length;
    
    // Monetary: Total transaction value
    const monetary = transactions.reduce((sum, txn) => sum + txn.amount, 0);
    
    return {
      recency: daysSinceLastTxn,
      recencyScore: this.getScore(daysSinceLastTxn, this.RECENCY_BINS, true),
      frequency: frequency,
      frequencyScore: this.getScore(frequency, this.FREQUENCY_BINS),
      monetary: monetary,
      monetaryScore: this.getScore(monetary, this.MONETARY_BINS)
    };
  }
  
  private determineSegment(rfm: RFMScores): string {
    const { recencyScore: r, frequencyScore: f, monetaryScore: m } = rfm;
    
    if (r >= 4 && f >= 4 && m >= 4) return 'champions';
    if (r >= 3 && f >= 3 && m >= 3) return 'loyal_customers';
    if (r >= 3 && f >= 1 && m >= 1) return 'potential_loyalists';
    if (r >= 4 && f <= 2) return 'new_customers';
    if (r <= 2 && f >= 3 && m >= 3) return 'at_risk';
    if (r <= 2 && f >= 4 && m >= 4) return 'cant_lose_them';
    if (r <= 2 && f <= 2) return 'lost';
    
    return 'others';
  }
}
```

## 2.7 Performance Optimization

### Caching Strategy

```typescript
// Multi-layer caching approach
class CacheService {
  private memoryCache = new Map<string, CacheEntry>();
  private readonly TTL = {
    RFM_ANALYSIS: 7 * 24 * 60 * 60 * 1000, // 7 days
    CONTENT: 30 * 24 * 60 * 60 * 1000,     // 30 days
    USER_DATA: 60 * 60 * 1000,             // 1 hour
  };
  
  async get<T>(key: string): Promise<T | null> {
    // L1: Memory cache
    const memoryResult = this.memoryCache.get(key);
    if (memoryResult && !this.isExpired(memoryResult)) {
      return memoryResult.data as T;
    }
    
    // L2: Database cache
    const dbResult = await this.getFromDatabase(key);
    if (dbResult && !this.isExpired(dbResult)) {
      // Populate memory cache
      this.memoryCache.set(key, dbResult);
      return dbResult.data as T;
    }
    
    return null;
  }
  
  async set(key: string, data: any, ttl?: number): Promise<void> {
    const entry: CacheEntry = {
      data,
      timestamp: Date.now(),
      ttl: ttl || this.getTTL(key)
    };
    
    // Set in both layers
    this.memoryCache.set(key, entry);
    await this.saveToDatabase(key, entry);
  }
}
```

### Query Optimization

```sql
-- Optimized RFM calculation query
WITH customer_stats AS (
  SELECT 
    customer_id,
    MAX(transaction_date) as last_transaction,
    COUNT(*) as transaction_count,
    SUM(amount) as total_amount
  FROM transactions
  WHERE user_id = $1
    AND transaction_date >= CURRENT_DATE - INTERVAL '1 year'
  GROUP BY customer_id
),
rfm_scores AS (
  SELECT
    cs.*,
    c.name,
    c.phone,
    EXTRACT(DAY FROM CURRENT_DATE - cs.last_transaction) as days_since_last,
    NTILE(5) OVER (ORDER BY cs.last_transaction DESC) as recency_score,
    NTILE(5) OVER (ORDER BY cs.transaction_count) as frequency_score,
    NTILE(5) OVER (ORDER BY cs.total_amount) as monetary_score
  FROM customer_stats cs
  JOIN customers c ON cs.customer_id = c.id
)
SELECT * FROM rfm_scores;
```

## 2.8 Monitoring & Observability

### Application Monitoring Stack

```typescript
// Sentry Integration
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Sentry.Integrations.Postgres(),
  ],
});

// Custom performance monitoring
export function trackPerformance(name: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function (...args: any[]) {
      const start = performance.now();
      const transaction = Sentry.startTransaction({ name });
      
      try {
        const result = await originalMethod.apply(this, args);
        const duration = performance.now() - start;
        
        // Log if slow
        if (duration > 1000) {
          console.warn(`Slow operation: ${name} took ${duration}ms`);
        }
        
        return result;
      } catch (error) {
        Sentry.captureException(error);
        throw error;
      } finally {
        transaction.finish();
      }
    };
    
    return descriptor;
  };
}
```

### Health Check Endpoints

```typescript
// System health monitoring
app.get('/api/health', async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    openai: await checkOpenAI(),
    storage: await checkStorage(),
  };
  
  const healthy = Object.values(checks).every(check => check.status === 'ok');
  
  res.status(healthy ? 200 : 503).json({
    status: healthy ? 'healthy' : 'unhealthy',
    timestamp: new Date().toISOString(),
    checks
  });
});
```

## 2.9 Deployment Architecture

### Production Deployment on Vercel

```yaml
# vercel.json
{
  "buildCommand": "pnpm build",
  "outputDirectory": ".next",
  "devCommand": "pnpm dev",
  "installCommand": "pnpm install",
  "framework": "nextjs",
  "regions": ["sin1"], # Singapore region untuk Indonesia
  "functions": {
    "api/rfm/calculate": {
      "maxDuration": 60
    },
    "api/content/generate": {
      "maxDuration": 30
    }
  },
  "crons": [
    {
      "path": "/api/cron/weekly-analysis",
      "schedule": "0 6 * * 1" # Every Monday 6 AM
    }
  ]
}
```

### Environment Configuration

```bash
# .env.production
# Database
DATABASE_URL=postgresql://[user]:[password]@[host]/[database]
DIRECT_URL=postgresql://[user]:[password]@[host]/[database]

# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://[project].supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=[anon-key]
SUPABASE_SERVICE_ROLE_KEY=[service-role-key]

# OpenAI
OPENAI_API_KEY=sk-[your-api-key]
OPENAI_MODEL=gpt-4-turbo-preview

# Email
RESEND_API_KEY=re_[your-api-key]
EMAIL_FROM=noreply@smartmarketingagent.com

# WhatsApp
FONNTE_TOKEN=[your-token]

# Monitoring
SENTRY_DSN=https://[your-dsn]@sentry.io/[project]

# Security
JWT_SECRET=[random-secret]
ENCRYPTION_KEY=[random-key]
```

## 2.10 Key Takeaways

Arsitektur Smart Marketing Agent dirancang dengan keseimbangan antara:

1. **Simplicity vs Power** - Mudah di-maintain namun tetap powerful
2. **Cost vs Performance** - Optimal untuk budget UMKM
3. **Flexibility vs Structure** - Terstruktur namun adaptable
4. **Security vs Usability** - Aman tanpa mengorbankan UX

Dengan fondasi arsitektur yang solid ini, Smart Marketing Agent siap melayani ribuan mitra UMKM dengan performa optimal dan biaya yang terjangkau.

---

*"Good architecture is not about using the latest technology, but choosing the right technology for the right problem."*
