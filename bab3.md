# BAB 3
# RFM ANALYSIS ENGINE

## 3.1 Memahami RFM untuk UMKM Indonesia

RFM Analysis adalah metode segmentasi pelanggan yang telah terbukti efektif selama puluhan tahun. Namun, implementasinya untuk UMKM Indonesia memerlukan penyesuaian khusus. Smart Marketing Agent mengadaptasi RFM klasik dengan mempertimbangkan karakteristik unik pasar Indonesia: pola belanja musiman (Lebaran, tahun ajaran baru), preferensi pembayaran (transfer, COD), dan perilaku konsumen lokal.

### Mengapa RFM Cocok untuk UMKM Batik?

**1. Sederhana namun Powerful**
Hanya dengan 3 metrik dasar, mitra UMKM dapat memahami seluruh customer base mereka. Tidak perlu data demografis kompleks atau behavioral tracking yang mahal.

**2. Actionable Insights**
Setiap segmen RFM memiliki strategi marketing yang jelas. Mitra UMKM tidak perlu bingung "sekarang ngapain?" setelah melihat analisis.

**3. Proven ROI**
Studi menunjukkan RFM dapat meningkatkan response rate hingga 300% dan revenue hingga 180% dibanding mass marketing.

**4. Adaptable untuk Batik**
Bisnis batik memiliki karakteristik unik: repeat customer tinggi, seasonal pattern, dan high customer lifetime value. RFM perfect fit untuk karakteristik ini.

## 3.2 Komponen RFM Explained

### Recency (R) - Seberapa Baru Transaksi Terakhir?

```typescript
// Recency menunjukkan engagement level pelanggan
interface RecencyMetric {
  dayssSinceLastPurchase: number;
  lastPurchaseDate: Date;
  engagementStatus: 'active' | 'cooling' | 'cold' | 'frozen';
}

// Threshold untuk UMKM Batik (dalam hari)
const RECENCY_THRESHOLDS = {
  ACTIVE: 30,      // < 30 hari: Masih hangat
  COOLING: 60,     // 30-60 hari: Mulai mendingin  
  COLD: 90,        // 60-90 hari: Dingin
  FROZEN: 180      // > 180 hari: Beku
};
```

**Insight untuk Batik:**
- Pelanggan batik typically beli 3-4x setahun
- Peak season: Lebaran, wisuda, wedding season
- Recency threshold disesuaikan dengan seasonal pattern

### Frequency (F) - Seberapa Sering Membeli?

```typescript
// Frequency menunjukkan loyalty level
interface FrequencyMetric {
  totalTransactions: number;
  purchaseInterval: number; // Average days between purchases
  loyaltyLevel: 'one-time' | 'occasional' | 'regular' | 'vip';
}

// Threshold untuk UMKM Batik (jumlah transaksi/tahun)
const FREQUENCY_THRESHOLDS = {
  ONE_TIME: 1,     // Pembeli coba-coba
  OCCASIONAL: 2,   // Pembeli musiman
  REGULAR: 4,      // Pembeli regular (quarterly)
  VIP: 6           // Pembeli VIP (bi-monthly)
};
```

**Insight untuk Batik:**
- Frequency rendah NORMAL untuk batik (bukan daily goods)
- Focus pada purchase interval, bukan absolute count
- Wedding/corporate customer punya pattern berbeda

### Monetary (M) - Seberapa Besar Nilai Transaksi?

```typescript
// Monetary menunjukkan customer value
interface MonetaryMetric {
  totalSpent: number;
  averageOrderValue: number;
  customerLifetimeValue: number;
  spendingTier: 'bronze' | 'silver' | 'gold' | 'platinum';
}

// Threshold untuk UMKM Batik (dalam Rupiah)
const MONETARY_THRESHOLDS = {
  BRONZE: 500_000,      // Entry level
  SILVER: 2_000_000,    // Mid tier
  GOLD: 5_000_000,      // High value
  PLATINUM: 10_000_000  // Premium
};
```

