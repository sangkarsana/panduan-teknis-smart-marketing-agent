# BAB 6
# IMPLEMENTASI TEKNIS

## 6.1 Development Environment Setup

Memulai development Smart Marketing Agent memerlukan setup environment yang tepat. Bab ini akan memandu Anda step-by-step dari zero hingga aplikasi running di local machine.

### Prerequisites & Tools

```bash
# Required tools dan minimum versions
- Node.js v18.17+ (LTS recommended)
- npm v9+ atau pnpm v8+ (preferred untuk speed)
- Git v2.30+
- VS Code atau IDE pilihan Anda
- PostgreSQL v14+ (atau gunakan Supabase cloud)

# Optional tapi recommended
- Docker Desktop (untuk consistent environment)
- Postman/Insomnia (untuk API testing)
- TablePlus/pgAdmin (untuk database management)
```

### Initial Project Setup

```bash
# 1. Create new Next.js project
npx create-next-app@latest smart-marketing-agent \
  --typescript \
  --tailwind \
  --app \
  --src-dir \
  --import-alias "@/*"

cd smart-marketing-agent

# 2. Install core dependencies
pnpm add @supabase/supabase-js @supabase/auth-helpers-nextjs
pnpm add openai resend
pnpm add @radix-ui/react-dialog @radix-ui/react-dropdown-menu
pnpm add @radix-ui/react-select @radix-ui/react-tabs
pnpm add lucide-react date-fns zod
pnpm add @tanstack/react-query @tanstack/react-table
pnpm add recharts react-hook-form
pnpm add class-variance-authority clsx tailwind-merge

# 3. Dev dependencies
pnpm add -D @types/node
pnpm add -D prettier eslint-config-prettier
pnpm add -D @typescript-eslint/eslint-plugin
```

### Project Structure

```bash
smart-marketing-agent/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/            # Auth group routes
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── (dashboard)/       # Protected routes
│   │   │   ├── layout.tsx
│   │   │   ├── dashboard/
│   │   │   ├── customers/
│   │   │   ├── transactions/
│   │   │   ├── analysis/
│   │   │   └── marketing/
│   │   ├── api/               # API routes
│   │   │   ├── auth/
│   │   │   ├── customers/
│   │   │   ├── rfm/
│   │   │   └── content/
│   │   ├── layout.tsx         # Root layout
│   │   └── page.tsx           # Landing page
│   ├── components/            # Reusable components
│   │   ├── ui/               # Base UI components
│   │   ├── forms/            # Form components
│   │   ├── charts/           # Chart components
│   │   └── layouts/          # Layout components
│   ├── lib/                   # Utilities & configs
│   │   ├── supabase/         # Supabase client
│   │   ├── openai/           # OpenAI config
│   │   └── utils/            # Helper functions
│   ├── hooks/                 # Custom React hooks
│   ├── services/              # Business logic
│   ├── types/                 # TypeScript types
│   └── styles/                # Global styles
├── public/                    # Static assets
├── prisma/                    # Database schema (optional)
├── .env.local                 # Environment variables
├── next.config.js             # Next.js config
├── tailwind.config.ts         # Tailwind config
├── tsconfig.json              # TypeScript config
└── package.json
```

### Environment Configuration

```bash
# .env.local
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# OpenAI
OPENAI_API_KEY=sk-your-api-key
OPENAI_ORG_ID=org-your-org-id # optional

# Email (Resend)
RESEND_API_KEY=re_your-api-key
EMAIL_FROM=noreply@yourdomain.com

# WhatsApp (Fonnte)
FONNTE_TOKEN=your-fonnte-token
FONNTE_DEVICE=your-device-id

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Analytics & Monitoring
NEXT_PUBLIC_POSTHOG_KEY=your-posthog-key
SENTRY_DSN=your-sentry-dsn
```

## 6.2 Database Setup with Supabase

### Creating Supabase Project

