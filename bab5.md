# BAB 5
# USER EXPERIENCE

## 5.1 Filosofi Design: Simplicity Meets Intelligence

Smart Marketing Agent didesain dengan prinsip "**Invisible Complexity**" - teknologi canggih yang terasa sederhana. Bayangkan smartphone: di balik layar sentuh yang intuitif, ada jutaan lines of code. Sama halnya dengan Smart Marketing Agent. Mitra UMKM tidak perlu tahu kompleksitas RFM algorithm atau AI processing. Yang mereka lihat adalah dashboard yang clean, tombol yang jelas, dan hasil yang actionable.

### Core Design Principles

**1. UMKM-First Design**
Setiap keputusan design dimulai dengan pertanyaan: "Apakah mitra UMKM yang sibuk bisa langsung paham?"

**2. Progressive Disclosure**
Informasi disajikan bertahap. Basic info di depan, detail bisa di-explore kalau needed.

**3. Action-Oriented**
Setiap screen punya clear next action. No dead ends.

**4. Forgiving Interface**
Mistakes happen. System harus forgiving dengan undo, confirmation, dan clear error messages.

**5. Mobile-First Reality**
70% UMKM access dari HP. Desktop adalah bonus, mobile adalah priority.

## 5.2 Information Architecture

### Site Structure yang Logical

```typescript
// Information Architecture Tree
const siteStructure = {
  root: {
    dashboard: {
      overview: "Snapshot semua yang penting",
      quickActions: "One-click actions",
      alerts: "Urgent items needing attention"
    },
    
    customers: {
      list: {
        allCustomers: "Table view dengan filters",
        segments: "Visual segment distribution",
        search: "Smart search dengan auto-suggest"
      },
      detail: {
        profile: "360° customer view",
        history: "Timeline of interactions",
        actions: "Quick actions untuk customer"
      },
      import: {
        excel: "Upload Excel dengan validation",
        manual: "Form untuk input satu-satu",
        sync: "Integration dengan POS (future)"
      }
    },
    
    transactions: {
      list: "Transaction history dengan filters",
      add: "Quick add transaction",
      bulkImport: "Import batch transactions",
      analytics: "Transaction trends & insights"
    },
    
    analysis: {
      rfm: {
        dashboard: "Visual RFM segments",
        details: "Drill down per segment",
        trends: "Historical segment movement"
      },
      insights: "AI-generated business insights",
      recommendations: "Actionable next steps"
    },
    
    marketing: {
      content: {
        generate: "AI content generator",
        templates: "Saved templates library",
        scheduled: "Content calendar view"
      },
      campaigns: {
        active: "Running campaigns",
        create: "Campaign wizard",
        performance: "Campaign analytics"
      }
    },
    
    settings: {
      profile: "Business profile & branding",
      team: "User management (future)",
      billing: "Subscription & usage",
      integrations: "Third-party connections"
    }
  }
};
```

### Navigation Design

```tsx
// components/navigation/MainNav.tsx
export function MainNav() {
  const pathname = usePathname();
  const [isCollapsed, setIsCollapsed] = useState(false);
  
  const navItems = [
    {
      title: 'Dashboard',
      icon: HomeIcon,
      href: '/dashboard',
      badge: null
    },
    {
      title: 'Pelanggan',
      icon: UsersIcon,
      href: '/customers',
      badge: 'NEW', // Show when new features
      subItems: [
        { title: 'Semua Pelanggan', href: '/customers' },
        { title: 'Segmentasi', href: '/customers/segments' },
        { title: 'Import Data', href: '/customers/import' }
      ]
    },
    {
      title: 'Transaksi',
      icon: ShoppingCartIcon,
      href: '/transactions',
      badge: null
    },
    {
      title: 'Analisis RFM',
      icon: ChartBarIcon,
      href: '/analysis',
      badge: hasNewAnalysis ? 'UPDATED' : null
    },
    {
      title: 'Marketing',
      icon: MegaphoneIcon,
      href: '/marketing',
      subItems: [
        { title: 'Buat Konten', href: '/marketing/content' },
        { title: 'Campaign', href: '/marketing/campaigns' },
        { title: 'Template', href: '/marketing/templates' }
      ]
    }
  ];
  
  return (
    <nav className={cn(
      "flex flex-col space-y-1 transition-all duration-300",
      isCollapsed ? "w-16" : "w-64"
    )}>
      {navItems.map((item) => (
        <NavItem
          key={item.href}
          item={item}
          isActive={pathname.startsWith(item.href)}
          isCollapsed={isCollapsed}
        />
      ))}
    </nav>
  );
}
```

## 5.3 Dashboard Design: The Command Center

### Main Dashboard Layout