**Insight untuk Batik:**
- AOV batik varies widely (100rb - 5jt)
- Corporate/wedding order boost monetary significantly
- Consider product mix (batik printing vs tulis)

## 3.3 Scoring Algorithm Implementation

### Dynamic Quintile Scoring

```typescript
export class RFMScoringEngine {
  private readonly QUINTILE_COUNT = 5;
  
  /**
   * Calculate RFM scores using dynamic quintiles
   * Adaptif terhadap distribusi data actual
   */
  calculateScores(customers: CustomerMetrics[]): RFMScore[] {
    // Sort untuk setiap metrik
    const recencyValues = customers
      .map(c => c.daysSinceLastPurchase)
      .sort((a, b) => a - b); // Ascending (lower is better)
      
    const frequencyValues = customers
      .map(c => c.transactionCount)
      .sort((a, b) => b - a); // Descending (higher is better)
      
    const monetaryValues = customers
      .map(c => c.totalSpent)
      .sort((a, b) => b - a); // Descending (higher is better)
    
    // Calculate quintile breakpoints
    const recencyBreaks = this.getQuintileBreakpoints(recencyValues);
    const frequencyBreaks = this.getQuintileBreakpoints(frequencyValues);
    const monetaryBreaks = this.getQuintileBreakpoints(monetaryValues);
    
    // Score each customer
    return customers.map(customer => ({
      customerId: customer.id,
      recencyScore: this.getRecencyScore(
        customer.daysSinceLastPurchase, 
        recencyBreaks
      ),
      frequencyScore: this.getScore(
        customer.transactionCount, 
        frequencyBreaks
      ),
      monetaryScore: this.getScore(
        customer.totalSpent, 
        monetaryBreaks
      ),
      metrics: customer
    }));
  }
  
  /**
   * Calculate quintile breakpoints dari data actual
   * Menghindari bias dari outliers
   */
  private getQuintileBreakpoints(values: number[]): number[] {
    const n = values.length;
    const breakpoints: number[] = [];
    
    for (let i = 1; i < this.QUINTILE_COUNT; i++) {
      const index = Math.floor((n * i) / this.QUINTILE_COUNT);
      breakpoints.push(values[index]);
    }
    
    return breakpoints;
  }
  
  /**
   * Recency scoring (inverse - lower is better)
   */
  private getRecencyScore(value: number, breakpoints: number[]): number {
    // Special handling untuk recency
    for (let i = 0; i < breakpoints.length; i++) {
      if (value <= breakpoints[i]) {
        return this.QUINTILE_COUNT - i;
      }
    }
    return 1; // Worst score
  }
  
  /**
   * Standard scoring (higher is better)
   */
  private getScore(value: number, breakpoints: number[]): number {
    for (let i = 0; i < breakpoints.length; i++) {
      if (value >= breakpoints[i]) {
        return this.QUINTILE_COUNT - i;
      }
    }
    return 1; // Worst score
  }
}
```

### Handling Edge Cases

```typescript
export class RFMEdgeCaseHandler {
  /**
   * Handle new customers dengan purchase history minimal
   */
  handleNewCustomers(customer: CustomerMetrics): RFMScore {
    const daysSinceJoin = this.daysBetween(
      customer.joinDate, 
      new Date()
    );
    
    if (daysSinceJoin < 30) {
      // New customer bonus scoring
      return {
        customerId: customer.id,
        recencyScore: 5, // Perfect recency
        frequencyScore: 3, // Neutral frequency
        monetaryScore: this.calculateMonetaryScore(customer.totalSpent),
        isNewCustomer: true
      };
    }
    
    // Standard scoring untuk customer yang sudah cukup lama
    return this.standardScoring(customer);
  }
  
  /**
   * Handle seasonal patterns untuk bisnis batik
   */
  adjustForSeasonality(scores: RFMScore[], analysisDate: Date): RFMScore[] {
    const isNearLebaran = this.isNearLebaran(analysisDate);
    const isWeddingSeason = this.isWeddingSeason(analysisDate);
    
    return scores.map(score => {
      // Adjust recency expectation during low season
      if (!isNearLebaran && !isWeddingSeason) {
        if (score.metrics.daysSinceLastPurchase < 120) {
          score.recencyScore = Math.min(5, score.recencyScore + 1);
        }
      }
      
      return score;
    });
  }
  
  /**
   * Handle B2B customers dengan pattern berbeda
   */
  handleCorporateCustomers(customer: CustomerMetrics): RFMScore {
    // Corporate customers biasanya:
    // - Order jarang tapi value besar
    // - Seasonal (event perusahaan)
    // - Perlu treatment khusus
    
    const isCorporate = customer.tags?.includes('corporate') ||
                       customer.averageOrderValue > 5_000_000;
    
    if (isCorporate) {
      return {
        customerId: customer.id,
        recencyScore: this.getCorporateRecencyScore(customer),
        frequencyScore: Math.max(3, this.standardFrequencyScore(customer)),
        monetaryScore: 5, // Usually high value
        isCorporate: true
      };
    }
    
    return this.standardScoring(customer);
  }
}
```