```typescript
// lib/supabase/migrations/001_initial_schema.sql

-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Create custom types
CREATE TYPE user_role AS ENUM ('super_admin', 'umkm_owner');
CREATE TYPE customer_segment AS ENUM (
  'champions',
  'loyal_customers', 
  'potential_loyalists',
  'new_customers',
  'promising',
  'need_attention',
  'about_to_sleep',
  'at_risk',
  'cant_lose_them',
  'lost'
);

-- Users table (extends Supabase auth.users)
CREATE TABLE public.profiles (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  business_name TEXT,
  phone TEXT,
  role user_role DEFAULT 'umkm_owner',
  is_active BOOLEAN DEFAULT true,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Customers table
CREATE TABLE public.customers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  email TEXT,
  phone TEXT,
  address TEXT,
  city TEXT,
  tags TEXT[] DEFAULT '{}',
  metadata JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  -- Indexes
  CONSTRAINT unique_user_phone UNIQUE(user_id, phone)
);

CREATE INDEX idx_customers_user_id ON customers(user_id);
CREATE INDEX idx_customers_phone ON customers(phone);
CREATE INDEX idx_customers_tags ON customers USING GIN(tags);

-- Transactions table
CREATE TABLE public.transactions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  customer_id UUID REFERENCES public.customers(id) ON DELETE CASCADE,
  transaction_date DATE NOT NULL,
  amount DECIMAL(15,2) NOT NULL CHECK (amount > 0),
  products JSONB DEFAULT '[]',
  notes TEXT,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  
  -- Indexes for RFM calculation
  CONSTRAINT positive_amount CHECK (amount > 0)
);

CREATE INDEX idx_transactions_user_id ON transactions(user_id);
CREATE INDEX idx_transactions_customer_date ON transactions(customer_id, transaction_date DESC);
CREATE INDEX idx_transactions_date ON transactions(transaction_date);

-- RFM Analysis Results
CREATE TABLE public.rfm_analysis (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  analysis_date DATE NOT NULL,
  customer_id UUID REFERENCES public.customers(id) ON DELETE CASCADE,
  
  -- RFM Scores
  recency_days INTEGER NOT NULL,
  recency_score INTEGER CHECK (recency_score BETWEEN 1 AND 5),
  frequency_count INTEGER NOT NULL,
  frequency_score INTEGER CHECK (frequency_score BETWEEN 1 AND 5),
  monetary_value DECIMAL(15,2) NOT NULL,
  monetary_score INTEGER CHECK (monetary_score BETWEEN 1 AND 5),
  
  -- Segment
  segment customer_segment NOT NULL,
  
  -- Additional metrics
  metrics JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  
  -- Ensure one analysis per customer per date
  CONSTRAINT unique_analysis UNIQUE(user_id, customer_id, analysis_date)
);

CREATE INDEX idx_rfm_user_date ON rfm_analysis(user_id, analysis_date DESC);
CREATE INDEX idx_rfm_segment ON rfm_analysis(user_id, segment);

-- Generated Content
CREATE TABLE public.generated_content (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  segment customer_segment,
  content_type TEXT NOT NULL,
  channel TEXT NOT NULL,
  
  -- Content
  content TEXT NOT NULL,
  metadata JSONB DEFAULT '{}',
  
  -- AI Details
  model_used TEXT,
  prompt_tokens INTEGER,
  completion_tokens INTEGER,
  total_tokens INTEGER,
  
  -- Status
  is_approved BOOLEAN DEFAULT false,
  is_sent BOOLEAN DEFAULT false,
  sent_at TIMESTAMPTZ,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ
);

CREATE INDEX idx_content_user_segment ON generated_content(user_id, segment);
CREATE INDEX idx_content_created ON generated_content(created_at DESC);

-- Campaign Performance
CREATE TABLE public.campaign_performance (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  content_id UUID REFERENCES public.generated_content(id) ON DELETE CASCADE,
  
  -- Metrics
  sent_count INTEGER DEFAULT 0,
  delivered_count INTEGER DEFAULT 0,
  open_count INTEGER DEFAULT 0,
  click_count INTEGER DEFAULT 0,
  conversion_count INTEGER DEFAULT 0,
  
  -- Rates
  delivery_rate DECIMAL(5,2),
  open_rate DECIMAL(5,2),
  click_rate DECIMAL(5,2),
  conversion_rate DECIMAL(5,2),
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE transactions ENABLE ROW LEVEL SECURITY;
ALTER TABLE rfm_analysis ENABLE ROW LEVEL SECURITY;
ALTER TABLE generated_content ENABLE ROW LEVEL SECURITY;
ALTER TABLE campaign_performance ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can view own profile" ON profiles
  FOR ALL USING (auth.uid() = id);

CREATE POLICY "Users can manage own customers" ON customers
  FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can manage own transactions" ON transactions
  FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can view own analysis" ON rfm_analysis
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can manage own content" ON generated_content
  FOR ALL USING (auth.uid() = user_id);

-- Functions
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Triggers
CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_customers_updated_at
  BEFORE UPDATE ON customers
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();
```

### Supabase Client Configuration

```typescript
// lib/supabase/client.ts
import { createBrowserClient } from '@supabase/ssr'
import type { Database } from '@/types/database'

export function createClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}

// lib/supabase/server.ts
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/types/database'

export function createServerClientInstance() {
  const cookieStore = cookies()

  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value, ...options })
          } catch (error) {
            // Handle cookie errors in Server Components
          }
        },
        remove(name: string, options: CookieOptions) {
          try {
            cookieStore.delete(name)
          } catch (error) {
            // Handle cookie errors in Server Components
          }
        },
      },
    }
  )
}

// lib/supabase/middleware.ts
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function updateSession(request: NextRequest) {
  let response = NextResponse.next({
    request: {
      headers: request.headers,
    },
  })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return request.cookies.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          request.cookies.set({ name, value, ...options })
          response = NextResponse.next({
            request: {
              headers: request.headers,
            },
          })
          response.cookies.set({ name, value, ...options })
        },
        remove(name: string, options: CookieOptions) {
          request.cookies.set({ name, value: '', ...options })
          response = NextResponse.next({
            request: {
              headers: request.headers,
            },
          })
          response.cookies.set({ name, value: '', ...options })
        },
      },
    }
  )

  await supabase.auth.getUser()

  return response
}
```

## 6.3 Authentication Implementation

### Auth Flow with Supabase

```typescript
// app/api/auth/register/route.ts
import { createServerClientInstance } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'
import { z } from 'zod'

const registerSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  fullName: z.string().min(2),
  businessName: z.string().min(2),
  phone: z.string().min(10)
})

export async function POST(request: Request) {
  try {
    const body = await request.json()
    const validatedData = registerSchema.parse(body)
    
    const supabase = createServerClientInstance()
    
    // 1. Create auth user
    const { data: authData, error: authError } = await supabase.auth.signUp({
      email: validatedData.email,
      password: validatedData.password,
      options: {
        data: {
          full_name: validatedData.fullName,
          business_name: validatedData.businessName
        }
      }
    })
    
    if (authError) throw authError
    
    // 2. Create profile
    if (authData.user) {
      const { error: profileError } = await supabase
        .from('profiles')
        .insert({
          id: authData.user.id,
          email: validatedData.email,
          full_name: validatedData.fullName,
          business_name: validatedData.businessName,
          phone: validatedData.phone
        })
      
      if (profileError) throw profileError
    }
    
    return NextResponse.json({
      success: true,
      message: 'Registration successful. Please check your email to verify.'
    })
    
  } catch (error) {
    console.error('Registration error:', error)
    
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { success: false, errors: error.errors },
        { status: 400 }
      )
    }
    
    return NextResponse.json(
      { success: false, message: 'Registration failed' },
      { status: 500 }
    )
  }
}

// app/api/auth/login/route.ts
export async function POST(request: Request) {
  try {
    const { email, password } = await request.json()
    const supabase = createServerClientInstance()
    
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password
    })
    
    if (error) throw error
    
    // Get user profile
    const { data: profile } = await supabase
      .from('profiles')
      .select('*')
      .eq('id', data.user.id)
      .single()
    
    return NextResponse.json({
      success: true,
      user: {
        id: data.user.id,
        email: data.user.email,
        profile
      }
    })
    
  } catch (error) {
    return NextResponse.json(
      { success: false, message: 'Invalid credentials' },
      { status: 401 }
    )
  }
}
```