```tsx
// app/dashboard/page.tsx
export default function DashboardPage() {
  const { data, isLoading } = useDashboard();
  
  return (
    <div className="space-y-6">
      {/* Welcome Section dengan Actionable Insights */}
      <WelcomeSection />
      
      {/* Key Metrics Cards */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        <MetricCard
          title="Total Pelanggan"
          value={data?.totalCustomers}
          change={data?.customerGrowth}
          icon={UsersIcon}
          href="/customers"
        />
        <MetricCard
          title="Champions"
          value={data?.championCount}
          change={data?.championChange}
          icon={TrophyIcon}
          href="/customers/segments/champions"
          highlight
        />
        <MetricCard
          title="At Risk"
          value={data?.atRiskCount}
          change={data?.atRiskChange}
          icon={ExclamationIcon}
          href="/customers/segments/at-risk"
          alert
        />
        <MetricCard
          title="Revenue MTD"
          value={formatCurrency(data?.revenueMTD)}
          change={data?.revenueGrowth}
          icon={CashIcon}
          href="/transactions"
        />
      </div>
      
      {/* Visual Segments Distribution */}
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2">
          <Card>
            <CardHeader>
              <CardTitle>Distribusi Segmen Pelanggan</CardTitle>
              <CardDescription>
                Update otomatis setiap Senin pagi
              </CardDescription>
            </CardHeader>
            <CardContent>
              <SegmentDonutChart data={data?.segments} />
            </CardContent>
          </Card>
        </div>
        
        <div className="space-y-4">
          <AlertsWidget alerts={data?.alerts} />
          <QuickActionsWidget />
        </div>
      </div>
      
      {/* Recent Activity & Trends */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <RecentTransactions transactions={data?.recentTransactions} />
        <CustomerTrends data={data?.trends} />
      </div>
    </div>
  );
}
```

### Smart Widgets Design

```tsx
// components/dashboard/AlertsWidget.tsx
export function AlertsWidget({ alerts }: { alerts: Alert[] }) {
  if (!alerts?.length) {
    return (
      <Card className="border-green-200 bg-green-50">
        <CardContent className="pt-6">
          <div className="flex items-center space-x-2 text-green-700">
            <CheckCircleIcon className="w-5 h-5" />
            <p className="text-sm font-medium">Semua baik! Tidak ada yang perlu perhatian khusus.</p>
          </div>
        </CardContent>
      </Card>
    );
  }
  
  return (
    <Card>
      <CardHeader>
        <CardTitle className="flex items-center justify-between">
          <span>Perlu Perhatian</span>
          <Badge variant="destructive">{alerts.length}</Badge>
        </CardTitle>
      </CardHeader>
      <CardContent className="space-y-3">
        {alerts.map((alert, index) => (
          <Alert key={index} variant={alert.severity}>
            <AlertIcon className="h-4 w-4" />
            <AlertDescription>
              <p className="font-medium">{alert.title}</p>
              <p className="text-sm mt-1">{alert.message}</p>
              {alert.action && (
                <Button
                  variant="link"
                  size="sm"
                  className="p-0 h-auto mt-2"
                  onClick={() => handleAlertAction(alert)}
                >
                  {alert.action.label} →
                </Button>
              )}
            </AlertDescription>
          </Alert>
        ))}
      </CardContent>
    </Card>
  );
}
```

## 5.4 Customer Management Interface

### Customer List with Smart Filters

```tsx
// app/customers/page.tsx
export default function CustomersPage() {
  const [filters, setFilters] = useState<CustomerFilters>({
    search: '',
    segment: 'all',
    sortBy: 'lastTransaction',
    sortOrder: 'desc'
  });
  
  const { data, isLoading } = useCustomers(filters);
  
  return (
    <div className="space-y-6">
      {/* Page Header with Actions */}
      <div className="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
        <div>
          <h1 className="text-2xl font-bold">Data Pelanggan</h1>
          <p className="text-muted-foreground mt-1">
            {data?.total || 0} pelanggan terdaftar
          </p>
        </div>
        
        <div className="flex gap-2">
          <Button variant="outline" asChild>
            <Link href="/customers/import">
              <UploadIcon className="mr-2 h-4 w-4" />
              Import
            </Link>
          </Button>
          <Button asChild>
            <Link href="/customers/new">
              <PlusIcon className="mr-2 h-4 w-4" />
              Tambah Pelanggan
            </Link>
          </Button>
        </div>
      </div>
      
      {/* Filters Bar */}
      <Card>
        <CardContent className="pt-6">
          <div className="flex flex-col lg:flex-row gap-4">
            {/* Search */}
            <div className="flex-1">
              <div className="relative">
                <SearchIcon className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
                <Input
                  placeholder="Cari nama, nomor HP, atau email..."
                  value={filters.search}
                  onChange={(e) => setFilters({ ...filters, search: e.target.value })}
                  className="pl-9"
                />
              </div>
            </div>
            
            {/* Segment Filter */}
            <Select
              value={filters.segment}
              onValueChange={(value) => setFilters({ ...filters, segment: value })}
            >
              <SelectTrigger className="w-full lg:w-[200px]">
                <SelectValue placeholder="Semua Segmen" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="all">Semua Segmen</SelectItem>
                <SelectItem value="champions">Champions</SelectItem>
                <SelectItem value="loyal_customers">Pelanggan Setia</SelectItem>
                <SelectItem value="at_risk">Berisiko</SelectItem>
                <SelectItem value="new_customers">Pelanggan Baru</SelectItem>
              </SelectContent>
            </Select>
            
            {/* Sort Options */}
            <Select
              value={`${filters.sortBy}-${filters.sortOrder}`}
              onValueChange={(value) => {
                const [sortBy, sortOrder] = value.split('-');
                setFilters({ ...filters, sortBy, sortOrder });
              }}
            >
              <SelectTrigger className="w-full lg:w-[200px]">
                <SelectValue placeholder="Urutkan" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="lastTransaction-desc">Transaksi Terbaru</SelectItem>
                <SelectItem value="totalSpent-desc">Nilai Tertinggi</SelectItem>
                <SelectItem value="frequency-desc">Paling Sering Beli</SelectItem>
                <SelectItem value="name-asc">Nama A-Z</SelectItem>
              </SelectContent>
            </Select>
          </div>
        </CardContent>
      </Card>
      
      {/* Customer Table */}
      <Card>
        <CardContent className="p-0">
          <CustomerTable
            customers={data?.customers}
            isLoading={isLoading}
            onCustomerClick={(customer) => router.push(`/customers/${customer.id}`)}
          />
        </CardContent>
      </Card>
    </div>
  );
}
```