## 3.4 Customer Segmentation Logic

### The 10 Core Segments

```typescript
export enum CustomerSegment {
  CHAMPIONS = 'champions',
  LOYAL_CUSTOMERS = 'loyal_customers',
  POTENTIAL_LOYALISTS = 'potential_loyalists',
  NEW_CUSTOMERS = 'new_customers',
  PROMISING = 'promising',
  NEED_ATTENTION = 'need_attention',
  ABOUT_TO_SLEEP = 'about_to_sleep',
  AT_RISK = 'at_risk',
  CANT_LOSE_THEM = 'cant_lose_them',
  LOST = 'lost'
}

export class SegmentationEngine {
  /**
   * Determine segment based on RFM scores
   * Using proven segmentation matrix
   */
  determineSegment(rfm: RFMScore): CustomerSegment {
    const { recencyScore: R, frequencyScore: F, monetaryScore: M } = rfm;
    
    // Champions: Best customers
    if (R >= 4 && F >= 4 && M >= 4) {
      return CustomerSegment.CHAMPIONS;
    }
    
    // Loyal Customers: Shop regularly
    if (R >= 3 && F >= 3 && M >= 3) {
      return CustomerSegment.LOYAL_CUSTOMERS;
    }
    
    // Potential Loyalists: Recent high value
    if (R >= 4 && F >= 2 && M >= 3) {
      return CustomerSegment.POTENTIAL_LOYALISTS;
    }
    
    // New Customers: First time buyer
    if (rfm.isNewCustomer || (R === 5 && F === 1)) {
      return CustomerSegment.NEW_CUSTOMERS;
    }
    
    // Promising: Recent but low frequency/value
    if (R >= 4 && F === 1 && M <= 3) {
      return CustomerSegment.PROMISING;
    }
    
    // Need Attention: Above average but declining
    if (R === 3 && F >= 3 && M >= 3) {
      return CustomerSegment.NEED_ATTENTION;
    }
    
    // About to Sleep: Below average recency
    if (R === 2 && F >= 2) {
      return CustomerSegment.ABOUT_TO_SLEEP;
    }
    
    // At Risk: Was good, now declining
    if (R <= 2 && F >= 3 && M >= 3) {
      return CustomerSegment.AT_RISK;
    }
    
    // Can't Lose Them: High value but gone
    if (R <= 2 && F >= 4 && M >= 4) {
      return CustomerSegment.CANT_LOSE_THEM;
    }
    
    // Lost: Long time no see
    if (R === 1) {
      return CustomerSegment.LOST;
    }
    
    // Default fallback
    return CustomerSegment.NEED_ATTENTION;
  }
  
  /**
   * Get segment characteristics and recommended actions
   */
  getSegmentProfile(segment: CustomerSegment): SegmentProfile {
    const profiles: Record<CustomerSegment, SegmentProfile> = {
      [CustomerSegment.CHAMPIONS]: {
        description: 'Pelanggan terbaik Anda',
        characteristics: [
          'Baru saja berbelanja',
          'Sering berbelanja',
          'Nilai belanja tinggi'
        ],
        recommendedActions: [
          'Berikan reward loyalty',
          'Early access ke koleksi baru',
          'Personal shopping assistance',
          'Jadikan brand ambassador'
        ],
        marketingTone: 'exclusive',
        priority: 'maintain'
      },
      
      [CustomerSegment.LOYAL_CUSTOMERS]: {
        description: 'Pelanggan setia yang konsisten',
        characteristics: [
          'Belanja rutin',
          'Nilai stabil',
          'Engagement bagus'
        ],
        recommendedActions: [
          'Program poin loyalty',
          'Birthday rewards',
          'Exclusive member benefits',
          'Ajak join community'
        ],
        marketingTone: 'appreciation',
        priority: 'maintain'
      },
      
      [CustomerSegment.POTENTIAL_LOYALISTS]: {
        description: 'Calon pelanggan setia',
        characteristics: [
          'Baru mulai sering belanja',
          'Nilai meningkat',
          'Respons bagus'
        ],
        recommendedActions: [
          'Welcome series email',
          'Edukasi produk',
          'Limited time offers',
          'Social proof content'
        ],
        marketingTone: 'nurturing',
        priority: 'grow'
      },
      
      [CustomerSegment.NEW_CUSTOMERS]: {
        description: 'Pelanggan baru bergabung',
        characteristics: [
          'First time buyer',
          'Exploring products',
          'Need guidance'
        ],
        recommendedActions: [
          'Welcome discount',
          'Product guide',
          'Follow-up care',
          'Collect feedback'
        ],
        marketingTone: 'welcoming',
        priority: 'nurture'
      },
      
      [CustomerSegment.AT_RISK]: {
        description: 'Pelanggan bagus yang mulai menjauh',
        characteristics: [
          'Dulu aktif',
          'Sekarang jarang',
          'Nilai masih tinggi'
        ],
        recommendedActions: [
          'Win-back campaign URGENT',
          'Personal outreach',
          'Special comeback offer',
          'Ask for feedback'
        ],
        marketingTone: 'concerned',
        priority: 'urgent'
      },
      
      [CustomerSegment.CANT_LOSE_THEM]: {
        description: 'Pelanggan VIP yang hampir hilang',
        characteristics: [
          'Highest value',
          'Dulu sangat loyal',
          'Sekarang tidak aktif'
        ],
        recommendedActions: [
          'CEO/Owner personal call',
          'Exclusive win-back offer',
          'VIP treatment',
          'Problem resolution'
        ],
        marketingTone: 'vip_recovery',
        priority: 'critical'
      },
      
      [CustomerSegment.LOST]: {
        description: 'Pelanggan yang sudah lama tidak aktif',
        characteristics: [
          'Tidak ada aktivitas 6+ bulan',
          'Tidak respons campaign',
          'Mungkin pindah kompetitor'
        ],
        recommendedActions: [
          'Reactivation campaign',
          '"We miss you" message',
          'Feedback survey',
          'Major incentive needed'
        ],
        marketingTone: 'reactivation',
        priority: 'low'
      }
      
      // ... other segments
    };
    
    return profiles[segment];
  }
}
```