### Protected Route Middleware

```typescript
// middleware.ts
import { type NextRequest } from 'next/server'
import { updateSession } from '@/lib/supabase/middleware'

export async function middleware(request: NextRequest) {
  // Update session
  const response = await updateSession(request)
  
  // Check if user is authenticated
  const isAuthRoute = request.nextUrl.pathname.startsWith('/login') || 
                     request.nextUrl.pathname.startsWith('/register')
  const isProtectedRoute = request.nextUrl.pathname.startsWith('/dashboard') ||
                          request.nextUrl.pathname.startsWith('/customers') ||
                          request.nextUrl.pathname.startsWith('/analysis')
  
  // Redirect logic
  const user = response.headers.get('x-user-id')
  
  if (!user && isProtectedRoute) {
    return Response.redirect(new URL('/login', request.url))
  }
  
  if (user && isAuthRoute) {
    return Response.redirect(new URL('/dashboard', request.url))
  }
  
  return response
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

## 6.4 Core Services Implementation

### Customer Service

```typescript
// services/customer.service.ts
import { createServerClientInstance } from '@/lib/supabase/server'
import { z } from 'zod'
import type { Customer, CreateCustomerDto, UpdateCustomerDto } from '@/types'

export class CustomerService {
  private supabase = createServerClientInstance()
  
  async findAll(userId: string, options?: {
    search?: string
    segment?: string
    page?: number
    limit?: number
    sortBy?: string
    sortOrder?: 'asc' | 'desc'
  }) {
    const { 
      search = '', 
      segment = 'all', 
      page = 1, 
      limit = 20,
      sortBy = 'created_at',
      sortOrder = 'desc'
    } = options || {}
    
    let query = this.supabase
      .from('customers')
      .select(`
        *,
        rfm_analysis!inner (
          segment,
          recency_score,
          frequency_score,
          monetary_score,
          analysis_date
        )
      `, { count: 'exact' })
      .eq('user_id', userId)
      .eq('is_active', true)
    
    // Search filter
    if (search) {
      query = query.or(`name.ilike.%${search}%,phone.ilike.%${search}%,email.ilike.%${search}%`)
    }
    
    // Segment filter
    if (segment !== 'all') {
      query = query.eq('rfm_analysis.segment', segment)
    }
    
    // Sorting
    query = query.order(sortBy, { ascending: sortOrder === 'asc' })
    
    // Pagination
    const from = (page - 1) * limit
    const to = from + limit - 1
    query = query.range(from, to)
    
    const { data, error, count } = await query
    
    if (error) throw error
    
    return {
      customers: data || [],
      pagination: {
        page,
        limit,
        total: count || 0,
        totalPages: Math.ceil((count || 0) / limit)
      }
    }
  }
  
  async findById(userId: string, customerId: string) {
    const { data, error } = await this.supabase
      .from('customers')
      .select(`
        *,
        transactions (
          id,
          transaction_date,
          amount,
          products
        ),
        rfm_analysis (
          segment,
          recency_score,
          frequency_score,
          monetary_score,
          analysis_date
        )
      `)
      .eq('id', customerId)
      .eq('user_id', userId)
      .single()
    
    if (error) throw error
    return data
  }
  
  async create(userId: string, dto: CreateCustomerDto) {
    // Validate input
    const schema = z.object({
      name: z.string().min(2),
      phone: z.string().min(10),
      email: z.string().email().optional(),
      address: z.string().optional(),
      city: z.string().optional(),
      tags: z.array(z.string()).optional()
    })
    
    const validated = schema.parse(dto)
    
    // Check for duplicate phone
    const { data: existing } = await this.supabase
      .from('customers')
      .select('id')
      .eq('user_id', userId)
      .eq('phone', validated.phone)
      .single()
    
    if (existing) {
      throw new Error('Customer with this phone number already exists')
    }
    
    // Create customer
    const { data, error } = await this.supabase
      .from('customers')
      .insert({
        ...validated,
        user_id: userId
      })
      .select()
      .single()
    
    if (error) throw error
    return data
  }
  
  async update(userId: string, customerId: string, dto: UpdateCustomerDto) {
    const { data, error } = await this.supabase
      .from('customers')
      .update(dto)
      .eq('id', customerId)
      .eq('user_id', userId)
      .select()
      .single()
    
    if (error) throw error
    return data
  }
  
  async delete(userId: string, customerId: string) {
    // Soft delete
    const { error } = await this.supabase
      .from('customers')
      .update({ is_active: false })
      .eq('id', customerId)
      .eq('user_id', userId)
    
    if (error) throw error
    return { success: true }
  }
  