### Customer Detail Page (360° View)

```tsx
// app/customers/[id]/page.tsx
export default function CustomerDetailPage({ params }: { params: { id: string } }) {
  const { data: customer, isLoading } = useCustomer(params.id);
  
  if (isLoading) return <CustomerDetailSkeleton />;
  if (!customer) return <CustomerNotFound />;
  
  return (
    <div className="space-y-6">
      {/* Customer Header */}
      <div className="bg-white rounded-lg border p-6">
        <div className="flex flex-col md:flex-row md:items-center justify-between gap-4">
          <div className="flex items-center gap-4">
            <Avatar className="h-16 w-16">
              <AvatarFallback>{getInitials(customer.name)}</AvatarFallback>
            </Avatar>
            <div>
              <h1 className="text-2xl font-bold">{customer.name}</h1>
              <div className="flex items-center gap-4 mt-1 text-sm text-muted-foreground">
                <span className="flex items-center gap-1">
                  <PhoneIcon className="h-3 w-3" />
                  {customer.phone}
                </span>
                {customer.email && (
                  <span className="flex items-center gap-1">
                    <MailIcon className="h-3 w-3" />
                    {customer.email}
                  </span>
                )}
              </div>
            </div>
          </div>
          
          <div className="flex gap-2">
            <Button variant="outline" size="sm">
              <EditIcon className="mr-2 h-4 w-4" />
              Edit
            </Button>
            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="outline" size="sm">
                  <MoreVerticalIcon className="h-4 w-4" />
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end">
                <DropdownMenuItem onClick={() => sendWhatsApp(customer)}>
                  <MessageSquareIcon className="mr-2 h-4 w-4" />
                  Kirim WhatsApp
                </DropdownMenuItem>
                <DropdownMenuItem onClick={() => sendEmail(customer)}>
                  <MailIcon className="mr-2 h-4 w-4" />
                  Kirim Email
                </DropdownMenuItem>
                <DropdownMenuSeparator />
                <DropdownMenuItem className="text-destructive">
                  <TrashIcon className="mr-2 h-4 w-4" />
                  Hapus
                </DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
          </div>
        </div>
      </div>
      
      {/* Key Metrics */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
        <MetricCard
          title="Segmen"
          value={getSegmentLabel(customer.segment)}
          icon={TagIcon}
          variant={getSegmentVariant(customer.segment)}
        />
        <MetricCard
          title="Total Pembelian"
          value={formatCurrency(customer.totalSpent)}
          icon={CashIcon}
        />
        <MetricCard
          title="Jumlah Transaksi"
          value={customer.transactionCount}
          icon={ShoppingBagIcon}
        />
        <MetricCard
          title="Terakhir Beli"
          value={formatRelativeTime(customer.lastTransaction)}
          icon={CalendarIcon}
        />
      </div>
      
      {/* Tabs for Details */}
      <Tabs defaultValue="timeline" className="space-y-4">
        <TabsList>
          <TabsTrigger value="timeline">Timeline</TabsTrigger>
          <TabsTrigger value="transactions">Transaksi</TabsTrigger>
          <TabsTrigger value="insights">Insights</TabsTrigger>
          <TabsTrigger value="notes">Catatan</TabsTrigger>
        </TabsList>
        
        <TabsContent value="timeline" className="space-y-4">
          <CustomerTimeline customerId={customer.id} />
        </TabsContent>
        
        <TabsContent value="transactions">
          <TransactionHistory customerId={customer.id} />
        </TabsContent>
        
        <TabsContent value="insights">
          <CustomerInsights customer={customer} />
        </TabsContent>
        
        <TabsContent value="notes">
          <CustomerNotes customerId={customer.id} />
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

## 5.5 RFM Analysis Visualization

### Interactive Segment Visualization

```tsx
// components/analysis/RFMSegmentChart.tsx
export function RFMSegmentChart({ data }: { data: SegmentData[] }) {
  const [selectedSegment, setSelectedSegment] = useState<string | null>(null);
  
  // Custom colors untuk setiap segment
  const segmentColors = {
    champions: '#10b981',        // green
    loyal_customers: '#3b82f6',  // blue
    potential_loyalists: '#8b5cf6', // purple
    new_customers: '#f59e0b',    // amber
    promising: '#06b6d4',        // cyan
    need_attention: '#f97316',   // orange
    about_to_sleep: '#ef4444',   // red
    at_risk: '#dc2626',          // red-dark
    cant_lose_them: '#991b1b',   // red-darker
    lost: '#6b7280'              // gray
  };
  
  return (
    <div className="space-y-6">
      {/* Donut Chart */}
      <div className="relative h-[400px]">
        <ResponsiveContainer width="100%" height="100%">
          <PieChart>
            <Pie
              data={data}
              cx="50%"
              cy="50%"
              innerRadius={80}
              outerRadius={140}
              paddingAngle={2}
              dataKey="count"
              animationBegin={0}
              animationDuration={800}
            >
              {data.map((entry, index) => (
                <Cell
                  key={entry.segment}
                  fill={segmentColors[entry.segment]}
                  stroke={selectedSegment === entry.segment ? '#1f2937' : 'none'}
                  strokeWidth={selectedSegment === entry.segment ? 2 : 0}
                  style={{ cursor: 'pointer' }}
                  onClick={() => setSelectedSegment(entry.segment)}
                />
              ))}
            </Pie>
            <Tooltip content={<CustomTooltip />} />
          </PieChart>
        </ResponsiveContainer>
        
        {/* Center Summary */}
        <div className="absolute inset-0 flex items-center justify-center pointer-events-none">
          <div className="text-center">
            <p className="text-3xl font-bold">{data.reduce((sum, d) => sum + d.count, 0)}</p>
            <p className="text-sm text-muted-foreground">Total Pelanggan</p>
          </div>
        </div>
      </div>
      
      {/* Segment Cards */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {data.map((segment) => (
          <SegmentCard
            key={segment.segment}
            segment={segment}
            color={segmentColors[segment.segment]}
            isSelected={selectedSegment === segment.segment}
            onClick={() => setSelectedSegment(segment.segment)}
          />
        ))}
      </div>
      
      {/* Selected Segment Details */}
      {selectedSegment && (
        <SegmentDetailPanel
          segment={data.find(s => s.segment === selectedSegment)}
          onClose={() => setSelectedSegment(null)}
        />
      )}
    </div>
  );
}
```

### RFM Matrix Visualization

```tsx
// components/analysis/RFMMatrix.tsx
export function RFMMatrix({ data }: { data: RFMData[] }) {
  // Create 5x5 matrix for R-F scores
  const matrix = createMatrix(data);
  
  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <h3 className="text-lg font-semibold">RFM Score Matrix</h3>
        <div className="flex items-center gap-4 text-sm">
          <span className="flex items-center gap-2">
            <div className="w-4 h-4 bg-green-500 rounded" />
            Best
          </span>
          <span className="flex items-center gap-2">
            <div className="w-4 h-4 bg-yellow-500 rounded" />
            Medium
          </span>
          <span className="flex items-center gap-2">
            <div className="w-4 h-4 bg-red-500 rounded" />
            Needs Attention
          </span>
        </div>
      </div>
      
      <div className="relative overflow-x-auto">
        <div className="inline-block">
          {/* Y-axis label */}
          <div className="absolute -left-16 top-1/2 -translate-y-1/2 -rotate-90 whitespace-nowrap text-sm font-medium">
            Frequency →
          </div>
          
          {/* Matrix Grid */}
          <div className="grid grid-cols-6 gap-1 p-8">
            {/* Header row */}
            <div /> {/* Empty corner */}
            {[1, 2, 3, 4, 5].map(r => (
              <div key={r} className="text-center text-sm font-medium p-2">
                {r}
              </div>
            ))}
            
            {/* Matrix cells */}
            {[5, 4, 3, 2, 1].map(f => (
              <React.Fragment key={f}>
                <div className="text-center text-sm font-medium p-2">{f}</div>
                {[1, 2, 3, 4, 5].map(r => {
                  const cell = matrix[`${r}-${f}`] || { count: 0, segment: null };
                  return (
                    <MatrixCell
                      key={`${r}-${f}`}
                      recency={r}
                      frequency={f}
                      count={cell.count}
                      segment={cell.segment}
                    />
                  );
                })}
              </React.Fragment>
            ))}
          </div>
          
          {/* X-axis label */}
          <div className="text-center text-sm font-medium mt-2">
            Recency →
          </div>
        </div>
      </div>
    </div>
  );
}
```

## 5.6 Mobile-First Responsive Design

### Adaptive Layouts

```tsx
// hooks/useResponsive.ts
export function useResponsive() {
  const [screen, setScreen] = useState({
    isMobile: false,
    isTablet: false,
    isDesktop: false
  });
  
  useEffect(() => {
    const checkScreen = () => {
      const width = window.innerWidth;
      setScreen({
        isMobile: width < 640,
        isTablet: width >= 640 && width < 1024,
        isDesktop: width >= 1024
      });
    };
    
    checkScreen();
    window.addEventListener('resize', checkScreen);
    return () => window.removeEventListener('resize', checkScreen);
  }, []);
  
  return screen;
}