## 3.5 Implementation in Production

### Weekly Analysis Cron Job

```typescript
// app/api/cron/weekly-analysis/route.ts
import { NextResponse } from 'next/server';
import { RFMAnalysisService } from '@/services/rfm';
import { NotificationService } from '@/services/notification';

export async function GET(request: Request) {
  // Verify cron secret
  const authHeader = request.headers.get('authorization');
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  try {
    console.log('[CRON] Starting weekly RFM analysis...');
    
    // Get all active UMKM users
    const users = await getUsersForAnalysis();
    
    // Process each user in parallel (with concurrency limit)
    const results = await processInBatches(users, async (user) => {
      try {
        // Run RFM analysis
        const analysis = await rfmService.calculateRFM(user.id);
        
        // Generate insights
        const insights = await generateInsights(analysis);
        
        // Send notification
        await notificationService.sendAnalysisComplete(user, insights);
        
        return { userId: user.id, success: true };
      } catch (error) {
        console.error(`[CRON] Error processing user ${user.id}:`, error);
        return { userId: user.id, success: false, error };
      }
    }, { concurrency: 5 });
    
    // Summary
    const successful = results.filter(r => r.success).length;
    const failed = results.filter(r => !r.success).length;
    
    console.log(`[CRON] Analysis complete. Success: ${successful}, Failed: ${failed}`);
    
    return NextResponse.json({
      success: true,
      processed: results.length,
      successful,
      failed,
      timestamp: new Date().toISOString()
    });
    
  } catch (error) {
    console.error('[CRON] Fatal error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}

/**
 * Process users in batches to avoid overwhelming the system
 */
async function processInBatches<T, R>(
  items: T[],
  processor: (item: T) => Promise<R>,
  options: { concurrency: number }
): Promise<R[]> {
  const results: R[] = [];
  
  for (let i = 0; i < items.length; i += options.concurrency) {
    const batch = items.slice(i, i + options.concurrency);
    const batchResults = await Promise.all(
      batch.map(item => processor(item))
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

### Real-time Dashboard Updates

```typescript
// hooks/useRFMDashboard.ts
export function useRFMDashboard() {
  const [data, setData] = useState<DashboardData | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Initial load
    loadDashboardData();
    
    // Subscribe to real-time updates
    const subscription = supabase
      .channel('rfm-updates')
      .on('postgres_changes', {
        event: '*',
        schema: 'public',
        table: 'rfm_analysis'
      }, (payload) => {
        // Update dashboard when new analysis available
        if (payload.eventType === 'INSERT') {
          loadDashboardData();
        }
      })
      .subscribe();
    
    return () => {
      subscription.unsubscribe();
    };
  }, []);
  
  const loadDashboardData = async () => {
    try {
      setLoading(true);
      
      // Fetch latest analysis
      const analysis = await fetchLatestAnalysis();
      
      // Calculate segment distribution
      const segmentDistribution = calculateSegmentDistribution(analysis);
      
      // Calculate trends
      const trends = await calculateTrends(analysis);
      
      // Get actionable insights
      const insights = generateActionableInsights(analysis);
      
      setData({
        totalCustomers: analysis.length,
        segmentDistribution,
        trends,
        insights,
        lastUpdated: new Date()
      });
    } finally {
      setLoading(false);
    }
  };
  
  return { data, loading, refresh: loadDashboardData };
}
```

### Performance Optimization

```typescript
// services/rfm/optimization.ts
export class RFMOptimization {
  private cache: Map<string, CachedResult> = new Map();
  
