# BAB 4
# AI-POWERED CONTENT

## 4.1 AI untuk Marketing UMKM: Game Changer

Bayangkan seorang mitra UMKM batik yang harus membuat konten marketing untuk 10 segmen pelanggan berbeda, masing-masing dengan 3 channel (WhatsApp, Instagram, Email). Itu berarti 30 konten unik setiap minggu! Tanpa bantuan AI, ini mission impossible. Smart Marketing Agent menghadirkan AI yang tidak hanya generate konten, tapi memahami konteks batik, budaya Indonesia, dan karakteristik setiap segmen pelanggan.

### Mengapa AI Essential untuk UMKM?

**1. Time Efficiency**
Dari 2 jam membuat 1 konten, menjadi 2 menit untuk 10 konten. Mitra UMKM bisa fokus pada produksi dan customer service.

**2. Consistency in Quality**
AI menjaga tone of voice dan brand message konsisten across all channels dan segments.

**3. Personalization at Scale**
Impossible untuk personalisasi manual ke ratusan pelanggan. AI makes it possible dan affordable.

**4. Data-Driven Creativity**
AI belajar dari performance data, continuously improving content effectiveness.

## 4.2 OpenAI Integration Architecture

### System Design for Optimal Performance

```typescript
// config/openai.ts
export class OpenAIConfig {
  private static instance: OpenAI;
  
  static getInstance(): OpenAI {
    if (!this.instance) {
      this.instance = new OpenAI({
        apiKey: process.env.OPENAI_API_KEY,
        defaultHeaders: {
          'OpenAI-Beta': 'assistants=v2'
        },
        maxRetries: 3,
        timeout: 30000 // 30 seconds
      });
    }
    return this.instance;
  }
  
  // Model selection based on use case
  static getModel(useCase: ContentUseCase): string {
    const models = {
      marketing_copy: 'gpt-4-turbo-preview', // Best for creative
      email_formal: 'gpt-4',                 // Best for professional
      whatsapp_casual: 'gpt-3.5-turbo',     // Fast & cost-effective
      product_description: 'gpt-4'           // Accuracy needed
    };
    
    return models[useCase] || 'gpt-3.5-turbo';
  }
  
  // Token limits untuk cost control
  static getTokenLimits(contentType: string): TokenLimits {
    return {
      whatsapp: { max: 300, target: 150 },      // Short & sweet
      instagram: { max: 500, target: 300 },     // Engaging but concise
      email: { max: 1000, target: 600 },        // More detailed
      blog: { max: 2000, target: 1500 }         // Long form
    }[contentType] || { max: 500, target: 300 };
  }
}
```

### Structured Output dengan Function Calling