  async importBulk(userId: string, customers: CreateCustomerDto[]) {
    // Validate all customers
    const schema = z.array(z.object({
      name: z.string().min(2),
      phone: z.string().min(10),
      email: z.string().email().optional(),
      address: z.string().optional(),
      city: z.string().optional(),
      tags: z.array(z.string()).optional()
    }))
    
    const validated = schema.parse(customers)
    
    // Get existing phones to avoid duplicates
    const phones = validated.map(c => c.phone)
    const { data: existing } = await this.supabase
      .from('customers')
      .select('phone')
      .eq('user_id', userId)
      .in('phone', phones)
    
    const existingPhones = new Set(existing?.map(e => e.phone) || [])
    
    // Filter out duplicates
    const newCustomers = validated.filter(c => !existingPhones.has(c.phone))
    
    if (newCustomers.length === 0) {
      return {
        success: true,
        imported: 0,
        skipped: customers.length,
        message: 'All customers already exist'
      }
    }
    
    // Insert in batches
    const batchSize = 100
    let imported = 0
    
    for (let i = 0; i < newCustomers.length; i += batchSize) {
      const batch = newCustomers.slice(i, i + batchSize)
      const { error } = await this.supabase
        .from('customers')
        .insert(batch.map(c => ({ ...c, user_id: userId })))
      
      if (error) throw error
      imported += batch.length
    }
    
    return {
      success: true,
      imported,
      skipped: customers.length - imported,
      message: `Successfully imported ${imported} customers`
    }
  }
}
```

### Transaction Service

```typescript
// services/transaction.service.ts
export class TransactionService {
  private supabase = createServerClientInstance()
  
  async create(userId: string, dto: CreateTransactionDto) {
    const schema = z.object({
      customerId: z.string().uuid(),
      transactionDate: z.string().datetime(),
      amount: z.number().positive(),
      products: z.array(z.object({
        name: z.string(),
        quantity: z.number().positive(),
        price: z.number().positive()
      })).optional(),
      notes: z.string().optional()
    })
    
    const validated = schema.parse(dto)
    
    // Verify customer belongs to user
    const { data: customer } = await this.supabase
      .from('customers')
      .select('id')
      .eq('id', validated.customerId)
      .eq('user_id', userId)
      .single()
    
    if (!customer) {
      throw new Error('Customer not found')
    }
    
    // Create transaction
    const { data, error } = await this.supabase
      .from('transactions')
      .insert({
        user_id: userId,
        customer_id: validated.customerId,
        transaction_date: validated.transactionDate,
        amount: validated.amount,
        products: validated.products || [],
        notes: validated.notes
      })
      .select()
      .single()
    
    if (error) throw error
    
    // Trigger RFM recalculation for this customer
    await this.triggerRFMUpdate(userId, validated.customerId)
    
    return data
  }
  
  async findAll(userId: string, options?: {
    customerId?: string
    dateFrom?: string
    dateTo?: string
    page?: number
    limit?: number
  }) {
    const { 
      customerId, 
      dateFrom, 
      dateTo, 
      page = 1, 
      limit = 50 
    } = options || {}
    
    let query = this.supabase
      .from('transactions')
      .select(`
        *,
        customers!inner (
          id,
          name,
          phone
        )
      `, { count: 'exact' })
      .eq('user_id', userId)
    
    if (customerId) {
      query = query.eq('customer_id', customerId)
    }
    
    if (dateFrom) {
      query = query.gte('transaction_date', dateFrom)
    }
    
    if (dateTo) {
      query = query.lte('transaction_date', dateTo)
    }
    
    // Sorting
    query = query.order('transaction_date', { ascending: false })
    
    // Pagination
    const from = (page - 1) * limit
    const to = from + limit - 1
    query = query.range(from, to)
    
    const { data, error, count } = await query
    
    if (error) throw error
    
    return {
      transactions: data || [],
      pagination: {
        page,
        limit,
        total: count || 0,
        totalPages: Math.ceil((count || 0) / limit)
      }
    }
  }
  
  async getStats(userId: string, period: 'day' | 'week' | 'month' | 'year' = 'month') {
    const now = new Date()
    let startDate: Date
    
    switch (period) {
      case 'day':
        startDate = new Date(now.setHours(0, 0, 0, 0))
        break
      case 'week':
        startDate = new Date(now.setDate(now.getDate() - 7))
        break
      case 'month':
        startDate = new Date(now.setMonth(now.getMonth() - 1))
        break
      case 'year':
        startDate = new Date(now.setFullYear(now.getFullYear() - 1))
        break
    }
    
    const { data, error } = await this.supabase
      .from('transactions')
      .select('amount, transaction_date')
      .eq('user_id', userId)
      .gte('transaction_date', startDate.toISOString())
    
    if (error) throw error
    
    const stats = {
      totalRevenue: data?.reduce((sum, t) => sum + Number(t.amount), 0) || 0,
      transactionCount: data?.length || 0,
      averageOrderValue: 0,
      dailyRevenue: {} as Record<string, number>
    }
    
    if (stats.transactionCount > 0) {
      stats.averageOrderValue = stats.totalRevenue / stats.transactionCount
    }
    
    // Group by day
    data?.forEach(transaction => {
      const date = new Date(transaction.transaction_date).toISOString().split('T')[0]
      stats.dailyRevenue[date] = (stats.dailyRevenue[date] || 0) + Number(transaction.amount)
    })
    
    return stats
  }
  
  private async triggerRFMUpdate(userId: string, customerId: string) {
    // Queue RFM calculation for this customer
    // In production, this would be a job queue
    // For now, we'll calculate directly
    
    try {
      const rfmService = new RFMService()
      await rfmService.calculateForCustomer(userId, customerId)
    } catch (error) {
      console.error('Failed to update RFM:', error)
      // Don't throw - RFM update failure shouldn't block transaction
    }
  }
}
```

### RFM Analysis Service

```typescript
// services/rfm.service.ts
export class RFMService {
  private supabase = createServerClientInstance()
  
  async calculateForAllCustomers(userId: string) {
    console.log(`Starting RFM calculation for user ${userId}`)
    
    // Get all transactions for the user in the last year
    const oneYearAgo = new Date()
    oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1)
    
    const { data: transactions, error } = await this.supabase
      .from('transactions')
      .select(`
        customer_id,
        transaction_date,
        amount
      `)
      .eq('user_id', userId)
      .gte('transaction_date', oneYearAgo.toISOString())
      .order('transaction_date', { ascending: false })
    