  /**
   * Incremental analysis - only process changed customers
   */
  async incrementalAnalysis(userId: string): Promise<RFMResult[]> {
    const lastAnalysis = await this.getLastAnalysis(userId);
    const lastAnalysisDate = lastAnalysis?.date || new Date(0);
    
    // Get only customers with new transactions
    const changedCustomers = await this.getChangedCustomers(
      userId,
      lastAnalysisDate
    );
    
    if (changedCustomers.length === 0) {
      // No changes, return cached results
      return lastAnalysis?.results || [];
    }
    
    // Fetch unchanged results from cache
    const unchangedResults = await this.getCachedResults(
      userId,
      changedCustomers.map(c => c.id)
    );
    
    // Calculate only for changed customers
    const newResults = await this.calculateForCustomers(
      changedCustomers
    );
    
    // Merge results
    const allResults = [...unchangedResults, ...newResults];
    
    // Update cache
    await this.updateCache(userId, allResults);
    
    return allResults;
  }
  
  /**
   * Batch processing untuk large datasets
   */
  async batchProcess(
    customers: Customer[],
    batchSize: number = 100
  ): Promise<RFMResult[]> {
    const results: RFMResult[] = [];
    
    // Process in batches to avoid memory issues
    for (let i = 0; i < customers.length; i += batchSize) {
      const batch = customers.slice(i, i + batchSize);
      
      // Parallel processing within batch
      const batchResults = await Promise.all(
        batch.map(customer => this.processCustomer(customer))
      );
      
      results.push(...batchResults);
      
      // Allow event loop to breathe
      if (i % (batchSize * 10) === 0) {
        await new Promise(resolve => setImmediate(resolve));
      }
    }
    
    return results;
  }
  