```typescript
// services/ai/content-generator.ts
export class AIContentGenerator {
  private openai: OpenAI;
  private rateLimiter: RateLimiter;
  
  constructor() {
    this.openai = OpenAIConfig.getInstance();
    this.rateLimiter = new RateLimiter({
      requestsPerMinute: 50,
      requestsPerDay: 1000
    });
  }
  
  /**
   * Generate marketing content dengan structured output
   */
  async generateMarketingContent(
    params: ContentGenerationParams
  ): Promise<GeneratedContent> {
    // Rate limiting check
    await this.rateLimiter.checkLimit(params.userId);
    
    // Build system prompt dengan context
    const systemPrompt = this.buildSystemPrompt(params);
    
    // Create function schema for structured output
    const functionSchema = {
      name: 'create_marketing_content',
      description: 'Create marketing content for UMKM batik',
      parameters: {
        type: 'object',
        properties: {
          content: {
            type: 'object',
            properties: {
              headline: { type: 'string' },
              body: { type: 'string' },
              callToAction: { type: 'string' },
              hashtags: {
                type: 'array',
                items: { type: 'string' }
              },
              tone: { type: 'string' },
              targetAudience: { type: 'string' }
            },
            required: ['headline', 'body', 'callToAction']
          },
          metadata: {
            type: 'object',
            properties: {
              estimatedEngagement: { type: 'string' },
              bestPostingTime: { type: 'string' },
              contentStrategy: { type: 'string' }
            }
          }
        },
        required: ['content']
      }
    };
    
    try {
      const completion = await this.openai.chat.completions.create({
        model: OpenAIConfig.getModel(params.useCase),
        messages: [
          { role: 'system', content: systemPrompt },
          { role: 'user', content: this.buildUserPrompt(params) }
        ],
        functions: [functionSchema],
        function_call: { name: 'create_marketing_content' },
        temperature: params.creativity || 0.7,
        max_tokens: OpenAIConfig.getTokenLimits(params.contentType).max
      });
      
      // Parse structured response
      const functionCall = completion.choices[0].message.function_call;
      const result = JSON.parse(functionCall.arguments);
      
      // Post-process untuk ensure quality
      const processed = await this.postProcess(result, params);
      
      // Track usage untuk billing
      await this.trackUsage({
        userId: params.userId,
        tokens: completion.usage.total_tokens,
        model: completion.model,
        contentType: params.contentType
      });
      
      return processed;
      
    } catch (error) {
      console.error('AI Generation Error:', error);
      
      // Fallback to template if AI fails
      if (error.code === 'rate_limit_exceeded') {
        return this.getFallbackTemplate(params);
      }
      
      throw new AIGenerationError(error.message);
    }
  }
  
  /**
   * Build system prompt dengan rich context
   */
  private buildSystemPrompt(params: ContentGenerationParams): string {
    return `Anda adalah expert marketing consultant untuk UMKM batik Indonesia.
    
    CONTEXT:
    - Business: ${params.businessProfile.name}
    - Location: ${params.businessProfile.location}
    - Speciality: ${params.businessProfile.speciality || 'Batik tradisional'}
    - Target Market: ${params.businessProfile.targetMarket}
    
    BRAND VOICE:
    ${params.brandVoice || 'Profesional namun hangat, menghargai tradisi namun modern'}
    
    CONTENT REQUIREMENTS:
    - Language: Bahasa Indonesia yang baik dan benar
    - Tone: ${this.getToneForSegment(params.segment)}
    - Channel: ${params.contentType}
    - Objective: ${params.objective}
    
    CULTURAL SENSITIVITY:
    - Gunakan sapaan yang sesuai (Bapak/Ibu untuk formal, Kak untuk casual)
    - Hindari hard selling untuk segmen tertentu
    - Include local wisdom atau philosophy jika relevan
    - Respect religious dan cultural values
    
    BATIK CONTEXT:
    - Understand batik bukan sekedar kain, tapi warisan budaya
    - Each motif has meaning dan story
    - Quality matters more than quantity
    - Emphasize craftsmanship dan authenticity`;
  }
  
  /**
   * Dynamic prompt based on segment
   */
  private buildUserPrompt(params: ContentGenerationParams): string {
    const segmentPrompts = {
      champions: `Buat konten eksklusif untuk pelanggan VIP kami. 
        Mereka sudah sangat loyal dan sering membeli. 
        Buat mereka merasa special dan appreciated.
        Fokus: Exclusive preview, VIP benefits, personal touch.`,
        
      potential_loyalists: `Buat konten untuk pelanggan yang mulai sering membeli.
        Mereka tertarik tapi belum fully committed.
        Nurture mereka menjadi pelanggan setia.
        Fokus: Education, value proposition, social proof.`,
        
      at_risk: `Buat konten win-back untuk pelanggan yang mulai tidak aktif.
        Mereka dulu sering beli tapi sekarang menghilang.
        Approach dengan empati dan personal touch.
        Fokus: "We miss you", special comeback offer, ask feedback.`,
        
      new_customers: `Buat konten welcome untuk pelanggan baru.
        First impression sangat penting.
        Buat mereka merasa welcome dan guide their journey.
        Fokus: Welcome message, product guide, community invitation.`
    };
    
    const basePrompt = segmentPrompts[params.segment] || segmentPrompts.new_customers;
    
    return `${basePrompt}
    
    Specific Request: ${params.specificRequest || 'Buat konten menarik dan engaging'}
    
    Product Focus: ${params.productFocus || 'Koleksi batik terbaru'}
    
    Additional Context:
    - Season: ${this.getCurrentSeason()}
    - Special Events: ${this.getUpcomingEvents()}
    - Recent Trends: ${params.recentTrends || 'Batik modern untuk generasi muda'}`;
  }
}
```

## 4.3 Prompt Engineering untuk Batik Context

### The Art of Crafting Perfect Prompts