// components/layouts/ResponsiveLayout.tsx
export function ResponsiveLayout({ children }: { children: React.ReactNode }) {
  const { isMobile } = useResponsive();
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);
  
  return (
    <div className="min-h-screen bg-background">
      {/* Desktop Sidebar */}
      <aside className={cn(
        "fixed left-0 top-0 z-40 h-screen w-64 border-r bg-card",
        "hidden lg:block"
      )}>
        <Sidebar />
      </aside>
      
      {/* Mobile Header */}
      <header className="sticky top-0 z-50 lg:hidden">
        <div className="flex h-16 items-center justify-between border-b bg-background px-4">
          <Logo />
          <Button
            variant="ghost"
            size="icon"
            onClick={() => setMobileMenuOpen(true)}
          >
            <MenuIcon className="h-5 w-5" />
          </Button>
        </div>
      </header>
      
      {/* Mobile Menu Drawer */}
      <MobileDrawer
        open={mobileMenuOpen}
        onClose={() => setMobileMenuOpen(false)}
      >
        <Sidebar mobile />
      </MobileDrawer>
      
      {/* Main Content */}
      <main className={cn(
        "min-h-screen",
        "lg:pl-64" // Account for desktop sidebar
      )}>
        <div className="container mx-auto p-4 lg:p-8">
          {children}
        </div>
      </main>
      
      {/* Mobile Bottom Navigation (Optional) */}
      {isMobile && <MobileBottomNav />}
    </div>
  );
}
```

### Touch-Optimized Components

```tsx
// components/mobile/TouchOptimizedTable.tsx
export function TouchOptimizedTable({ data, columns }: TableProps) {
  const { isMobile } = useResponsive();
  
  if (isMobile) {
    // Card-based layout for mobile
    return (
      <div className="space-y-4">
        {data.map((item, index) => (
          <Card key={index} className="p-4">
            {columns.map((column) => (
              <div key={column.key} className="flex justify-between py-2 border-b last:border-0">
                <span className="text-sm text-muted-foreground">
                  {column.label}
                </span>
                <span className="text-sm font-medium">
                  {column.render ? column.render(item) : item[column.key]}
                </span>
              </div>
            ))}
          </Card>
        ))}
      </div>
    );
  }
  
  // Traditional table for desktop
  return (
    <Table>
      <TableHeader>
        <TableRow>
          {columns.map((column) => (
            <TableHead key={column.key}>{column.label}</TableHead>
          ))}
        </TableRow>
      </TableHeader>
      <TableBody>
        {data.map((item, index) => (
          <TableRow key={index}>
            {columns.map((column) => (
              <TableCell key={column.key}>
                {column.render ? column.render(item) : item[column.key]}
              </TableCell>
            ))}
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}
```

## 5.7 Form Design & Validation

### Smart Form Components

```tsx
// components/forms/CustomerForm.tsx
export function CustomerForm({ 
  initialData, 
  onSubmit 
}: { 
  initialData?: Customer;
  onSubmit: (data: CustomerFormData) => Promise<void>;
}) {
  const form = useForm<CustomerFormData>({
    resolver: zodResolver(customerSchema),
    defaultValues: initialData || {
      name: '',
      phone: '',
      email: '',
      address: '',
      city: '',
      tags: []
    }
  });
  
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleSubmit = async (data: CustomerFormData) => {
    setIsSubmitting(true);
    try {
      await onSubmit(data);
      toast.success(initialData ? 'Pelanggan berhasil diupdate' : 'Pelanggan berhasil ditambahkan');
    } catch (error) {
      toast.error('Terjadi kesalahan. Silakan coba lagi.');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(handleSubmit)} className="space-y-6">
        {/* Name Field */}
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Nama Lengkap</FormLabel>
              <FormControl>
                <Input 
                  {...field} 
                  placeholder="Contoh: Budi Santoso"
                  autoComplete="name"
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        {/* Phone Field with Format Helper */}
        <FormField
          control={form.control}
          name="phone"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Nomor HP</FormLabel>
              <FormControl>
                <PhoneInput
                  {...field}
                  placeholder="08xx-xxxx-xxxx"
                  onChange={(value) => {
                    field.onChange(formatPhoneNumber(value));
                  }}
                />
              </FormControl>
              <FormDescription>
                Format: 08xx-xxxx-xxxx atau +628xx
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />
        
        {/* Email Field (Optional) */}
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>
                Email 
                <span className="text-muted-foreground ml-1">(Opsional)</span>
              </FormLabel>
              <FormControl>
                <Input 
                  {...field} 
                  type="email"
                  placeholder="email@example.com"
                  autoComplete="email"
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        {/* City Selection with Autocomplete */}
        <FormField
          control={form.control}
          name="city"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Kota</FormLabel>
              <FormControl>
                <CityAutocomplete
                  value={field.value}
                  onChange={field.onChange}
                  placeholder="Pilih atau ketik nama kota"
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        {/* Tags Multi-Select */}
        <FormField
          control={form.control}
          name="tags"
          render={({ field }) => (
            <FormItem>
              <FormLabel>
                Tags
                <span className="text-muted-foreground ml-1">(Opsional)</span>
              </FormLabel>
              <FormControl>
                <TagInput
                  value={field.value}
                  onChange={field.onChange}
                  suggestions={['VIP', 'Reseller', 'Corporate', 'Online', 'Offline']}
                  placeholder="Tambah tag..."
                />
              </FormControl>
              <FormDescription>
                Gunakan tags untuk grouping pelanggan
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />
        
        {/* Submit Button */}
        <div className="flex gap-4">
          <Button
            type="submit"
            disabled={isSubmitting}
            className="flex-1 sm:flex-none"
          >
            {isSubmitting && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
            {initialData ? 'Update' : 'Simpan'} Pelanggan
          </Button>
          <Button
            type="button"
            variant="outline"
            onClick={() => router.back()}
          >
            Batal
          </Button>
        </div>
      </form>
    </Form>
  );
}
```

### Inline Validation & Help

```tsx
// components/forms/ValidationHelper.tsx
export function ValidationHelper({ 
  field, 
  validations 
}: { 
  field: string;
  validations: Validation[];
}) {
  const [showHelp, setShowHelp] = useState(false);
  
  return (
    <div className="mt-2">
      <button
        type="button"
        onClick={() => setShowHelp(!showHelp)}
        className="text-xs text-muted-foreground hover:text-foreground flex items-center gap-1"
      >
        <InfoIcon className="h-3 w-3" />
        Format yang benar
      </button>
      
      {showHelp && (
        <div className="mt-2 p-3 bg-muted rounded-md text-sm space-y-1">
          {validations.map((validation, index) => (
            <div key={index} className="flex items-start gap-2">
              {validation.isValid ? (
                <CheckIcon className="h-4 w-4 text-green-600 mt-0.5" />
              ) : (
                <XIcon className="h-4 w-4 text-red-600 mt-0.5" />
              )}
              <span className={validation.isValid ? 'text-green-600' : ''}>
                {validation.message}
              </span>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

## 5.8 Loading States & Skeleton Screens

### Consistent Loading Experience

```tsx
// components/skeletons/DashboardSkeleton.tsx
export function DashboardSkeleton() {
  return (
    <div className="space-y-6 animate-pulse">
      {/* Welcome Section Skeleton */}
      <div className="h-20 bg-muted rounded-lg" />
      
      {/* Metric Cards Skeleton */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        {[...Array(4)].map((_, i) => (
          <div key={i} className="h-32 bg-muted rounded-lg" />
        ))}
      </div>
      
      {/* Chart Section Skeleton */}
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2 h-96 bg-muted rounded-lg" />
        <div className="space-y-4">
          <div className="h-40 bg-muted rounded-lg" />
          <div className="h-40 bg-muted rounded-lg" />
        </div>
      </div>
    </div>
  );
}

// components/loading/LoadingState.tsx
export function LoadingState({ 
  message = "Loading...",
  size = "default" 
}: {
  message?: string;
  size?: "small" | "default" | "large";
}) {
  const sizes = {
    small: "h-4 w-4",
    default: "h-8 w-8",
    large: "h-12 w-12"
  };
  
  return (
    <div className="flex flex-col items-center justify-center p-8">
      <Loader2 className={cn("animate-spin text-primary", sizes[size])} />
      <p className="mt-4 text-sm text-muted-foreground">{message}</p>
    </div>
  );
}
```

## 5.9 Error Handling & Empty States

### Friendly Error Messages

```tsx
// components/errors/ErrorBoundary.tsx
export function ErrorBoundary({ children }: { children: React.ReactNode }) {
  return (
    <ReactErrorBoundary
      fallbackRender={({ error, resetErrorBoundary }) => (
        <div className="min-h-[400px] flex items-center justify-center p-8">
          <Card className="max-w-md w-full">
            <CardHeader>
              <div className="w-12 h-12 rounded-full bg-destructive/10 flex items-center justify-center mb-4">
                <AlertCircleIcon className="h-6 w-6 text-destructive" />
              </div>
              <CardTitle>Oops! Ada yang tidak beres</CardTitle>
              <CardDescription>
                Mohon maaf, terjadi kesalahan yang tidak terduga.
              </CardDescription>
            </CardHeader>
            <CardContent>
              <details className="text-sm text-muted-foreground">
                <summary className="cursor-pointer hover:text-foreground">
                  Detail teknis
                </summary>
                <pre className="mt-2 p-2 bg-muted rounded text-xs overflow-auto">
                  {error.message}
                </pre>
              </details>
            </CardContent>
            <CardFooter className="flex gap-2">
              <Button onClick={resetErrorBoundary}>
                Coba Lagi
              </Button>
              <Button variant="outline" onClick={() => window.location.href = '/dashboard'}>
                Ke Dashboard
              </Button>
            </CardFooter>
          </Card>
        </div>
      )}
    >
      {children}
    </ReactErrorBoundary>
  );
}

// components/empty/EmptyState.tsx
export function EmptyState({
  icon: Icon = InboxIcon,
  title,
  description,
  action
}: EmptyStateProps) {
  return (
    <div className="flex flex-col items-center justify-center p-8 text-center">
      <div className="w-16 h-16 rounded-full bg-muted flex items-center justify-center mb-4">
        <Icon className="h-8 w-8 text-muted-foreground" />
      </div>
      <h3 className="text-lg font-semibold mb-1">{title}</h3>
      <p className="text-sm text-muted-foreground max-w-sm mb-4">
        {description}
      </p>
      {action && (
        <Button
          variant={action.variant || "default"}
          size="sm"
          onClick={action.onClick}
          asChild={action.href ? true : false}
        >
          {action.href ? (
            <Link href={action.href}>
              {action.icon && <action.icon className="mr-2 h-4 w-4" />}
              {action.label}
            </Link>
          ) : (
            <>
              {action.icon && <action.icon className="mr-2 h-4 w-4" />}
              {action.label}
            </>
          )}
        </Button>
      )}
    </div>
  );
}
```

## 5.10 Accessibility & Keyboard Navigation

### Built-in Accessibility Features

```tsx
// components/a11y/AccessibleDataTable.tsx
export function AccessibleDataTable({ data, columns }: DataTableProps) {
  const [focusedRow, setFocusedRow] = useState(-1);
  const tableRef = useRef<HTMLTableElement>(null);
  
  // Keyboard navigation
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (!tableRef.current) return;
      
      const rows = tableRef.current.querySelectorAll('tbody tr');
      const maxIndex = rows.length - 1;
      
      switch (e.key) {
        case 'ArrowDown':
          e.preventDefault();
          setFocusedRow(prev => Math.min(prev + 1, maxIndex));
          break;
        case 'ArrowUp':
          e.preventDefault();
          setFocusedRow(prev => Math.max(prev - 1, 0));
          break;
        case 'Enter':
          if (focusedRow >= 0 && focusedRow <= maxIndex) {
            const row = rows[focusedRow] as HTMLElement;
            row.click();
          }
          break;
      }
    };
    
    if (tableRef.current) {
      tableRef.current.addEventListener('keydown', handleKeyDown);
    }
    
    return () => {
      if (tableRef.current) {
        tableRef.current.removeEventListener('keydown', handleKeyDown);
      }
    };
  }, [focusedRow]);
  
  return (
    <div role="region" aria-label="Data table" className="relative">
      <table
        ref={tableRef}
        role="table"
        tabIndex={0}
        className="w-full"
        aria-rowcount={data.length}
      >
        <thead>
          <tr role="row">
            {columns.map((column) => (
              <th
                key={column.key}
                role="columnheader"
                scope="col"
                className="text-left p-4"
                aria-sort={column.sortable ? 'none' : undefined}
              >
                {column.label}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {data.map((row, index) => (
            <tr
              key={row.id}
              role="row"
              tabIndex={-1}
              aria-rowindex={index + 2}
              className={cn(
                "hover:bg-muted/50 cursor-pointer",
                focusedRow === index && "ring-2 ring-primary"
              )}
              onClick={() => onRowClick?.(row)}
            >
              {columns.map((column) => (
                <td
                  key={column.key}
                  role="cell"
                  className="p-4"
                >
                  {column.render ? column.render(row) : row[column.key]}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
      
      {/* Screen reader announcements */}
      <div className="sr-only" role="status" aria-live="polite">
        {focusedRow >= 0 && `Row ${focusedRow + 1} of ${data.length} selected`}
      </div>
    </div>
  );
}

// Skip to main content link
export function SkipToMain() {
  return (
    <a
      href="#main-content"
      className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 z-50 bg-background px-4 py-2 rounded-md ring-2 ring-primary"
    >
      Skip to main content
    </a>
  );
}
```

## 5.11 User Onboarding Experience

### Interactive Product Tour

```tsx
// components/onboarding/ProductTour.tsx
export function ProductTour() {
  const [tourStep, setTourStep] = useState(0);
  const [showTour, setShowTour] = useState(false);
  
  const tourSteps = [
    {
      target: '[data-tour="dashboard"]',
      title: 'Selamat Datang di Dashboard!',
      content: 'Di sini Anda bisa melihat ringkasan bisnis Anda dalam sekali pandang.',
      placement: 'bottom'
    },
    {
      target: '[data-tour="add-customer"]',
      title: 'Tambah Pelanggan',
      content: 'Klik tombol ini untuk menambah pelanggan baru. Anda juga bisa import dari Excel.',
      placement: 'left'
    },
    {
      target: '[data-tour="segments"]',
      title: 'Segmentasi Otomatis',
      content: 'Sistem akan otomatis mengelompokkan pelanggan Anda berdasarkan perilaku pembelian.',
      placement: 'right'
    },
    {
      target: '[data-tour="generate-content"]',
      title: 'AI Content Generator',
      content: 'Buat konten marketing personal untuk setiap segmen dengan bantuan AI.',
      placement: 'top'
    }
  ];
  
  useEffect(() => {
    // Check if user is new
    const hasSeenTour = localStorage.getItem('hasSeenTour');
    if (!hasSeenTour) {
      setShowTour(true);
    }
  }, []);
  
  const handleNext = () => {
    if (tourStep < tourSteps.length - 1) {
      setTourStep(tourStep + 1);
    } else {
      completeTour();
    }
  };
  
  const completeTour = () => {
    localStorage.setItem('hasSeenTour', 'true');
    setShowTour(false);
    toast.success('Selamat! Anda siap menggunakan Smart Marketing Agent');
  };
  
  if (!showTour) return null;
  
  return (
    <Joyride
      steps={tourSteps}
      stepIndex={tourStep}
      continuous
      showProgress
      showSkipButton
      styles={{
        options: {
          primaryColor: 'hsl(var(--primary))',
          zIndex: 10000,
        }
      }}
      locale={{
        back: 'Kembali',
        close: 'Tutup',
        last: 'Selesai',
        next: 'Lanjut',
        skip: 'Lewati'
      }}
      callback={(data) => {
        if (data.action === 'next') {
          handleNext();
        } else if (data.action === 'skip' || data.action === 'close') {
          completeTour();
        }
      }}
    />
  );
}
```

## 5.12 Performance & Optimization

### Optimized Rendering

```tsx
// hooks/useVirtualization.ts
export function useVirtualizedList<T>({
  items,
  itemHeight,
  containerHeight,
  overscan = 5
}: VirtualizationProps<T>) {
  const [scrollTop, setScrollTop] = useState(0);
  
  const startIndex = Math.max(0, Math.floor(scrollTop / itemHeight) - overscan);
  const endIndex = Math.min(
    items.length - 1,
    Math.ceil((scrollTop + containerHeight) / itemHeight) + overscan
  );
  
  const visibleItems = items.slice(startIndex, endIndex + 1);
  const totalHeight = items.length * itemHeight;
  const offsetY = startIndex * itemHeight;
  
  return {
    visibleItems,
    totalHeight,
    offsetY,
    onScroll: (e: React.UIEvent<HTMLDivElement>) => {
      setScrollTop(e.currentTarget.scrollTop);
    }
  };
}

// components/optimized/VirtualizedCustomerList.tsx
export function VirtualizedCustomerList({ customers }: { customers: Customer[] }) {
  const containerRef = useRef<HTMLDivElement>(null);
  const [containerHeight, setContainerHeight] = useState(600);
  
  const {
    visibleItems,
    totalHeight,
    offsetY,
    onScroll
  } = useVirtualizedList({
    items: customers,
    itemHeight: 80,
    containerHeight
  });
  
  useEffect(() => {
    if (containerRef.current) {
      const resizeObserver = new ResizeObserver((entries) => {
        setContainerHeight(entries[0].contentRect.height);
      });
      
      resizeObserver.observe(containerRef.current);
      return () => resizeObserver.disconnect();
    }
  }, []);
  
  return (
    <div
      ref={containerRef}
      className="h-full overflow-auto"
      onScroll={onScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((customer) => (
            <CustomerRow key={customer.id} customer={customer} />
          ))}
        </div>
      </div>
    </div>
  );
}
```

## 5.13 Key Takeaways

User Experience dalam Smart Marketing Agent dirancang dengan filosofi:

1. **Simplicity First** - Complex technology, simple interface
2. **Mobile Reality** - 70% users on mobile, design accordingly  
3. **Progressive Disclosure** - Show what's needed, when needed
4. **Forgiving Design** - Mistakes happen, make recovery easy
5. **Performance Matters** - Fast response = happy users
6. **Accessibility Built-in** - Inclusive design for all users

Dengan UX yang thoughtful, mitra UMKM dapat fokus pada bisnis mereka, bukan struggling dengan teknologi.

---

*"The best interface is no interface. The best UX is when users don't even think about it - it just works."*