  /**
   * Query optimization dengan proper indexing
   */
  getOptimizedQuery(): string {
    return `
      WITH customer_summary AS (
        SELECT 
          c.id,
          c.name,
          c.phone,
          c.tags,
          COALESCE(
            DATE_PART('day', CURRENT_DATE - MAX(t.transaction_date)),
            999
          ) as days_since_last,
          COUNT(t.id) as transaction_count,
          COALESCE(SUM(t.amount), 0) as total_amount,
          COALESCE(AVG(t.amount), 0) as avg_amount
        FROM customers c
        LEFT JOIN transactions t ON c.id = t.customer_id
        WHERE c.user_id = $1
          AND c.is_active = true
        GROUP BY c.id, c.name, c.phone, c.tags
      ),
      rfm_calc AS (
        SELECT
          *,
          NTILE(5) OVER (ORDER BY days_since_last DESC) as r_score,
          NTILE(5) OVER (ORDER BY transaction_count) as f_score,
          NTILE(5) OVER (ORDER BY total_amount) as m_score
        FROM customer_summary
      )
      SELECT * FROM rfm_calc
      ORDER BY (r_score + f_score + m_score) DESC;
    `;
  }
}
```

## 3.6 Insights Generation

### AI-Powered Insights

```typescript
export class InsightsGenerator {
  /**
   * Generate human-readable insights from RFM analysis
   */
  async generateInsights(
    analysis: RFMAnalysis
  ): Promise<BusinessInsights> {
    const insights: BusinessInsights = {
      summary: await this.generateSummary(analysis),
      alerts: this.detectAlerts(analysis),
      opportunities: this.findOpportunities(analysis),
      recommendations: await this.getRecommendations(analysis)
    };
    
    return insights;
  }
  
  private async generateSummary(analysis: RFMAnalysis): Promise<string> {
    const segmentCounts = this.countBySegment(analysis.results);
    const trends = this.calculateTrends(analysis);
    
    const prompt = `
      Buat summary singkat untuk mitra UMKM batik dengan data:
      - Total pelanggan: ${analysis.totalCustomers}
      - Champions: ${segmentCounts.champions}
      - At Risk: ${segmentCounts.at_risk}
      - Lost: ${segmentCounts.lost}
      - Trend: ${trends.direction}
      
      Gunakan bahasa Indonesia yang friendly dan encouraging.
      Max 3 kalimat.
    `;
    
    return await this.aiService.generate(prompt);
  }
  
  private detectAlerts(analysis: RFMAnalysis): Alert[] {
    const alerts: Alert[] = [];
    
    // Alert 1: High value customers at risk
    const atRiskHighValue = analysis.results.filter(
      r => r.segment === 'at_risk' && r.monetaryScore >= 4
    );
    
    if (atRiskHighValue.length > 0) {
      alerts.push({
        level: 'critical',
        title: 'Pelanggan VIP Berisiko!',
        message: `${atRiskHighValue.length} pelanggan dengan nilai tinggi mulai tidak aktif`,
        actionRequired: 'Hubungi segera dengan penawaran spesial',
        customers: atRiskHighValue.map(c => c.customerId)
      });
    }
    
    // Alert 2: Increasing churn rate
    const churnRate = this.calculateChurnRate(analysis);
    if (churnRate > 0.2) { // 20% churn
      alerts.push({
        level: 'warning',
        title: 'Churn Rate Meningkat',
        message: `${(churnRate * 100).toFixed(0)}% pelanggan tidak aktif dalam 3 bulan terakhir`,
        actionRequired: 'Launch reactivation campaign'
      });
    }
    
    // Alert 3: New customer opportunity
    const newCustomers = analysis.results.filter(
      r => r.segment === 'new_customers'
    );
    
    if (newCustomers.length >= 5) {
      alerts.push({
        level: 'info',
        title: 'Momentum Pelanggan Baru!',
        message: `${newCustomers.length} pelanggan baru bulan ini`,
        actionRequired: 'Maksimalkan dengan welcome series'
      });
    }
    
    return alerts;
  }
  