```typescript
export class BatikPromptEngineering {
  /**
   * Prompt templates yang sudah proven effective
   */
  private promptTemplates = {
    instagram_caption: {
      structure: [
        "Hook (pertanyaan atau statement menarik)",
        "Story/Context (2-3 kalimat)",
        "Value proposition",
        "Call to action",
        "Hashtags (5-10 relevant)"
      ],
      example: `"Tahukah Anda setiap motif batik punya makna mendalam? 🎨
      
      Batik Parang yang Anda lihat ini melambangkan kontinuitas dan 
      kekuatan. Dipercaya membawa energi positif bagi pemakainya.
      
      Dibuat dengan teknik tulis tradisional oleh pengrajin berpengalaman 
      20+ tahun. Setiap goresan malam adalah dedikasi untuk melestarikan 
      warisan budaya.
      
      Miliki batik bermakna Anda di www.batiksekar.com atau kunjungi 
      toko kami. Which motif tells YOUR story?
      
      #BatikIndonesia #BatikTulis #BatikParang #WisataBelanja 
      #FashionLokal #BatikModern #ProudlyIndonesian"`
    },
    
    whatsapp_personal: {
      structure: [
        "Personal greeting dengan nama",
        "Reference to past purchase/interaction",
        "Relevant offer/information",
        "Clear next step",
        "Warm closing"
      ],
      example: `"Selamat pagi Ibu Sarah 🌸
      
      Apa kabar? Semoga sehat selalu ya Bu. Saya notice Ibu terakhir 
      membeli batik motif Kawung 2 bulan lalu. Gimana Bu, nyaman dipakainya?
      
      Kebetulan kami baru terima koleksi eksklusif motif Kawung dengan 
      warna-warna baru yang lebih soft. Ingat Ibu pernah mention suka 
      warna pastel. This might be perfect for you!
      
      Kalau Ibu ada waktu, boleh saya kirimkan foto-fotonya via WA? 
      Atau kalau mau lihat langsung, Ibu bisa mampir ke toko. 
      I'll prepare some options untuk Ibu.
      
      Have a wonderful day!
      Salam hangat,
      Tim Batik Sekar 🙏"`
    },
    
    email_newsletter: {
      structure: [
        "Subject line yang compelling",
        "Personal greeting",
        "Value-first content",
        "Product showcase",
        "Customer story/testimonial",
        "Clear CTA",
        "Footer dengan contact info"
      ],
      maxLength: 800
    }
  };
  
  /**
   * Optimize prompt based on performance data
   */
  async optimizePrompt(
    basePrompt: string,
    performanceData: ContentPerformance[]
  ): Promise<string> {
    // Analyze what worked
    const highPerformers = performanceData
      .filter(p => p.engagementRate > 0.1)
      .sort((a, b) => b.engagementRate - a.engagementRate)
      .slice(0, 5);
    
    const successPatterns = this.extractPatterns(highPerformers);
    
    // Build optimization instructions
    const optimizationPrompt = `
      Optimize this prompt based on successful patterns:
      
      Successful elements:
      ${successPatterns.map(p => `- ${p}`).join('\n')}
      
      Original prompt: ${basePrompt}
      
      Create an improved version that incorporates successful elements.
    `;
    
    const improved = await this.callOpenAI(optimizationPrompt);
    return improved;
  }
  
  /**
   * Contextual adjustments untuk Indonesian market
   */
  applyIndonesianContext(prompt: string, params: ContextParams): string {
    const adjustments = [];
    
    // Religious sensitivity
    if (this.isNearReligiousEvent(params.date)) {
      adjustments.push(
        "Include appropriate religious greetings",
        "Avoid promotional tone, focus on values"
      );
    }
    
    // Regional preferences
    if (params.region) {
      const regionalPrefs = this.getRegionalPreferences(params.region);
      adjustments.push(...regionalPrefs);
    }
    
    // Generation-specific
    if (params.targetAge) {
      if (params.targetAge < 30) {
        adjustments.push(
          "Use casual tone dengan sedikit English",
          "Reference pop culture atau trending topics",
          "Focus on style dan self-expression"
        );
      } else if (params.targetAge > 50) {
        adjustments.push(
          "Use respectful formal tone",
          "Emphasize quality dan heritage",
          "Include philosophy atau wisdom"
        );
      }
    }
    
    return `${prompt}\n\nADDITIONAL CONTEXT:\n${adjustments.join('\n')}`;
  }
}
```

## 4.4 Content Personalization Engine

### Dynamic Personalization Based on Customer Data

```typescript
export class ContentPersonalizationEngine {
  /**
   * Personalize content untuk individual customer
   */
  async personalizeContent(
    template: string,
    customer: CustomerProfile,
    context: PersonalizationContext
  ): Promise<string> {
    // Collect personalization tokens
    const tokens = {
      // Basic info
      customerName: customer.name,
      customerTitle: this.getAppropriateTile(customer),
      
      // Purchase history
      lastProduct: customer.lastPurchase?.productName,
      favoriteCategory: this.analyzeFavoriteCategory(customer.purchases),
      totalPurchases: customer.purchases.length,
      memberSince: this.formatMemberDuration(customer.createdAt),
      
      // Behavioral
      preferredChannel: customer.preferredChannel,
      bestTimeToContact: customer.engagementPatterns?.bestTime,
      
      // Contextual
      currentSeason: this.getCurrentSeason(),
      upcomingEvent: this.getRelevantEvent(customer),
      weatherContext: await this.getWeatherContext(customer.city),
      
      // Dynamic offers
      specialOffer: this.generatePersonalOffer(customer),
      loyaltyPoints: customer.loyaltyPoints || 0,
      nextReward: this.calculateNextReward(customer.loyaltyPoints)
    };
    
    // Smart token replacement
    let personalized = template;
    
    // Basic replacement
    Object.entries(tokens).forEach(([key, value]) => {
      const regex = new RegExp(`{{${key}}}`, 'g');
      personalized = personalized.replace(regex, value?.toString() || '');
    });
    
    // Conditional content
    personalized = this.processConditionals(personalized, customer);
    
    // Dynamic content blocks
    personalized = await this.insertDynamicBlocks(personalized, customer);
    
    return personalized;
  }
  