    if (error) throw error
    if (!transactions || transactions.length === 0) {
      console.log('No transactions found')
      return []
    }
    
    // Group transactions by customer
    const customerMetrics = this.calculateCustomerMetrics(transactions)
    
    // Calculate RFM scores
    const rfmScores = this.calculateRFMScores(customerMetrics)
    
    // Determine segments
    const segments = rfmScores.map(score => ({
      ...score,
      segment: this.determineSegment(score)
    }))
    
    // Save to database
    await this.saveAnalysisResults(userId, segments)
    
    return segments
  }
  
  async calculateForCustomer(userId: string, customerId: string) {
    const { data: transactions, error } = await this.supabase
      .from('transactions')
      .select('transaction_date, amount')
      .eq('user_id', userId)
      .eq('customer_id', customerId)
      .order('transaction_date', { ascending: false })
    
    if (error) throw error
    if (!transactions || transactions.length === 0) return null
    
    const metrics = this.calculateSingleCustomerMetrics(customerId, transactions)
    const scores = this.calculateSingleRFMScore(metrics)
    const segment = this.determineSegment(scores)
    
    // Save to database
    await this.saveSingleAnalysisResult(userId, {
      ...scores,
      customerId,
      segment
    })
    
    return { ...scores, segment }
  }
  
  private calculateCustomerMetrics(transactions: any[]) {
    const metricsMap = new Map<string, {
      customerId: string
      lastTransactionDate: Date
      transactionCount: number
      totalAmount: number
    }>()
    
    transactions.forEach(transaction => {
      const existing = metricsMap.get(transaction.customer_id) || {
        customerId: transaction.customer_id,
        lastTransactionDate: new Date(0),
        transactionCount: 0,
        totalAmount: 0
      }
      
      const transactionDate = new Date(transaction.transaction_date)
      
      if (transactionDate > existing.lastTransactionDate) {
        existing.lastTransactionDate = transactionDate
      }
      
      existing.transactionCount++
      existing.totalAmount += Number(transaction.amount)
      
      metricsMap.set(transaction.customer_id, existing)
    })
    
    return Array.from(metricsMap.values())
  }
  
  private calculateRFMScores(customerMetrics: any[]) {
    const now = new Date()
    
    // Calculate raw values
    const rawValues = customerMetrics.map(metric => {
      const daysSinceLastTransaction = Math.floor(
        (now.getTime() - metric.lastTransactionDate.getTime()) / (1000 * 60 * 60 * 24)
      )
      
      return {
        customerId: metric.customerId,
        recency: daysSinceLastTransaction,
        frequency: metric.transactionCount,
        monetary: metric.totalAmount
      }
    })
    
    // Sort for quintile calculation
    const recencyValues = [...rawValues].sort((a, b) => a.recency - b.recency)
    const frequencyValues = [...rawValues].sort((a, b) => b.frequency - a.frequency)
    const monetaryValues = [...rawValues].sort((a, b) => b.monetary - a.monetary)
    
    // Calculate quintile breakpoints
    const getQuintileBreakpoints = (sortedValues: any[], field: string) => {
      const n = sortedValues.length
      const breakpoints = []
      
      for (let i = 1; i < 5; i++) {
        const index = Math.floor((n * i) / 5)
        breakpoints.push(sortedValues[index][field])
      }
      
      return breakpoints
    }
    
    const recencyBreaks = getQuintileBreakpoints(recencyValues, 'recency')
    const frequencyBreaks = getQuintileBreakpoints(frequencyValues, 'frequency')
    const monetaryBreaks = getQuintileBreakpoints(monetaryValues, 'monetary')
    
    // Assign scores
    return rawValues.map(value => {
      const recencyScore = this.getScore(value.recency, recencyBreaks, true) // inverse for recency
      const frequencyScore = this.getScore(value.frequency, frequencyBreaks)
      const monetaryScore = this.getScore(value.monetary, monetaryBreaks)
      
      return {
        customerId: value.customerId,
        recencyDays: value.recency,
        recencyScore,
        frequencyCount: value.frequency,
        frequencyScore,
        monetaryValue: value.monetary,
        monetaryScore
      }
    })
  }
  
  private getScore(value: number, breakpoints: number[], inverse = false): number {
    if (inverse) {
      // For recency: lower is better
      for (let i = 0; i < breakpoints.length; i++) {
        if (value <= breakpoints[i]) {
          return 5 - i
        }
      }
      return 1
    } else {
      // For frequency and monetary: higher is better
      for (let i = 0; i < breakpoints.length; i++) {
        if (value >= breakpoints[i]) {
          return 5 - i
        }
      }
      return 1
    }
  }
  
  private determineSegment(scores: {
    recencyScore: number
    frequencyScore: number
    monetaryScore: number
  }): string {
    const { recencyScore: r, frequencyScore: f, monetaryScore: m } = scores
    
    if (r >= 4 && f >= 4 && m >= 4) return 'champions'
    if (r >= 3 && f >= 3 && m >= 3) return 'loyal_customers'
    if (r >= 4 && f >= 2 && m >= 3) return 'potential_loyalists'
    if (r >= 4 && f <= 2) return 'new_customers'
    if (r >= 3 && f <= 2) return 'promising'
    if (r === 3 && f >= 3) return 'need_attention'
    if (r === 2 && f >= 2) return 'about_to_sleep'
    if (r <= 2 && f >= 3 && m >= 3) return 'at_risk'
    if (r <= 2 && f >= 4 && m >= 4) return 'cant_lose_them'
    if (r <= 2 && f <= 2) return 'lost'
    
    return 'need_attention' // default fallback
  }
  
  private async saveAnalysisResults(userId: string, results: any[]) {
    const analysisDate = new Date().toISOString().split('T')[0]
    
    // Prepare batch insert
    const records = results.map(result => ({
      user_id: userId,
      customer_id: result.customerId,
      analysis_date: analysisDate,
      recency_days: result.recencyDays,
      recency_score: result.recencyScore,
      frequency_count: result.frequencyCount,
      frequency_score: result.frequencyScore,
      monetary_value: result.monetaryValue,
      monetary_score: result.monetaryScore,
      segment: result.segment,
      metrics: {
        rfm_string: `${result.recencyScore}${result.frequencyScore}${result.monetaryScore}`
      }
    }))
    
    // Insert in batches
    const batchSize = 100
    for (let i = 0; i < records.length; i += batchSize) {
      const batch = records.slice(i, i + batchSize)
      
      const { error } = await this.supabase
        .from('rfm_analysis')
        .upsert(batch, {
          onConflict: 'user_id,customer_id,analysis_date'
        })
      
      if (error) throw error
    }
    
    console.log(`Saved ${records.length} RFM analysis results`)
  }
}
```

## 6.5 API Routes Implementation

### Customer API Routes

```typescript
// app/api/customers/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { CustomerService } from '@/services/customer.service'
import { withAuth } from '@/lib/auth/middleware'