  private findOpportunities(analysis: RFMAnalysis): Opportunity[] {
    const opportunities: Opportunity[] = [];
    
    // Opportunity 1: Upsell to potential loyalists
    const potentialLoyalists = analysis.results.filter(
      r => r.segment === 'potential_loyalists'
    );
    
    if (potentialLoyalists.length > 0) {
      const avgValue = this.calculateAvgValue(potentialLoyalists);
      const potentialRevenue = avgValue * potentialLoyalists.length * 2;
      
      opportunities.push({
        type: 'upsell',
        segment: 'potential_loyalists',
        estimatedValue: potentialRevenue,
        description: 'Tingkatkan frequency dengan loyalty program',
        confidence: 0.7
      });
    }
    
    // Opportunity 2: Win back campaign
    const canWinBack = analysis.results.filter(
      r => ['at_risk', 'about_to_sleep'].includes(r.segment)
    );
    
    if (canWinBack.length > 0) {
      opportunities.push({
        type: 'retention',
        segment: 'at_risk',
        estimatedValue: this.estimateWinBackValue(canWinBack),
        description: 'Win back campaign dengan personal approach',
        confidence: 0.5
      });
    }
    
    return opportunities.sort((a, b) => b.estimatedValue - a.estimatedValue);
  }
}
```

## 3.7 Practical Examples

### Case Study: Batik Sekar Arum

```typescript
// Real implementation example
const batikSekarArumAnalysis = {
  businessProfile: {
    name: "Batik Sekar Arum",
    location: "Yogyakarta",
    customerBase: 547,
    averageOrderValue: 450_000
  },
  
  beforeRFM: {
    marketingApproach: "Broadcast ke semua",
    responseRate: "2-3%",
    monthlyRevenue: 25_000_000,
    customerComplaints: "Spam, tidak relevan"
  },
  
  rfmImplementation: {
    week1: "Data collection dan initial analysis",
    week2: "Segment customers dan create strategies",
    week3: "Launch targeted campaigns",
    week4: "Measure dan optimize"
  },
  
  results: {
    segments: {
      champions: 47,
      loyal: 89,
      potentialLoyalist: 124,
      newCustomers: 67,
      atRisk: 98,
      lost: 122
    },
    
    campaignResults: {
      championsVIP: {
        approach: "Exclusive preview koleksi baru",
        responseRate: "68%",
        additionalRevenue: 8_500_000
      },
      
      atRiskWinBack: {
        approach: "Personal WA dengan special discount",
        responseRate: "34%",
        savedCustomers: 33,
        savedRevenue: 12_000_000
      },
      
      newCustomerNurture: {
        approach: "Welcome series + education",
        retention: "78%",
        upgradedToLoyal: 23
      }
    },
    
    afterRFM: {
      responseRate: "24% average",
      monthlyRevenue: 42_000_000,
      revenueIncrease: "68%",
      customerSatisfaction: "Sangat personal dan relevan"
    }
  }
};
```

### Common Patterns in Batik Business

```typescript
const batikBusinessPatterns = {
  seasonalPatterns: {
    lebaran: {
      months: ["March", "April"],
      behavior: "3x normal transaction",
      focus: "Stock preparation, pre-order campaign"
    },
    
    weddingSeason: {
      months: ["June", "July", "August"],
      behavior: "Corporate & bulk orders",
      focus: "B2B marketing, volume discounts"
    },
    
    yearEnd: {
      months: ["November", "December"],
      behavior: "Gift purchases",
      focus: "Gift wrapping, hampers"
    }
  },
  
  customerArchetypes: {
    collector: {
      characteristics: "High frequency, high value",
      segment: "Champions",
      strategy: "Exclusive access, limited edition"
    },
    
    occasional: {
      characteristics: "Seasonal buyer, medium value",
      segment: "Loyal/Potential",
      strategy: "Reminder before season, bundle offers"
    },
    
    corporate: {
      characteristics: "Low frequency, very high value",
      segment: "Special handling",
      strategy: "Relationship management, custom service"
    },
    
    tourist: {
      characteristics: "One-time, varies value",
      segment: "New customers",
      strategy: "Memorable experience, online follow-up"
    }
  }
};
```

## 3.8 Integration dengan Smart Marketing Agent

### API Endpoints

```typescript
// RFM Analysis API
app.get('/api/rfm/analysis/:userId', async (req, res) => {
  const { userId } = req.params;
  const { forceRefresh = false } = req.query;
  
  try {
    // Check cache first
    if (!forceRefresh) {
      const cached = await getCachedAnalysis(userId);
      if (cached && !isStale(cached)) {
        return res.json({
          success: true,
          data: cached,
          source: 'cache'
        });
      }
    }
    
    // Run fresh analysis
    const analysis = await rfmService.calculateRFM(userId);
    const insights = await insightsGenerator.generateInsights(analysis);
    
    res.json({
      success: true,
      data: {
        analysis,
        insights,
        generatedAt: new Date()
      },
      source: 'fresh'
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

// Get specific segment details
app.get('/api/rfm/segments/:segment', async (req, res) => {
  const { segment } = req.params;
  const { userId } = req.user;
  
  const customers = await getCustomersBySegment(userId, segment);
  const profile = segmentationEngine.getSegmentProfile(segment);
  const suggestedContent = await contentGenerator.generateForSegment(segment);
  
  res.json({
    success: true,
    data: {
      segment,
      profile,
      customers,
      suggestedContent,
      totalCount: customers.length,
      totalValue: customers.reduce((sum, c) => sum + c.totalSpent, 0)
    }
  });
});
```

## 3.9 Best Practices & Tips

### Do's and Don'ts

```typescript
const rfmBestPractices = {
  dos: [
    {
      practice: "Regular analysis schedule",
      reason: "Consistency helps track progress",
      example: "Every Monday morning at 6 AM"
    },
    {
      practice: "Act on insights immediately",
      reason: "Fresh data = better response",
      example: "Contact at-risk within 48 hours"
    },
    {
      practice: "Personalize based on segment",
      reason: "Relevance drives engagement",
      example: "Different tone for VIP vs new"
    },
    {
      practice: "Track campaign results",
      reason: "Learn what works",
      example: "A/B test messages per segment"
    }
  ],
  
  donts: [
    {
      practice: "Over-segment",
      reason: "Too complex to manage",
      example: "Stick to 8-10 segments max"
    },
    {
      practice: "Ignore seasonal context",
      reason: "Misinterpret customer behavior",
      example: "Low sales in January is normal"
    },
    {
      practice: "Focus only on Champions",
      reason: "Miss growth opportunity",
      example: "Potential loyalists = future champions"
    },
    {
      practice: "Set and forget",
      reason: "RFM needs continuous refinement",
      example: "Review thresholds quarterly"
    }
  ]
};
```

## 3.10 Kesimpulan

RFM Analysis Engine adalah jantung dari Smart Marketing Agent. Dengan mengotomatisasi analisis yang kompleks dan menyajikannya dalam format yang actionable, mitra UMKM dapat:

1. **Memahami** pelanggan mereka lebih dalam
2. **Mengidentifikasi** peluang dan risiko lebih cepat
3. **Mengambil** action yang tepat waktu dan tepat sasaran
4. **Meningkatkan** revenue dan customer lifetime value

Implementasi RFM yang disesuaikan dengan konteks UMKM batik Indonesia membuat analisis ini bukan sekedar teori, tapi tool praktis yang langsung berdampak pada bisnis.

---

*"Data without action is just numbers. RFM transforms your customer data into strategic actions that drive real business growth."*