  /**
   * Smart offer generation based on customer profile
   */
  private generatePersonalOffer(customer: CustomerProfile): string {
    const segment = customer.rfmSegment;
    const daysSinceLastPurchase = customer.daysSinceLastPurchase;
    
    const offerStrategies = {
      champions: () => {
        return `Exclusive 20% untuk koleksi LIMITED EDITION khusus member VIP`;
      },
      
      at_risk: () => {
        const discount = Math.min(30, 15 + Math.floor(daysSinceLastPurchase / 30) * 5);
        return `Special comeback offer ${discount}% karena kami rindu!`;
      },
      
      potential_loyalists: () => {
        if (customer.purchases.length === 2) {
          return `Pembelian ke-3 dapat GRATIS batik scarf senilai 200rb`;
        }
        return `Join member dapat 15% off untuk 3 pembelian berikutnya`;
      },
      
      new_customers: () => {
        return `Welcome gift! 10% off + FREE ongkir untuk pembelian kedua`;
      }
    };
    
    const strategy = offerStrategies[segment] || offerStrategies.new_customers;
    return strategy();
  }
  
  /**
   * Process conditional content blocks
   */
  private processConditionals(
    content: string,
    customer: CustomerProfile
  ): string {
    // IF-THEN-ELSE blocks
    const conditionalRegex = /{{IF (.*?)}}(.*?){{ELSE}}(.*?){{ENDIF}}/gs;
    
    content = content.replace(conditionalRegex, (match, condition, ifBlock, elseBlock) => {
      const evalResult = this.evaluateCondition(condition, customer);
      return evalResult ? ifBlock.trim() : elseBlock.trim();
    });
    
    // Simple IF blocks
    const simpleIfRegex = /{{IF (.*?)}}(.*?){{ENDIF}}/gs;
    
    content = content.replace(simpleIfRegex, (match, condition, ifBlock) => {
      const evalResult = this.evaluateCondition(condition, customer);
      return evalResult ? ifBlock.trim() : '';
    });
    
    return content;
  }
  