const customerService = new CustomerService()

export const GET = withAuth(async (request: NextRequest, { user }) => {
  try {
    const searchParams = request.nextUrl.searchParams
    const options = {
      search: searchParams.get('search') || undefined,
      segment: searchParams.get('segment') || undefined,
      page: Number(searchParams.get('page')) || 1,
      limit: Number(searchParams.get('limit')) || 20,
      sortBy: searchParams.get('sortBy') || 'created_at',
      sortOrder: (searchParams.get('sortOrder') || 'desc') as 'asc' | 'desc'
    }
    
    const result = await customerService.findAll(user.id, options)
    
    return NextResponse.json({
      success: true,
      data: result
    })
  } catch (error) {
    console.error('Get customers error:', error)
    return NextResponse.json(
      { success: false, message: 'Failed to fetch customers' },
      { status: 500 }
    )
  }
})

export const POST = withAuth(async (request: NextRequest, { user }) => {
  try {
    const body = await request.json()
    const customer = await customerService.create(user.id, body)
    
    return NextResponse.json({
      success: true,
      data: customer
    }, { status: 201 })
  } catch (error) {
    console.error('Create customer error:', error)
    return NextResponse.json(
      { success: false, message: error.message },
      { status: 400 }
    )
  }
})

// app/api/customers/[id]/route.ts
export const GET = withAuth(async (
  request: NextRequest,
  { params, user }: { params: { id: string }, user: any }
) => {
  try {
    const customer = await customerService.findById(user.id, params.id)
    
    if (!customer) {
      return NextResponse.json(
        { success: false, message: 'Customer not found' },
        { status: 404 }
      )
    }
    
    return NextResponse.json({
      success: true,
      data: customer
    })
  } catch (error) {
    console.error('Get customer error:', error)
    return NextResponse.json(
      { success: false, message: 'Failed to fetch customer' },
      { status: 500 }
    )
  }
})

export const PUT = withAuth(async (
  request: NextRequest,
  { params, user }: { params: { id: string }, user: any }
) => {
  try {
    const body = await request.json()
    const customer = await customerService.update(user.id, params.id, body)
    
    return NextResponse.json({
      success: true,
      data: customer
    })
  } catch (error) {
    console.error('Update customer error:', error)
    return NextResponse.json(
      { success: false, message: error.message },
      { status: 400 }
    )
  }
})

export const DELETE = withAuth(async (
  request: NextRequest,
  { params, user }: { params: { id: string }, user: any }
) => {
  try {
    await customerService.delete(user.id, params.id)
    
    return NextResponse.json({
      success: true,
      message: 'Customer deleted successfully'
    })
  } catch (error) {
    console.error('Delete customer error:', error)
    return NextResponse.json(
      { success: false, message: 'Failed to delete customer' },
      { status: 500 }
    )
  }
})
```

### RFM Analysis API Routes

```typescript
// app/api/rfm/analysis/route.ts
import { RFMService } from '@/services/rfm.service'

const rfmService = new RFMService()

export const GET = withAuth(async (request: NextRequest, { user }) => {
  try {
    // Get latest analysis from database
    const { data, error } = await supabase
      .from('rfm_analysis')
      .select(`
        *,
        customers!inner (
          id,
          name,
          phone,
          email
        )
      `)
      .eq('user_id', user.id)
      .eq('analysis_date', new Date().toISOString().split('T')[0])
      .order('monetary_value', { ascending: false })
    
    if (error) throw error
    
    // Group by segment
    const segmentDistribution = data?.reduce((acc, item) => {
      acc[item.segment] = (acc[item.segment] || 0) + 1
      return acc
    }, {} as Record<string, number>)
    
    return NextResponse.json({
      success: true,
      data: {
        results: data || [],
        segmentDistribution,
        totalCustomers: data?.length || 0,
        lastUpdated: new Date().toISOString()
      }
    })
  } catch (error) {
    console.error('Get RFM analysis error:', error)
    return NextResponse.json(
      { success: false, message: 'Failed to fetch analysis' },
      { status: 500 }
    )
  }
})

export const POST = withAuth(async (request: NextRequest, { user }) => {
  try {
    // Trigger new calculation
    const results = await rfmService.calculateForAllCustomers(user.id)
    
    return NextResponse.json({
      success: true,
      data: {
        message: 'RFM analysis completed',
        customersAnalyzed: results.length,
        timestamp: new Date().toISOString()
      }
    })
  } catch (error) {
    console.error('Calculate RFM error:', error)
    return NextResponse.json(
      { success: false, message: 'Failed to calculate RFM' },
      { status: 500 }
    )
  }
})

// app/api/rfm/segments/[segment]/route.ts
export const GET = withAuth(async (
  request: NextRequest,
  { params, user }: { params: { segment: string }, user: any }
) => {
  try {
    const { data, error } = await supabase
      .from('rfm_analysis')
      .select(`
        *,
        customers!inner (*)
      `)
      .eq('user_id', user.id)
      .eq('segment', params.segment)
      .eq('analysis_date', new Date().toISOString().split('T')[0])
      .order('monetary_value', { ascending: false })
    
    if (error) throw error
    
    // Get segment characteristics
    const segmentInfo = getSegmentInfo(params.segment)
    
    return NextResponse.json({
      success: true,
      data: {
        segment: params.segment,
        info: segmentInfo,
        customers: data || [],
        count: data?.length || 0
      }
    })
  } catch (error) {
    console.error('Get segment details error:', error)
    return NextResponse.json(
      { success: false, message: 'Failed to fetch segment details' },
      { status: 500 }
    )
  }
})
```

## 6.6 Testing Strategy

### Unit Tests

```typescript
// __tests__/services/customer.service.test.ts
import { CustomerService } from '@/services/customer.service'
import { createMockSupabaseClient } from '@/tests/mocks/supabase'

describe('CustomerService', () => {
  let service: CustomerService
  let mockSupabase: any
  
  beforeEach(() => {
    mockSupabase = createMockSupabaseClient()
    service = new CustomerService(mockSupabase)
  })
  
  describe('create', () => {
    it('should create a new customer', async () => {
      const mockCustomer = {
        id: 'test-id',
        name: 'Test Customer',
        phone: '08123456789',
        email: 'test@example.com'
      }
      
      mockSupabase.from.mockReturnValue({
        insert: jest.fn().mockReturnValue({
          select: jest.fn().mockReturnValue({
            single: jest.fn().mockResolvedValue({ data: mockCustomer })
          })
        })
      })
      
      const result = await service.create('user-id', {
        name: 'Test Customer',
        phone: '08123456789',
        email: 'test@example.com'
      })
      
      expect(result).toEqual(mockCustomer)
      expect(mockSupabase.from).toHaveBeenCalledWith('customers')
    })
    
    it('should throw error for duplicate phone', async () => {
      mockSupabase.from.mockReturnValue({
        select: jest.fn().mockReturnValue({
          eq: jest.fn().mockReturnValue({
            eq: jest.fn().mockReturnValue({
              single: jest.fn().mockResolvedValue({ data: { id: 'existing' } })
            })
          })
        })
      })
      
      await expect(
        service.create('user-id', {
          name: 'Test',
          phone: '08123456789'
        })
      ).rejects.toThrow('Customer with this phone number already exists')
    })
  })
})

// __tests__/services/rfm.service.test.ts
describe('RFMService', () => {
  describe('calculateRFMScores', () => {
    it('should calculate correct RFM scores', () => {
      const metrics = [
        {
          customerId: '1',
          lastTransactionDate: new Date('2025-07-01'),
          transactionCount: 10,
          totalAmount: 5000000
        },
        {
          customerId: '2',
          lastTransactionDate: new Date('2025-06-01'),
          transactionCount: 5,
          totalAmount: 2000000
        },
        {
          customerId: '3',
          lastTransactionDate: new Date('2025-01-01'),
          transactionCount: 1,
          totalAmount: 500000
        }
      ]
      
      const service = new RFMService()
      const scores = service.calculateRFMScores(metrics)
      
      expect(scores[0].recencyScore).toBe(5) // Most recent
      expect(scores[0].frequencyScore).toBe(5) // Most frequent
      expect(scores[0].monetaryScore).toBe(5) // Highest value
      
      expect(scores[2].recencyScore).toBe(1) // Least recent
      expect(scores[2].frequencyScore).toBe(1) // Least frequent
      expect(scores[2].monetaryScore).toBe(1) // Lowest value
    })
  })
  
  describe('determineSegment', () => {
    it('should identify champions correctly', () => {
      const service = new RFMService()
      const segment = service.determineSegment({
        recencyScore: 5,
        frequencyScore: 5,
        monetaryScore: 5
      })
      
      expect(segment).toBe('champions')
    })
    
    it('should identify at_risk correctly', () => {
      const service = new RFMService()
      const segment = service.determineSegment({
        recencyScore: 2,
        frequencyScore: 4,
        monetaryScore: 4
      })
      
      expect(segment).toBe('at_risk')
    })
  })
})
```

### Integration Tests

```typescript
// __tests__/api/customers.test.ts
import { GET, POST } from '@/app/api/customers/route'
import { createMockRequest } from '@/tests/utils'