  /**
   * Evaluate conditions safely
   */
  private evaluateCondition(condition: string, customer: CustomerProfile): boolean {
    // Parse condition
    const conditions = {
      'isVIP': () => customer.rfmSegment === 'champions',
      'isNewCustomer': () => customer.purchases.length <= 1,
      'hasNotPurchasedRecently': () => customer.daysSinceLastPurchase > 60,
      'isBirthdayMonth': () => {
        if (!customer.birthdate) return false;
        const birthMonth = new Date(customer.birthdate).getMonth();
        return birthMonth === new Date().getMonth();
      },
      'hasHighAOV': () => customer.averageOrderValue > 1_000_000,
      'prefersSocialMedia': () => ['instagram', 'facebook'].includes(customer.preferredChannel),
      'isInJava': () => ['Jakarta', 'Bandung', 'Yogyakarta', 'Surabaya'].includes(customer.city)
    };
    
    // Clean condition string
    const cleanCondition = condition.trim().replace(/['"]/g, '');
    
    // Evaluate
    if (conditions[cleanCondition]) {
      return conditions[cleanCondition]();
    }
    
    // Handle complex conditions
    if (cleanCondition.includes('>')) {
      const [field, value] = cleanCondition.split('>').map(s => s.trim());
      return this.getFieldValue(customer, field) > parseInt(value);
    }
    
    return false;
  }
}
```

## 4.5 Multi-Channel Content Optimization

### Channel-Specific Adaptations

```typescript
export class MultiChannelOptimizer {
  /**
   * Adapt content untuk different channels
   */
  async optimizeForChannel(
    baseContent: GeneratedContent,
    channel: MarketingChannel
  ): Promise<ChannelOptimizedContent> {
    const optimizers = {
      whatsapp: this.optimizeForWhatsApp,
      instagram: this.optimizeForInstagram,
      email: this.optimizeForEmail,
      facebook: this.optimizeForFacebook,
      sms: this.optimizeForSMS
    };
    
    const optimizer = optimizers[channel] || this.defaultOptimizer;
    return await optimizer.call(this, baseContent);
  }
  
  /**
   * WhatsApp optimization
   */
  private async optimizeForWhatsApp(content: GeneratedContent): Promise<ChannelOptimizedContent> {
    return {
      text: this.formatWhatsAppText(content),
      mediaRecommendations: [
        {
          type: 'image',
          specs: 'Max 5MB, JPEG/PNG',
          aiPrompt: 'Product photo dengan good lighting, fokus pada detail batik'
        }
      ],
      formatting: {
        useBold: true,
        useItalic: true,
        useEmoji: true,
        maxLength: 1024
      },
      bestPractices: [
        'Start dengan greeting personal',
        'Keep it conversational',
        'Clear CTA dengan link',
        'Avoid spam trigger words'
      ],
      deliveryRecommendations: {
        bestTime: this.calculateBestTimeForWA(),
        frequency: 'Max 2x per minggu',
        personalizedTiming: true
      }
    };
  }
  
  /**
   * Instagram optimization
   */
  private async optimizeForInstagram(content: GeneratedContent): Promise<ChannelOptimizedContent> {
    // Carousel content generation
    const carouselSlides = await this.generateCarouselContent(content);
    
    return {
      caption: this.formatInstagramCaption(content),
      carouselSlides,
      hashtags: await this.generateOptimalHashtags(content),
      mediaRecommendations: [
        {
          type: 'carousel',
          specs: '1080x1080px atau 1080x1350px',
          slides: carouselSlides.map((slide, index) => ({
            slideNumber: index + 1,
            content: slide.visualDescription,
            textOverlay: slide.textOverlay
          }))
        }
      ],
      formatting: {
        firstLineHook: true,
        useLineBreaks: true,
        emojiPlacement: 'strategic',
        hashtagPlacement: 'end'
      },
      engagementBoosts: [
        'Ask question di caption',
        'Use carousel untuk education',
        'Include user-generated content',
        'Story teaser sebelum post'
      ]
    };
  }
  
  /**
   * Generate Instagram carousel content
   */
  private async generateCarouselContent(
    baseContent: GeneratedContent
  ): Promise<CarouselSlide[]> {
    const prompt = `Create Instagram carousel content:
    Base content: ${baseContent.body}
    
    Create 5 slides:
    1. Hook slide (grab attention)
    2-4. Value slides (educate/inspire)
    5. CTA slide
    
    For each slide provide:
    - Visual description
    - Text overlay (max 50 chars)
    - Design notes`;
    
    const carouselContent = await this.aiService.generate(prompt);
    return this.parseCarouselContent(carouselContent);
  }
  
  /**
   * Email optimization dengan responsive design
   */
  private async optimizeForEmail(content: GeneratedContent): Promise<ChannelOptimizedContent> {
    const emailTemplate = `
      <!DOCTYPE html>
      <html>
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <style>
          /* Responsive styles */
          @media only screen and (max-width: 600px) {
            .container { width: 100% !important; }
            .content { padding: 10px !important; }
          }
        </style>
      </head>
      <body>
        <div class="container">
          <div class="header">
            <img src="{{logoUrl}}" alt="Logo" />
          </div>
          <div class="content">
            {{content}}
          </div>
          <div class="footer">
            {{footer}}
          </div>
        </div>
      </body>
      </html>
    `;
    
    return {
      subject: await this.optimizeEmailSubject(content.headline),
      htmlBody: this.renderEmailTemplate(emailTemplate, content),
      textBody: this.createPlainTextVersion(content),
      preheader: this.generatePreheader(content),
      formatting: {
        template: 'responsive',
        includeImages: true,
        ctaButtons: true
      },
      deliveryOptimization: {
        bestSendTime: await this.calculateOptimalEmailTime(),
        avoidSpamTriggers: true,
        testSubjectLines: this.generateSubjectVariations(content.headline)
      }
    };
  }
}
```

## 4.6 Performance Tracking & Optimization

### Content Performance Analytics

```typescript
export class ContentPerformanceTracker {
  /**
   * Track content performance across channels
   */
  async trackPerformance(
    contentId: string,
    metrics: PerformanceMetrics
  ): Promise<void> {
    const performance = {
      contentId,
      ...metrics,
      timestamp: new Date(),
      
      // Calculate engagement rate
      engagementRate: this.calculateEngagementRate(metrics),
      
      // Calculate conversion rate
      conversionRate: metrics.conversions / metrics.delivered,
      
      // Calculate ROI
      roi: this.calculateROI(metrics),
      
      // Performance score (0-100)
      performanceScore: this.calculatePerformanceScore(metrics)
    };
    
    // Store in database
    await this.db.contentPerformance.create({
      data: performance
    });
    
    // Update AI learning dataset
    await this.updateAILearningData(performance);
    
    // Trigger optimization if needed
    if (performance.performanceScore < 50) {
      await this.triggerContentOptimization(contentId);
    }
  }
  
  /**
   * AI learning from performance data
   */
  private async updateAILearningData(performance: ContentPerformance): Promise<void> {
    // Extract successful patterns
    if (performance.performanceScore > 80) {
      const patterns = await this.extractSuccessPatterns(performance);
      
      // Update prompt templates
      await this.updatePromptTemplates(patterns);
      
      // Fine-tune segment strategies
      await this.updateSegmentStrategies(performance.segment, patterns);
    }
    
    // Learn from failures
    if (performance.performanceScore < 30) {
      const issues = await this.analyzeFailureReasons(performance);
      await this.updateAvoidanceRules(issues);
    }
  }
  
  /**
   * Real-time optimization suggestions
   */
  async getOptimizationSuggestions(
    contentId: string
  ): Promise<OptimizationSuggestion[]> {
    const performance = await this.getContentPerformance(contentId);
    const suggestions: OptimizationSuggestion[] = [];
    
    // Low open rate
    if (performance.openRate < 0.2) {
      suggestions.push({
        issue: 'Low open rate',
        suggestion: 'Try more compelling subject line',
        examples: await this.generateBetterSubjects(performance.content)
      });
    }
    
    // Low click rate
    if (performance.clickRate < 0.05) {
      suggestions.push({
        issue: 'Low click rate',
        suggestion: 'Stronger CTA needed',
        examples: await this.generateBetterCTAs(performance.content)
      });
    }
    
    // High unsubscribe
    if (performance.unsubscribeRate > 0.02) {
      suggestions.push({
        issue: 'High unsubscribe rate',
        suggestion: 'Content may be too promotional',
        adjustment: 'Add more value, less selling'
      });
    }
    
    return suggestions;
  }
}
```

## 4.7 Content Templates & Libraries

### Pre-Built Templates for Quick Start

```typescript
export class ContentTemplateLibrary {
  private templates = {
    seasonal: {
      lebaran: {
        segments: {
          champions: `Selamat Hari Raya Idul Fitri {{customerTitle}} {{customerName}} 🌙
          
          Minal Aidin Wal Faizin. Mohon maaf lahir dan batin.
          
          Sebagai apresiasi untuk loyalitas {{customerTitle}}, kami persembahkan:
          ✨ Koleksi Eksklusif Lebaran dengan 25% off
          ✨ FREE Packaging Cantik untuk hadiah
          ✨ Priority delivery sebelum Lebaran
          
          {{IF isVIP}}
          🎁 BONUS: Batik Scarf Limited Edition untuk pembelian diatas 2jt
          {{ENDIF}}
          
          Lihat koleksi: {{websiteUrl}}/lebaran-exclusive
          
          Selamat merayakan bersama keluarga tercinta 🙏`,
          
          at_risk: `Selamat Lebaran {{customerTitle}} {{customerName}} 🌙
          
          Sudah lama kita tidak bertemu. Di moment spesial ini,
          kami ingin mengucapkan Minal Aidin Wal Faizin.
          
          Tahun ini, kami punya koleksi Lebaran yang special.
          Dan khusus untuk {{customerTitle}}, kami berikan:
          
          🎁 Welcome Back Voucher 30% OFF
          🎁 FREE Ongkir ke seluruh Indonesia
          
          Kami rindu melayani {{customerTitle}} lagi.
          
          Klik disini: {{comebackUrl}}`,
          
          // ... other segments
        }
      },
      
      independence_day: {
        // Templates for 17 Agustus
      },
      
      year_end: {
        // Templates for year end
      }
    },
    
    product_launch: {
      teaser: {
        // Pre-launch teaser templates
      },
      launch: {
        // Launch day templates
      },
      follow_up: {
        // Post-launch templates
      }
    },
    
    educational: {
      motif_meaning: {
        // Templates explaining batik motifs
      },
      care_guide: {
        // How to care for batik
      },
      styling_tips: {
        // How to style batik
      }
    }
  };
  
  /**
   * Get template dengan personalization
   */
  async getTemplate(
    category: string,
    subcategory: string,
    segment: string
  ): Promise<string> {
    const template = this.templates[category]?.[subcategory]?.[segment];
    
    if (!template) {
      // Generate new template if not exists
      return await this.generateNewTemplate(category, subcategory, segment);
    }
    
    return template;
  }
  
  /**
   * Template builder UI helper
   */
  getTemplateStructure(): TemplateStructure {
    return {
      variables: [
        { key: 'customerName', description: 'Nama pelanggan' },
        { key: 'customerTitle', description: 'Bapak/Ibu/Kak' },
        { key: 'lastProduct', description: 'Produk terakhir dibeli' },
        { key: 'loyaltyPoints', description: 'Poin loyalty saat ini' },
        // ... more variables
      ],
      
      conditionals: [
        { key: 'isVIP', description: 'Jika pelanggan VIP' },
        { key: 'isNewCustomer', description: 'Jika pelanggan baru' },
        { key: 'isBirthdayMonth', description: 'Jika bulan ulang tahun' },
        // ... more conditionals
      ],
      
      dynamicBlocks: [
        { key: 'productRecommendations', description: 'Rekomendasi produk otomatis' },
        { key: 'recentTestimonials', description: 'Testimoni terbaru' },
        { key: 'upcomingEvents', description: 'Event mendatang' }
      ]
    };
  }
}
```

## 4.8 Cost Optimization Strategies

### Managing AI Costs for UMKM

```typescript
export class AICostOptimizer {
  private costPerToken = {
    'gpt-4': 0.03 / 1000,           // $0.03 per 1K tokens
    'gpt-4-turbo': 0.01 / 1000,     // $0.01 per 1K tokens
    'gpt-3.5-turbo': 0.002 / 1000   // $0.002 per 1K tokens
  };
  
  /**
   * Smart model selection based on use case
   */
  selectOptimalModel(params: ModelSelectionParams): ModelChoice {
    const { contentType, segment, businessTier } = params;
    
    // High-value content needs better model
    if (segment === 'champions' || segment === 'cant_lose_them') {
      return {
        model: 'gpt-4-turbo',
        reason: 'High-value customer deserves best quality'
      };
    }
    
    // Bulk content can use cheaper model
    if (contentType === 'bulk_whatsapp' || contentType === 'reminder') {
      return {
        model: 'gpt-3.5-turbo',
        reason: 'Simple content, cost optimization'
      };
    }
    
    // Default to balanced option
    return {
      model: 'gpt-4-turbo',
      reason: 'Best balance of quality and cost'
    };
  }
  
  /**
   * Content caching strategy
   */
  async getCachedOrGenerate(params: ContentParams): Promise<GeneratedContent> {
    // Generate cache key
    const cacheKey = this.generateCacheKey(params);
    
    // Check cache first
    const cached = await this.cache.get(cacheKey);
    if (cached && !this.isStale(cached)) {
      return cached.content;
    }
    
    // Check if similar content exists
    const similar = await this.findSimilarContent(params);
    if (similar && this.canReuse(similar, params)) {
      const adapted = await this.adaptContent(similar, params);
      await this.cache.set(cacheKey, adapted);
      return adapted;
    }
    
    // Generate new content
    const fresh = await this.generateFresh(params);
    await this.cache.set(cacheKey, fresh, {
      ttl: this.calculateTTL(params)
    });
    
    return fresh;
  }
  
  /**
   * Batch generation for efficiency
   */
  async batchGenerate(requests: ContentRequest[]): Promise<GeneratedContent[]> {
    // Group by similar parameters
    const groups = this.groupSimilarRequests(requests);
    
    const results: GeneratedContent[] = [];
    
    for (const group of groups) {
      // Generate base content once
      const baseContent = await this.generateBaseContent(group.common);
      
      // Personalize for each request
      const personalized = await Promise.all(
        group.requests.map(req => 
          this.personalizeFromBase(baseContent, req)
        )
      );
      
      results.push(...personalized);
    }
    
    return results;
  }
  
  /**
   * Usage monitoring and alerts
   */
  async monitorUsage(userId: string): Promise<UsageReport> {
    const usage = await this.getMonthlyUsage(userId);
    const budget = await this.getUserBudget(userId);
    
    const report = {
      currentSpend: usage.totalCost,
      budgetRemaining: budget.monthly - usage.totalCost,
      projectedMonthlySpend: this.projectMonthlySpend(usage),
      costPerContent: usage.totalCost / usage.contentGenerated,
      recommendations: []
    };
    
    // Recommendations
    if (report.projectedMonthlySpend > budget.monthly) {
      report.recommendations.push({
        type: 'cost_alert',
        message: 'Projected to exceed budget',
        action: 'Consider using more templates or cached content'
      });
    }
    
    if (report.costPerContent > 0.10) { // $0.10 per content
      report.recommendations.push({
        type: 'optimization',
        message: 'High cost per content',
        action: 'Use batch generation and smarter caching'
      });
    }
    
    return report;
  }
}
```

## 4.9 Integration Examples

### Complete Content Generation Flow

```typescript
// Example: Generate content for weekly campaign
async function generateWeeklyCampaign(userId: string) {
  const contentService = new ContentGenerationService();
  
  try {
    // 1. Get customer segments
    const segments = await getCustomerSegments(userId);
    
    // 2. Prepare generation tasks
    const tasks = segments.map(segment => ({
      segment: segment.name,
      customerCount: segment.count,
      channels: ['whatsapp', 'instagram', 'email'],
      objective: determineObjective(segment.name),
      tone: determineTone(segment.name)
    }));
    
    // 3. Batch generate content
    const generatedContent = await contentService.batchGenerate(tasks);
    
    // 4. Review and approve
    const reviewed = await presentForReview(generatedContent);
    
    // 5. Schedule distribution
    await scheduleDistribution(reviewed, {
      whatsapp: { time: '10:00', days: ['Monday', 'Wednesday'] },
      instagram: { time: '19:00', days: ['Tuesday', 'Thursday'] },
      email: { time: '09:00', days: ['Friday'] }
    });
    
    // 6. Return summary
    return {
      success: true,
      contentGenerated: generatedContent.length,
      estimatedReach: calculateReach(segments),
      scheduledPosts: reviewed.filter(r => r.approved).length
    };
    
  } catch (error) {
    console.error('Campaign generation failed:', error);
    throw error;
  }
}

// Helper functions
function determineObjective(segment: string): string {
  const objectives = {
    champions: 'maintain_loyalty',
    potential_loyalists: 'increase_frequency',
    at_risk: 'win_back',
    new_customers: 'onboarding',
    lost: 'reactivation'
  };
  
  return objectives[segment] || 'general_engagement';
}

function determineTone(segment: string): string {
  const tones = {
    champions: 'exclusive_appreciative',
    potential_loyalists: 'encouraging_friendly',
    at_risk: 'concerned_personal',
    new_customers: 'welcoming_helpful',
    lost: 'nostalgic_incentivizing'
  };
  
  return tones[segment] || 'professional_warm';
}
```

## 4.10 Best Practices & Guidelines

### Do's and Don'ts for AI Content

```typescript
const aiContentGuidelines = {
  dos: [
    {
      practice: "Always review AI output",
      reason: "AI can make mistakes or miss context",
      example: "Check cultural appropriateness, factual accuracy"
    },
    {
      practice: "Provide rich context",
      reason: "Better input = better output",
      example: "Include business details, customer preferences, seasonal context"
    },
    {
      practice: "Use templates as starting point",
      reason: "Consistency and efficiency",
      example: "Build template library from successful content"
    },
    {
      practice: "Track performance religiously",
      reason: "Data drives improvement",
      example: "A/B test different AI-generated variations"
    },
    {
      practice: "Maintain human touch",
      reason: "AI assists, not replaces",
      example: "Add personal anecdotes, specific customer references"
    }
  ],
  
  donts: [
    {
      practice: "Blind trust AI output",
      reason: "Can contain errors or inappropriate content",
      example: "Always human review before sending"
    },
    {
      practice: "Over-automate",
      reason: "Loses authenticity",
      example: "Keep some content fully human-written"
    },
    {
      practice: "Ignore cultural nuances",
      reason: "AI may not understand local context",
      example: "Religious sensitivity, local customs"
    },
    {
      practice: "Use same prompt repeatedly",
      reason: "Content becomes repetitive",
      example: "Vary prompts, add random elements"
    },
    {
      practice: "Forget cost management",
      reason: "AI costs can escalate quickly",
      example: "Monitor usage, use caching"
    }
  ]
};
```

## 4.11 Kesimpulan

AI-Powered Content Generation dalam Smart Marketing Agent bukan sekadar fitur, tapi game-changer untuk mitra UMKM. Dengan kombinasi yang tepat antara AI sophistication dan human touch, mitra UMKM dapat:

1. **Scale personalization** - Personal message untuk ratusan customer
2. **Save time** - Dari jam menjadi menit
3. **Improve quality** - Consistently good content
4. **Learn and adapt** - Semakin pintar over time
5. **Control costs** - Efficient use of AI resources

Yang terpenting, AI membebaskan mitra UMKM untuk fokus pada apa yang mereka do best: creating beautiful batik dan building relationships.

---

*"AI is not about replacing human creativity, but amplifying it. In the hands of UMKM, it becomes a tool for preserving tradition while embracing innovation."*