describe('/api/customers', () => {
  describe('GET', () => {
    it('should return customers with pagination', async () => {
      const request = createMockRequest({
        method: 'GET',
        url: '/api/customers?page=1&limit=10'
      })
      
      const response = await GET(request)
      const data = await response.json()
      
      expect(response.status).toBe(200)
      expect(data.success).toBe(true)
      expect(data.data).toHaveProperty('customers')
      expect(data.data).toHaveProperty('pagination')
    })
    
    it('should filter by segment', async () => {
      const request = createMockRequest({
        method: 'GET',
        url: '/api/customers?segment=champions'
      })
      
      const response = await GET(request)
      const data = await response.json()
      
      expect(response.status).toBe(200)
      expect(data.data.customers).toBeInstanceOf(Array)
    })
  })
  
  describe('POST', () => {
    it('should create new customer', async () => {
      const request = createMockRequest({
        method: 'POST',
        body: {
          name: 'New Customer',
          phone: '08123456789',
          email: 'new@example.com'
        }
      })
      
      const response = await POST(request)
      const data = await response.json()
      
      expect(response.status).toBe(201)
      expect(data.success).toBe(true)
      expect(data.data).toHaveProperty('id')
    })
    
    it('should validate input', async () => {
      const request = createMockRequest({
        method: 'POST',
        body: {
          name: 'A', // Too short
          phone: '123' // Invalid format
        }
      })
      
      const response = await POST(request)
      const data = await response.json()
      
      expect(response.status).toBe(400)
      expect(data.success).toBe(false)
    })
  })
})
```

### E2E Tests

```typescript
// e2e/customer-journey.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Customer Management Journey', () => {
  test.beforeEach(async ({ page }) => {
    // Login
    await page.goto('/login')
    await page.fill('[name="email"]', 'test@example.com')
    await page.fill('[name="password"]', 'password123')
    await page.click('button[type="submit"]')
    await page.waitForURL('/dashboard')
  })
  
  test('should add new customer', async ({ page }) => {
    // Navigate to customers
    await page.click('a[href="/customers"]')
    await page.waitForSelector('h1:has-text("Data Pelanggan")')
    
    // Click add button
    await page.click('a:has-text("Tambah Pelanggan")')
    await page.waitForURL('/customers/new')
    
    // Fill form
    await page.fill('[name="name"]', 'Test Customer')
    await page.fill('[name="phone"]', '08123456789')
    await page.fill('[name="email"]', 'test@customer.com')
    await page.fill('[name="city"]', 'Jakarta')
    
    // Submit
    await page.click('button:has-text("Simpan Pelanggan")')
    
    // Verify success
    await page.waitForSelector('text=Pelanggan berhasil ditambahkan')
    await page.waitForURL(/\/customers\/[a-z0-9-]+/)
    
    // Verify customer details
    await expect(page.locator('h1')).toContainText('Test Customer')
    await expect(page.locator('text=08123456789')).toBeVisible()
  })
  
  test('should import customers from Excel', async ({ page }) => {
    await page.goto('/customers/import')
    
    // Upload file
    const fileInput = page.locator('input[type="file"]')
    await fileInput.setInputFiles('tests/fixtures/customers.xlsx')
    
    // Preview
    await page.click('button:has-text("Preview")')
    await page.waitForSelector('table')
    
    // Verify preview
    const rows = page.locator('tbody tr')
    await expect(rows).toHaveCount(10)
    
    // Import
    await page.click('button:has-text("Import")')
    await page.waitForSelector('text=Successfully imported 10 customers')
  })
})
```

## 6.7 Deployment Guide

### Production Build

```bash
# Build optimization
pnpm build

# Analyze bundle size
pnpm analyze

# Type check
pnpm type-check

# Lint and format
pnpm lint
pnpm format
```

### Vercel Deployment

```json
// vercel.json
{
  "buildCommand": "pnpm build",
  "outputDirectory": ".next",
  "devCommand": "pnpm dev",
  "installCommand": "pnpm install",
  "framework": "nextjs",
  "regions": ["sin1"],
  "env": {
    "NEXT_PUBLIC_SUPABASE_URL": "@supabase_url",
    "NEXT_PUBLIC_SUPABASE_ANON_KEY": "@supabase_anon_key",
    "SUPABASE_SERVICE_ROLE_KEY": "@supabase_service_key",
    "OPENAI_API_KEY": "@openai_api_key"
  },
  "functions": {
    "app/api/rfm/calculate/route.ts": {
      "maxDuration": 60
    },
    "app/api/content/generate/route.ts": {
      "maxDuration": 30
    }
  }
}
```

### Environment Variables Setup

```bash
# Production environment
vercel env add NEXT_PUBLIC_SUPABASE_URL
vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY
vercel env add SUPABASE_SERVICE_ROLE_KEY
vercel env add OPENAI_API_KEY
vercel env add RESEND_API_KEY
vercel env add FONNTE_TOKEN
```

### Monitoring Setup

```typescript
// lib/monitoring/sentry.ts
import * as Sentry from '@sentry/nextjs'

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  integrations: [
    new Sentry.BrowserTracing(),
    new Sentry.Replay()
  ],
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
})

// lib/monitoring/analytics.ts
import posthog from 'posthog-js'

if (typeof window !== 'undefined') {
  posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
    api_host: 'https://app.posthog.com',
    loaded: (posthog) => {
      if (process.env.NODE_ENV === 'development') posthog.opt_out_capturing()
    }
  })
}
```

## 6.8 Performance Optimization

### Database Query Optimization

```typescript
// services/optimized-queries.ts
export class OptimizedQueries {
  // Use database functions for complex calculations
  async getRFMSummary(userId: string) {
    const { data, error } = await supabase.rpc('get_rfm_summary', {
      p_user_id: userId
    })
    
    if (error) throw error
    return data
  }
  
  // Batch operations
  async batchUpdateCustomers(updates: CustomerUpdate[]) {
    const chunks = chunk(updates, 100) // Split into chunks of 100
    
    const results = await Promise.all(
      chunks.map(chunk => 
        supabase.from('customers').upsert(chunk)
      )
    )
    
    return results
  }
  
  // Use database views for complex joins
  async getCustomerInsights(userId: string) {
    const { data, error } = await supabase
      .from('customer_insights_view')
      .select('*')
      .eq('user_id', userId)
    
    if (error) throw error
    return data
  }
}

// Database function example
CREATE OR REPLACE FUNCTION get_rfm_summary(p_user_id UUID)
RETURNS TABLE (
  segment TEXT,
  customer_count BIGINT,
  total_value DECIMAL,
  avg_recency DECIMAL,
  avg_frequency DECIMAL
) AS $$
BEGIN
  RETURN QUERY
  SELECT 
    r.segment::TEXT,
    COUNT(*)::BIGINT as customer_count,
    SUM(r.monetary_value) as total_value,
    AVG(r.recency_days)::DECIMAL as avg_recency,
    AVG(r.frequency_count)::DECIMAL as avg_frequency
  FROM rfm_analysis r
  WHERE r.user_id = p_user_i
