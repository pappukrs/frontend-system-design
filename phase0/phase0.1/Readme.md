# Phase 0.1: What is Frontend System Design? - Complete Deep Dive Guide

## Table of Contents
1. [Understanding Frontend System Design](#understanding-frontend-system-design)
2. [UI Coding vs System Design](#ui-coding-vs-system-design)
3. [Frontend vs Backend System Design](#frontend-vs-backend-system-design)
4. [Browser Constraints](#browser-constraints)
5. [Trade-offs in Frontend Development](#trade-offs)
6. [Real-World Examples](#real-world-examples)
7. [Best Practices & Day-to-Day Coding](#best-practices)
8. [Common Interview Questions](#interview-questions)

---

## Understanding Frontend System Design

### What is Frontend System Design?

Frontend System Design is the art and science of architecting **scalable, performant, and maintainable** client-side applications that handle complex user interactions, large datasets, and varying network conditions while delivering excellent user experiences.

Unlike backend system design which focuses on servers, databases, and APIs, frontend system design deals with:
- **Client-side architecture** and component organization
- **State management** across the application
- **Performance optimization** within browser constraints
- **User experience** under various conditions
- **Accessibility** for all users
- **Security** in an untrusted environment

### Core Principles

```typescript
// Frontend System Design is about answering questions like:

// 1. ARCHITECTURE
// How do we structure a large application with 100+ components?
// How do we manage shared state across the app?
// How do we handle routing and navigation?

// 2. PERFORMANCE
// How do we load a dashboard with 10,000 items?
// How do we optimize for slow 3G networks?
// How do we minimize Time to Interactive (TTI)?

// 3. SCALABILITY
// How do we build a codebase that 50 engineers can work on?
// How do we add new features without breaking existing ones?
// How do we handle increasing user traffic?

// 4. USER EXPERIENCE
// How do we handle network failures gracefully?
// How do we provide feedback during long operations?
// How do we support accessibility?

// 5. SECURITY
// How do we protect against XSS attacks?
// How do we securely handle authentication tokens?
// How do we prevent CSRF attacks?
```

### Scope of Frontend System Design

```typescript
// What Frontend System Design Covers:

interface FrontendSystemDesignScope {
  // 1. Application Architecture
  architecture: {
    componentStructure: 'atomic design' | 'feature-based' | 'layered';
    stateManagement: 'Redux' | 'Zustand' | 'Context' | 'Jotai';
    routing: 'client-side' | 'server-side' | 'hybrid';
    codeOrganization: 'monolith' | 'micro-frontends';
  };

  // 2. Data Management
  dataLayer: {
    fetching: 'REST' | 'GraphQL' | 'tRPC';
    caching: 'React Query' | 'SWR' | 'Apollo';
    realtime: 'WebSockets' | 'Server-Sent Events' | 'Polling';
    offline: 'IndexedDB' | 'LocalStorage' | 'Service Workers';
  };

  // 3. Performance Optimization
  performance: {
    bundling: 'code splitting' | 'tree shaking' | 'lazy loading';
    rendering: 'CSR' | 'SSR' | 'SSG' | 'ISR';
    images: 'lazy loading' | 'responsive' | 'next-gen formats';
    fonts: 'self-hosted' | 'CDN' | 'variable fonts';
  };

  // 4. User Experience
  ux: {
    loading: 'skeletons' | 'spinners' | 'progressive loading';
    errors: 'retry mechanisms' | 'fallback UI' | 'error boundaries';
    offline: 'offline mode' | 'queue and sync' | 'notifications';
    feedback: 'optimistic updates' | 'toast messages' | 'confirmations';
  };

  // 5. Quality Assurance
  quality: {
    testing: 'unit' | 'integration' | 'e2e' | 'visual regression';
    monitoring: 'errors' | 'performance' | 'analytics';
    accessibility: 'WCAG compliance' | 'keyboard navigation' | 'screen readers';
    security: 'XSS prevention' | 'CSRF protection' | 'CSP headers';
  };
}
```

---

## UI Coding vs System Design

### The Fundamental Difference

```typescript
// ═══════════════════════════════════════════════════════════════
// UI CODING - Building a single component
// ═══════════════════════════════════════════════════════════════

// ❌ UI Coding Interview Question:
// "Build a button component with loading state"

interface ButtonProps {
  onClick: () => void;
  loading?: boolean;
  children: React.ReactNode;
}

function Button({ onClick, loading, children }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={loading}>
      {loading ? <Spinner /> : children}
    </button>
  );
}

// Focus: Implementation details, styling, props API
// Scope: Single component
// Complexity: Low to Medium
// Time: 15-30 minutes

// ═══════════════════════════════════════════════════════════════
// SYSTEM DESIGN - Architecting an application
// ═══════════════════════════════════════════════════════════════

// ✅ System Design Interview Question:
// "Design a news feed like Facebook/Twitter"

interface NewsFeedSystemDesign {
  // 1. Requirements Analysis
  requirements: {
    functional: [
      'Display posts from followed users',
      'Infinite scroll',
      'Real-time updates',
      'Like/comment interactions',
      'Image/video support'
    ];
    nonFunctional: [
      'Support 10M daily active users',
      'Load feed in <2 seconds',
      'Work on slow 3G networks',
      'Offline support',
      'Accessible to screen readers'
    ];
  };

  // 2. High-Level Architecture
  architecture: {
    components: [
      'FeedContainer',
      'PostCard',
      'InfiniteScroller',
      'MediaViewer',
      'CommentSection'
    ];
    stateManagement: 'Redux Toolkit with RTK Query';
    routing: 'React Router with lazy loading';
    styling: 'Tailwind CSS with CSS Modules';
  };

  // 3. Data Flow
  dataFlow: {
    fetching: 'Cursor-based pagination';
    caching: 'Optimistic updates with cache invalidation';
    realtime: 'WebSocket for new posts notification';
    offline: 'IndexedDB for recent posts';
  };

  // 4. Performance Strategy
  performance: {
    rendering: 'Virtual scrolling for long lists';
    images: 'Lazy loading with placeholder';
    bundling: 'Route-based code splitting';
    caching: 'Service Worker for assets';
  };

  // 5. Scalability Considerations
  scalability: {
    codebase: 'Feature-based folder structure';
    team: 'Micro-frontends for different sections';
    deployment: 'CDN with edge caching';
  };
}

// Focus: Architecture, scalability, performance, trade-offs
// Scope: Entire application/feature
// Complexity: High
// Time: 45-60 minutes
```

### Detailed Comparison

```typescript
// ╔═══════════════════════════════════════════════════════════════════════╗
// ║                    UI CODING vs SYSTEM DESIGN                          ║
// ╚═══════════════════════════════════════════════════════════════════════╝

interface ComparisonMatrix {
  uiCoding: {
    description: 'Building individual UI components';
    
    focus: [
      'Component implementation',
      'Props and state management',
      'Styling and layout',
      'Event handling',
      'Basic accessibility'
    ];
    
    questions: [
      'How do I structure this component?',
      'What props should it accept?',
      'How do I handle user interactions?',
      'What styling approach should I use?'
    ];
    
    skills: [
      'HTML/CSS mastery',
      'JavaScript/TypeScript',
      'React/Vue/Angular APIs',
      'Component design patterns',
      'CSS frameworks'
    ];
    
    examples: [
      'Build a dropdown menu',
      'Create a modal dialog',
      'Implement a carousel',
      'Build a form with validation',
      'Create a data table'
    ];
    
    complexity: 'Low to Medium';
    timeframe: '15-30 minutes';
    depth: 'Implementation focused';
  };

  systemDesign: {
    description: 'Architecting scalable frontend applications';
    
    focus: [
      'Application architecture',
      'Data flow and state management',
      'Performance optimization',
      'Scalability planning',
      'Trade-off analysis',
      'Infrastructure decisions'
    ];
    
    questions: [
      'How do we structure the entire application?',
      'How do we handle millions of users?',
      'How do we optimize for performance?',
      'How do we ensure maintainability?',
      'What are the trade-offs of our decisions?',
      'How do we handle failures gracefully?'
    ];
    
    skills: [
      'Software architecture',
      'Performance optimization',
      'Caching strategies',
      'Network protocols',
      'Browser APIs',
      'CDN and deployment',
      'Monitoring and observability',
      'Security best practices'
    ];
    
    examples: [
      'Design Instagram feed',
      'Design Google Docs',
      'Design Netflix video player',
      'Design Slack messaging',
      'Design Uber ride booking'
    ];
    
    complexity: 'High';
    timeframe: '45-60 minutes';
    depth: 'Architecture and strategy focused';
  };
}

// ═══════════════════════════════════════════════════════════════
// PRACTICAL EXAMPLE: Autocomplete Search
// ═══════════════════════════════════════════════════════════════

// UI CODING APPROACH
// "Build an autocomplete input component"

function AutocompleteInput({ options, onChange }: AutocompleteProps) {
  const [query, setQuery] = useState('');
  const [filtered, setFiltered] = useState<string[]>([]);
  const [isOpen, setIsOpen] = useState(false);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value);
    setFiltered(options.filter(opt => opt.toLowerCase().includes(value.toLowerCase())));
    setIsOpen(true);
  };

  return (
    <div className="autocomplete">
      <input value={query} onChange={handleChange} />
      {isOpen && (
        <ul className="suggestions">
          {filtered.map(item => (
            <li key={item} onClick={() => onChange(item)}>
              {item}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

// SYSTEM DESIGN APPROACH
// "Design Google Search autocomplete for millions of users"

interface GoogleAutocompleteSystemDesign {
  // 1. Requirements Clarification
  requirements: {
    scale: '100M searches per day',
    latency: '<100ms for suggestions',
    accuracy: 'Context-aware suggestions',
    features: ['Trending searches', 'Personalized', 'Typo correction'];
  };

  // 2. Architecture Decisions
  architecture: {
    // Client-side
    debouncing: '300ms to reduce API calls',
    caching: {
      strategy: 'LRU cache for recent queries',
      storage: 'IndexedDB for persistent cache',
      ttl: '1 hour for trending, 7 days for static'
    },
    
    // Network optimization
    network: {
      protocol: 'HTTP/2 for multiplexing',
      compression: 'Brotli for payload',
      prefetching: 'Predict next character',
      fallback: 'Offline suggestions from cache'
    },
    
    // Performance
    performance: {
      rendering: 'Virtual scrolling for long lists',
      highlighting: 'Web Workers for text processing',
      lazyLoad: 'Load images only when visible'
    }
  };

  // 3. Data Flow
  dataFlow: {
    input: 'User types "JavaS"',
    debounce: 'Wait 300ms',
    cacheCheck: 'Check IndexedDB for "JavaS*"',
    apiCall: 'GET /suggest?q=JavaS&limit=10',
    response: ['JavaScript', 'Java Spring', 'JavaScript Array'],
    cacheUpdate: 'Store in IndexedDB',
    render: 'Display suggestions with highlighting'
  };

  // 4. Edge Cases
  edgeCases: {
    slowNetwork: 'Show cached results immediately',
    noResults: 'Suggest "Did you mean..." with typo correction',
    rateLimit: 'Implement exponential backoff',
    largeResponse: 'Paginate or limit to top 10',
    specialCharacters: 'Sanitize and escape input'
  };

  // 5. Monitoring
  monitoring: {
    metrics: ['API latency', 'Cache hit rate', 'Suggestion click rate'],
    errors: ['Network failures', 'Timeout errors'],
    analytics: ['Popular searches', 'Zero result queries']
  };
}
```

### Interview Scenario Comparison

```typescript
// ═══════════════════════════════════════════════════════════════
// SAME PRODUCT, DIFFERENT INTERVIEW TYPES
// ═══════════════════════════════════════════════════════════════

// Product: YouTube Video Player

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// UI CODING INTERVIEW
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

// Question: "Build custom video player controls"

interface VideoControlsProps {
  videoRef: React.RefObject<HTMLVideoElement>;
}

function VideoControls({ videoRef }: VideoControlsProps) {
  const [playing, setPlaying] = useState(false);
  const [progress, setProgress] = useState(0);
  const [volume, setVolume] = useState(1);

  const togglePlay = () => {
    if (videoRef.current) {
      if (playing) {
        videoRef.current.pause();
      } else {
        videoRef.current.play();
      }
      setPlaying(!playing);
    }
  };

  const handleProgress = () => {
    if (videoRef.current) {
      const progress = (videoRef.current.currentTime / videoRef.current.duration) * 100;
      setProgress(progress);
    }
  };

  return (
    <div className="controls">
      <button onClick={togglePlay}>
        {playing ? <PauseIcon /> : <PlayIcon />}
      </button>
      <input
        type="range"
        value={progress}
        onChange={(e) => {
          const time = (Number(e.target.value) / 100) * videoRef.current!.duration;
          videoRef.current!.currentTime = time;
        }}
      />
      <VolumeControl volume={volume} setVolume={setVolume} />
    </div>
  );
}

// Expected: Working video controls in 30 minutes
// Evaluation: Code quality, user interactions, edge cases

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// SYSTEM DESIGN INTERVIEW
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

// Question: "Design YouTube video player for millions of concurrent users"

interface YouTubeVideoPlayerSystemDesign {
  // 1. Requirements Clarification (5-10 min)
  requirements: {
    functional: [
      'Play/pause video',
      'Seek to any position',
      'Quality adjustment (360p to 4K)',
      'Subtitles/captions',
      'Picture-in-picture',
      'Theater/fullscreen modes',
      'Playback speed control'
    ],
    nonFunctional: [
      'Support 10M concurrent viewers',
      'Start playback in <3 seconds',
      'Work on 3G networks',
      'Support all major browsers',
      'Minimize buffering',
      'Accessible to screen readers'
    ],
    constraints: [
      'Videos hosted on CDN',
      'Adaptive bitrate streaming',
      'Analytics tracking',
      'Ad insertion points'
    ]
  };

  // 2. High-Level Architecture (10-15 min)
  architecture: {
    clientSide: {
      videoEngine: 'HLS.js or Dash.js for adaptive streaming',
      bufferStrategy: 'Maintain 30s buffer ahead',
      qualitySelection: 'Automatic based on bandwidth + manual override',
      stateManagement: 'Zustand for player state',
    },
    
    dataFlow: {
      initialization: 'Load manifest file → Parse quality levels',
      playback: 'Download segments → Decode → Render',
      adaptation: 'Monitor bandwidth → Switch quality',
      seeking: 'Cancel pending segments → Fetch new segments',
    },
    
    networking: {
      protocol: 'HLS (HTTP Live Streaming)',
      segmentSize: '2-10 seconds',
      parallelDownload: 'Prefetch next 2 segments',
      retryStrategy: 'Exponential backoff with max 3 retries',
    }
  };

  // 3. Deep Dive Areas (15-20 min)
  deepDives: {
    // A. Adaptive Bitrate Streaming
    abr: {
      algorithm: 'Measure bandwidth every 5 seconds',
      switching: {
        up: 'When bandwidth > 1.5x current quality for 10s',
        down: 'Immediately when bandwidth < current quality',
        buffer: 'Switch down if buffer < 10s',
      },
      implementation: `
        class AdaptiveBitrateController {
          private bandwidthHistory: number[] = [];
          private currentQuality: Quality;
          
          selectQuality(availableQualities: Quality[]): Quality {
            const avgBandwidth = this.calculateAverageBandwidth();
            const bufferLevel = this.getBufferLevel();
            
            // Prioritize buffer health
            if (bufferLevel < 10) {
              return this.selectLowerQuality();
            }
            
            // Find best quality within bandwidth
            return availableQualities
              .filter(q => q.bitrate < avgBandwidth * 0.8)
              .sort((a, b) => b.bitrate - a.bitrate)[0];
          }
          
          private calculateAverageBandwidth(): number {
            // Weighted average, recent measurements matter more
            return this.bandwidthHistory
              .slice(-5)
              .reduce((sum, bw, idx) => sum + bw * (idx + 1), 0) 
              / 15;
          }
        }
      `
    },

    // B. Performance Optimization
    performance: {
      preloading: 'Preload metadata on page load',
      thumbnails: 'Load preview thumbnails for seek bar',
      webWorkers: 'Decode segments in worker thread',
      memoryManagement: 'Clear old segments from buffer',
      rendering: 'Use hardware acceleration (WebGL when available)',
    },

    // C. Offline Support
    offline: {
      download: 'Service Worker intercepts video segments',
      storage: 'IndexedDB for segment storage',
      playback: 'Read from IndexedDB when offline',
      sync: 'Background sync for partial downloads',
    },

    // D. Analytics & Monitoring
    monitoring: {
      metrics: [
        'Time to first frame',
        'Rebuffering ratio',
        'Quality switches',
        'Completion rate',
        'Error rate by type'
      ],
      tracking: `
        class VideoAnalytics {
          trackPlaybackEvent(event: PlaybackEvent) {
            sendToAnalytics({
              sessionId: this.sessionId,
              timestamp: Date.now(),
              eventType: event.type,
              quality: this.currentQuality,
              bufferLevel: this.bufferLevel,
              bandwidth: this.estimatedBandwidth,
            });
          }
        }
      `
    }
  };

  // 4. Trade-offs Discussion (5-10 min)
  tradeoffs: {
    bufferSize: {
      large: { pros: 'Less rebuffering', cons: 'More memory, slower startup' },
      small: { pros: 'Quick startup', cons: 'More rebuffering' },
      decision: '30s buffer - balance between UX and resources'
    },
    
    qualitySwitch: {
      aggressive: { pros: 'Best quality always', cons: 'Frequent switches annoy users' },
      conservative: { pros: 'Stable experience', cons: 'Suboptimal quality' },
      decision: 'Conservative with smooth transitions'
    },
    
    segmentSize: {
      large: { pros: 'Fewer requests', cons: 'Slower seeking, more waste' },
      small: { pros: 'Better seeking', cons: 'More overhead' },
      decision: '6s segments - good balance'
    }
  };

  // 5. Scalability & Edge Cases (5 min)
  scalability: {
    cdn: 'Multi-CDN with failover',
    caching: 'Cache segments at edge locations',
    loadBalancing: 'Geographic routing',
  },
  
  edgeCases: {
    slowNetwork: 'Automatically downgrade quality',
    networkLoss: 'Pause and show buffering indicator',
    corruptedSegment: 'Skip to next segment with error tracking',
    unsupportedBrowser: 'Fallback to basic HTML5 video',
    lowMemory: 'Reduce buffer size dynamically',
  };
}

// Expected: Comprehensive design document
// Evaluation: Architecture decisions, trade-off analysis, scalability thinking
```

---

## Frontend vs Backend System Design

### Fundamental Differences

```typescript
// ╔═══════════════════════════════════════════════════════════════════════╗
// ║            FRONTEND vs BACKEND SYSTEM DESIGN                           ║
// ╚═══════════════════════════════════════════════════════════════════════╝

interface SystemDesignComparison {
  frontend: {
    environment: 'Browser (untrusted, limited resources)';
    
    constraints: {
      computation: 'Limited by user device (mobile CPU)';
      memory: '< 100MB for optimal performance';
      storage: 'Limited localStorage/IndexedDB (~50MB)';
      network: 'Variable (3G to 5G), high latency';
      security: 'Code is visible, XSS vulnerable';
    };
    
    focusAreas: [
      'User Experience (UX)',
      'Render performance (60fps)',
      'Bundle size optimization',
      'Perceived performance',
      'Accessibility',
      'Browser compatibility',
      'Offline functionality'
    ];
    
    challenges: {
      stateManagement: 'Distributed state across components',
      dataFetching: 'Race conditions, stale data, cache invalidation',
      performance: 'Main thread blocking, large DOM trees',
      security: 'XSS, CSRF, secure token storage',
      testing: 'UI testing is brittle and slow',
    };
    
    patterns: [
      'Component composition',
      'State management (Redux, Zustand)',
      'Data fetching (React Query, SWR)',
      'Code splitting',
      'Lazy loading',
      'Virtual scrolling',
      'Optimistic updates'
    ];
    
    metrics: {
      performance: ['FCP', 'LCP', 'TTI', 'CLS', 'FID'],
      business: ['Bounce rate', 'Time on page', 'Conversion rate'],
      technical: ['Bundle size', 'API calls', 'Cache hit rate'],
    };
  };

  backend: {
    environment: 'Server (trusted, predictable resources)';
    
    constraints: {
      computation: 'Controlled, can scale horizontally';
      memory: 'GBs available';
      storage: 'TBs available (databases)';
      network: 'Fast, low latency within data center';
      security: 'Code is private, focus on auth/authz';
    };
    
    focusAreas: [
      'Scalability (millions of requests)',
      'Data consistency',
      'Database design',
      'API design',
      'Caching strategy',
      'Load balancing',
      'Fault tolerance'
    ];
    
    challenges: {
      scalability: 'Handle millions of concurrent requests',
      database: 'ACID vs BASE, SQL vs NoSQL',
      consistency: 'CAP theorem, eventual consistency',
      performance: 'Query optimization, N+1 problems',
      security: 'SQL injection, authentication, rate limiting',
    };
    
    patterns: [
      'Microservices',
      'Event-driven architecture',
      'CQRS',
      'Database sharding',
      'Load balancing',
      'Circuit breaker',
      'Message queues'
    ];
    
    metrics: {
      performance: ['Latency', 'Throughput', 'QPS'],
      reliability: ['Uptime', 'Error rate', 'MTTR'],
      scalability: ['Concurrent users', 'DB connections'],
    };
  };
}

// ═══════════════════════════════════════════════════════════════
// PRACTICAL EXAMPLE: E-commerce Product Page
// ═══════════════════════════════════════════════════════════════

// FRONTEND DESIGN CONCERNS
interface FrontendEcommerceDesign {
  // 1. Initial Load Performance
  initialLoad: {
    criticalCSS: 'Inline above-the-fold CSS',
    imageLazyLoad: 'Load hero image first, others on scroll',
    codeSplitting: 'Load checkout code only when needed',
    caching: 'Cache product images in Service Worker',
  };

  // 2. User Interaction
  interactions: {
    addToCart: {
      optimisticUpdate: 'Update UI immediately',
      rollback: 'Revert if API fails',
      feedback: 'Show toast notification',
      animation: 'Cart icon shake + badge update',
    },
    
    quantitySelector: {
      debounce: 'Update after 500ms of inactivity',
      validation: 'Check stock availability',
      feedback: 'Disable button if out of stock',
    },
    
    imageGallery: {
      preload: 'Prefetch next/prev images',
      lazy: 'Load thumbnails first',
      zoom: 'Load high-res on hover',
    },
  };

  // 3. State Management
  state: {
    cart: 'Global state (Zustand)',
    product: 'Server state (React Query)',
    ui: 'Local state (useState)',
    
    syncStrategy: {
      cart: 'Sync to backend + localStorage',
      wishlist: 'Sync across tabs (BroadcastChannel)',
      recentlyViewed: 'Store in IndexedDB',
    },
  };

  // 4. Error Handling
  errors: {
    networkFailure: 'Show cached product data',
    outOfStock: 'Show "Notify me" button',
    paymentFailed: 'Keep cart data, show retry',
  };

  // 5. Accessibility
  a11y: {
    keyboard: 'Tab through all interactive elements',
    screenReader: 'Announce cart updates',
    colorContrast: 'WCAG AA compliance',
    focusManagement: 'Trap focus in modal',
  };
}

// BACKEND DESIGN CONCERNS
interface BackendEcommerceDesign {
  // 1. Scalability
  scalability: {
    loadBalancing: 'Distribute across multiple servers',
    caching: {
      cdn: 'Cache static assets (images, CSS)',
      redis: 'Cache product data (5 min TTL)',
      database: 'Query result cache',
    },
    
    database: {
      readReplicas: '3 read replicas for product queries',
      sharding: 'Shard by product category',
      indexing: 'Index on product_id, category_id',
    },
  };

  // 2. Data Consistency
  consistency: {
    inventory: {
      problem: 'Prevent overselling',
      solution: 'Pessimistic locking + stock reservation',
      implementation: `
        BEGIN TRANSACTION;
        SELECT stock FROM products WHERE id = ? FOR UPDATE;
        IF stock >= quantity THEN
          UPDATE products SET stock = stock - quantity WHERE id = ?;
          INSERT INTO orders ...;
        ELSE
          ROLLBACK;
        END IF;
        COMMIT;
      `,
    },
    
    cart: {
      problem: 'Cart-checkout race condition',
      solution: 'Two-phase commit',
    },
  };

  // 3. API Design
  api: {
    endpoints: {
      '/products/:id': 'Cache 5 min, 1000 req/s',
      '/cart': 'No cache, 500 req/s per user',
      '/checkout': 'Rate limit 5 req/min',
    },
    
    optimization: {
      batching: 'Batch multiple product requests',
      pagination: 'Limit 20 products per page',
      partialResponse: 'Allow field selection (?fields=id,name,price)',
    },
  };

  // 4. Security
  security: {
    authentication: 'JWT with refresh tokens',
    authorization: 'RBAC for admin actions',
    rateLimit: 'Token bucket algorithm',
    validation: 'Sanitize all inputs',
    encryption: 'Encrypt payment info at rest',
  };

  // 5. Monitoring
  monitoring: {
    metrics: ['Request latency', 'Error rate', 'DB connection pool'],
    alerts: ['Error rate > 1%', 'Latency > 500ms', 'Low stock'],
    logging: 'Structured logs to ELK stack',
  };
}
```

### Architecture Comparison

```typescript
// ═══════════════════════════════════════════════════════════════
// ARCHITECTURE PATTERNS: FRONTEND VS BACKEND
// ═══════════════════════════════════════════════════════════════

// FRONTEND: Component-Based Architecture
interface FrontendArchitecture {
  structure: {
    // Hierarchical component tree
    app: {
      layout: {
        header: ['Logo', 'Navigation', 'UserMenu'],
        main: {
          router: {
            '/': 'HomePage',
            '/products': 'ProductList',
            '/product/:id': 'ProductDetail',
            '/cart': 'Cart',
            '/checkout': 'Checkout',
          },
        },
        footer: ['Links', 'SocialMedia'],
      },
    },
  };

  concerns: {
    rendering: 'Virtual DOM diffing, reconciliation',
    stateFlow: 'Unidirectional data flow (props down, events up)',
    sideEffects: 'Managed via hooks (useEffect)',
    codeReuse: 'Higher-order components, hooks, render props',
  };

  example: `
    // Frontend component composition
    function App() {
      return (
        <QueryClientProvider client={queryClient}>
          <Router>
            <Layout>
              <Routes>
                <Route path="/" element={<HomePage />} />
                <Route path="/products" element={<ProductList />} />
              </Routes>
            </Layout>
          </Router>
        </QueryClientProvider>
      );
    }
  `;
}

// BACKEND: Layered Architecture
interface BackendArchitecture {
  structure: {
    // Horizontal layers
    layers: {
      presentation: 'Controllers (handle HTTP requests)',
      business: 'Services (business logic)',
      dataAccess: 'Repositories (database queries)',
      database: 'PostgreSQL/MongoDB',
    },
  };

  concerns: {
    separation: 'Clear separation of concerns',
    dependencies: 'Dependency injection',
    transactions: 'ACID guarantees',
    codeReuse: 'Service layer reusability',
  };

  example: `
    // Backend layered architecture
    
    // Controller Layer
    @Controller('/products')
    class ProductController {
      constructor(private productService: ProductService) {}
      
      @Get('/:id')
      async getProduct(@Param('id') id: string) {
        return this.productService.getProductById(id);
      }
    }
    
    // Service Layer
    class ProductService {
      constructor(private productRepo: ProductRepository) {}
      
      async getProductById(id: string) {
        const product = await this.productRepo.findById(id);
        if (!product) throw new NotFoundException();
        return this.enrichProductData(product);
      }
    }
    
    // Repository Layer
    class ProductRepository {
      async findById(id: string) {
        return this.db.query('SELECT * FROM products WHERE id = $1', [id]);
      }
    }
  `;
}

// ═══════════════════════════════════════════════════════════════
// DATA FLOW: FRONTEND VS BACKEND
// ═══════════════════════════════════════════════════════════════

// FRONTEND DATA FLOW
/*
┌─────────────┐
│   User      │
│   Action    │
└──────┬──────┘
       │
       ↓
┌─────────────────┐
│  Component      │  Optimistic Update
│  (UI Update)    │ ←─────────┐
└────────┬────────┘           │
         │                    │
         ↓                    │
┌─────────────────┐           │
│  State Manager  │           │
│  (Redux/Zustand)│           │
└────────┬────────┘           │
         │                    │
         ↓                    │
┌─────────────────┐           │
│   API Call      │           │
│ (React Query)   │           │
└────────┬────────┘           │
         │                    │
         ↓                    │
┌─────────────────┐           │
│    Backend      │───────────┘
│    Response     │  Success/Error
└─────────────────┘

Key: State updates BEFORE API response (optimistic)
*/

// BACKEND DATA FLOW
/*
┌─────────────┐
│ HTTP Request│
└──────┬──────┘
       │
       ↓
┌─────────────────┐
│  Load Balancer  │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   Auth Middleware│
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   Controller    │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│    Service      │
│  (Business Logic)│
└────────┬────────┘
         │
         ├──→ Cache Check
         │
         ↓
┌─────────────────┐
│   Repository    │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│    Database     │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ HTTP Response   │
└─────────────────┘

Key: Each layer has single responsibility
*/
```

### Key Takeaways

```typescript
// When to think Frontend vs Backend in interviews

interface InterviewApproach {
  frontendFocus: {
    when: [
      'Question mentions "user experience"',
      'Discusses rendering performance',
      'Involves browser APIs',
      'Concerns UI state management',
      'Deals with offline functionality',
      'Focuses on accessibility'
    ],
    
    keyQuestions: [
      'How does this perform on mobile?',
      'What if the network is slow?',
      'How do we handle loading states?',
      'Can users interact during loading?',
      'Is this accessible to screen readers?',
      'How do we optimize bundle size?'
    ],
  };

  backendFocus: {
    when: [
      'Question mentions "millions of users"',
      'Discusses data consistency',
      'Involves database design',
      'Concerns server scalability',
      'Deals with rate limiting',
      'Focuses on security (SQL injection, auth)'
    ],
    
    keyQuestions: [
      'How do we scale to millions of requests?',
      'How do we ensure data consistency?',
      'What database should we use?',
      'How do we prevent race conditions?',
      'How do we handle authentication?',
      'What's our caching strategy?'
    ],
  };

  fullStackBalance: {
    realWorld: `
      In practice, frontend and backend are deeply intertwined:
      
      - API design affects frontend data fetching strategy
      - Backend caching impacts frontend performance
      - Database schema influences frontend state shape
      - Server errors determine frontend error handling
      
      Great engineers understand both and design cohesively.
    `,
  };
}
```

---

## Browser Constraints

### Understanding the Browser Environment

```typescript
// ╔═══════════════════════════════════════════════════════════════════════╗
// ║                    BROWSER CONSTRAINTS                                 ║
// ╚═══════════════════════════════════════════════════════════════════════╝

interface BrowserConstraints {
  // 1. SINGLE-THREADED EXECUTION
  singleThread: {
    description: `
      JavaScript runs on a single main thread (Event Loop).
      Long-running tasks block the UI.
    `,
    
    problem: `
      // ❌ This blocks the UI for 5 seconds
      function processLargeDataset(data) {
        let result = [];
        for (let i = 0; i < 1000000; i++) {
          result.push(heavyComputation(data[i]));
        }
        return result;
      }
      
      // User can't interact with page during this time!
    `,
    
    solution: `
      // ✅ Solution 1: Web Workers (offload to background thread)
      // main.js
      const worker = new Worker('worker.js');
      worker.postMessage(largeDataset);
      worker.onmessage = (e) => {
        displayResults(e.data);
      };
      
      // worker.js
      self.onmessage = (e) => {
        const result = processLargeDataset(e.data);
        self.postMessage(result);
      };
      
      // ✅ Solution 2: Chunking (time-slicing)
      async function processInChunks(data, chunkSize = 100) {
        const results = [];
        for (let i = 0; i < data.length; i += chunkSize) {
          const chunk = data.slice(i, i + chunkSize);
          results.push(...chunk.map(heavyComputation));
          
          // Yield to browser every chunk
          await new Promise(resolve => setTimeout(resolve, 0));
        }
        return results;
      }
      
      // ✅ Solution 3: requestIdleCallback (use idle time)
      function processWhenIdle(data) {
        let index = 0;
        
        function processChunk(deadline) {
          while (deadline.timeRemaining() > 0 && index < data.length) {
            processItem(data[index++]);
          }
          
          if (index < data.length) {
            requestIdleCallback(processChunk);
          }
        }
        
        requestIdleCallback(processChunk);
      }
    `,
    
    impact: {
      goodPractices: [
        'Keep tasks < 50ms to maintain 60fps',
        'Use Web Workers for CPU-intensive tasks',
        'Debounce/throttle frequent operations',
        'Use requestAnimationFrame for animations',
        'Implement virtual scrolling for long lists'
      ],
    },
  };

  // 2. MEMORY LIMITATIONS
  memory: {
    description: `
      Browsers have strict memory limits per tab:
      - Desktop: ~2GB
      - Mobile: ~500MB
      
      Exceeding causes crashes or slowdowns.
    `,
    
    problem: `
      // ❌ Memory leak - event listeners not removed
      function addClickListeners() {
        items.forEach(item => {
          item.addEventListener('click', handleClick);
        });
      }
      
      // When items are re-rendered, old listeners remain!
      // After 100 re-renders = 100x memory usage
      
      // ❌ Memory leak - large cache grows unbounded
      const cache = new Map();
      
      function getCachedData(key) {
        if (!cache.has(key)) {
          cache.set(key, fetchData(key)); // Never cleared!
        }
        return cache.get(key);
      }
    `,
    
    solution: `
      // ✅ Cleanup listeners
      useEffect(() => {
        const handleClick = () => { /* ... */ };
        element.addEventListener('click', handleClick);
        
        return () => {
          element.removeEventListener('click', handleClick);
        };
      }, []);
      
      // ✅ LRU Cache with size limit
      class LRUCache {
        constructor(maxSize = 100) {
          this.cache = new Map();
          this.maxSize = maxSize;
        }
        
        get(key) {
          if (!this.cache.has(key)) return null;
          
          // Move to end (most recently used)
          const value = this.cache.get(key);
          this.cache.delete(key);
          this.cache.set(key, value);
          return value;
        }
        
        set(key, value) {
          this.cache.delete(key);
          this.cache.set(key, value);
          
          // Evict oldest if over capacity
          if (this.cache.size > this.maxSize) {
            const oldestKey = this.cache.keys().next().value;
            this.cache.delete(oldestKey);
          }
        }
      }
      
      // ✅ Cleanup large objects
      function processLargeImage(imageData) {
        const result = process(imageData);
        
        // Clear reference to allow garbage collection
        imageData = null;
        
        return result;
      }
      
      // ✅ Use weak references for caches
      const cache = new WeakMap(); // Auto-cleaned when keys are GC'd
    `,
    
    monitoring: `
      // Monitor memory usage
      if (performance.memory) {
        console.log({
          used: performance.memory.usedJSHeapSize / 1048576,
          total: performance.memory.totalJSHeapSize / 1048576,
          limit: performance.memory.jsHeapSizeLimit / 1048576,
        });
      }
    `,
  };

  // 3. NETWORK CONSTRAINTS
  network: {
    description: `
      Unlike backend (fast, reliable network),
      frontend deals with:
      - Variable latency (50ms to 5000ms)
      - Limited bandwidth (3G: ~400kbps)
      - Intermittent connectivity
      - HTTP/1.1 limit: 6 concurrent connections per domain
    `,
    
    problem: `
      // ❌ Sequential API calls (slow on high latency)
      async function loadUserData() {
        const user = await fetch('/api/user');
        const posts = await fetch('/api/posts'); // Waits for user!
        const friends = await fetch('/api/friends'); // Waits for posts!
        
        // On 200ms latency network = 600ms total
      }
      
      // ❌ Large bundle blocks page load
      import 'lodash'; // 70KB
      import 'moment'; // 289KB
      import 'chart.js'; // 260KB
      // = 619KB to download before app starts
    `,
    
    solution: `
      // ✅ Parallel requests
      async function loadUserData() {
        const [user, posts, friends] = await Promise.all([
          fetch('/api/user'),
          fetch('/api/posts'),
          fetch('/api/friends'),
        ]);
        // On 200ms latency = 200ms total (3x faster!)
      }
      
      // ✅ Request batching
      class BatchedFetcher {
        private queue: string[] = [];
        private timeout: NodeJS.Timeout | null = null;
        
        fetch(id: string): Promise<Data> {
          return new Promise((resolve) => {
            this.queue.push({ id, resolve });
            
            if (!this.timeout) {
              this.timeout = setTimeout(() => this.flush(), 10);
            }
          });
        }
        
        private async flush() {
          const ids = this.queue.map(item => item.id);
          const data = await fetch(\`/api/batch?ids=\${ids.join(',')}\`);
          
          // Resolve all promises
          this.queue.forEach((item, i) => item.resolve(data[i]));
          
          this.queue = [];
          this.timeout = null;
        }
      }
      
      // ✅ Code splitting
      // Instead of:
      // import Chart from 'chart.js';
      
      // Do:
      const Chart = await import('chart.js');
      
      // ✅ Resource hints
      <link rel="preconnect" href="https://api.example.com" />
      <link rel="dns-prefetch" href="https://cdn.example.com" />
      <link rel="preload" href="/critical.css" as="style" />
      
      // ✅ Compression
      // Enable Brotli/Gzip on server
      // 619KB → 150KB (4x smaller!)
    `,
    
    offlineStrategy: `
      // Service Worker for offline support
      self.addEventListener('fetch', (event) => {
        event.respondWith(
          caches.match(event.request).then((response) => {
            // Return cached version if available
            if (response) return response;
            
            // Otherwise fetch from network
            return fetch(event.request).then((response) => {
              // Cache successful responses
              if (response.ok) {
                const cache = await caches.open('v1');
                cache.put(event.request, response.clone());
              }
              return response;
            });
          })
        );
      });
    `,
  };

  // 4. STORAGE LIMITATIONS
  storage: {
    description: `
      Browser storage is limited and user-controlled:
      - localStorage: 5-10MB
      - IndexedDB: 50MB-100MB (can request more)
      - Cache API: ~50MB
      - Cookies: 4KB per cookie
    `,
    
    problem: `
      // ❌ Store too much in localStorage
      const largeData = Array(10000).fill({ /* large object */ });
      localStorage.setItem('data', JSON.stringify(largeData));
      // QuotaExceededError!
      
      // ❌ Cookies for large data
      document.cookie = \`token=\${veryLongJWT}; path=/\`;
      // Exceeds 4KB limit
    `,
    
    solution: `
      // ✅ Use IndexedDB for large data
      const db = await openDB('myDB', 1, {
        upgrade(db) {
          db.createObjectStore('data');
        },
      });
      
      await db.put('data', largeData, 'key');
      
      // ✅ Compression for localStorage
      import pako from 'pako';
      
      function saveCompressed(key, data) {
        const json = JSON.stringify(data);
        const compressed = pako.deflate(json);
        const base64 = btoa(String.fromCharCode(...compressed));
        localStorage.setItem(key, base64);
      }
      
      // ✅ Paginate data storage
      function saveInChunks(data, chunkSize = 1000) {
        for (let i = 0; i < data.length; i += chunkSize) {
          const chunk = data.slice(i, i + chunkSize);
          localStorage.setItem(\`data_chunk_\${i}\`, JSON.stringify(chunk));
        }
      }
      
      // ✅ Monitor storage quota
      if ('storage' in navigator && 'estimate' in navigator.storage) {
        const estimate = await navigator.storage.estimate();
        const percentUsed = (estimate.usage / estimate.quota) * 100;
        
        if (percentUsed > 80) {
          alert('Storage almost full! Please clear cache.');
        }
      }
    `,
  };

  // 5. SECURITY CONSTRAINTS
  security: {
    description: `
      Browser is an untrusted environment:
      - All code is visible to users
      - XSS attacks inject malicious scripts
      - CSRF attacks perform unwanted actions
      - Same-Origin Policy restricts cross-domain requests
    `,
    
    problem: `
      // ❌ XSS vulnerability
      function displayUserComment(comment) {
        document.getElementById('comments').innerHTML = comment;
        // If comment = "<script>stealCookies()</script>"
        // → Code executes!
      }
      
      // ❌ Storing sensitive data in localStorage
      localStorage.setItem('apiKey', 'secret-key-123');
      // Accessible to any script, including XSS attacks!
      
      // ❌ CORS bypass attempt
      fetch('https://api.other-domain.com/data', {
        credentials: 'include' // ❌ Blocked by Same-Origin Policy
      });
    `,
    
    solution: `
      // ✅ Sanitize user input
      import DOMPurify from 'dompurify';
      
      function displayUserComment(comment) {
        const clean = DOMPurify.sanitize(comment);
        document.getElementById('comments').innerHTML = clean;
      }
      
      // ✅ Use textContent for plain text
      element.textContent = userInput; // Safer than innerHTML
      
      // ✅ Content Security Policy
      <meta http-equiv="Content-Security-Policy" 
            content="default-src 'self'; script-src 'self' https://trusted-cdn.com">
      
      // ✅ HttpOnly cookies for sensitive data
      // Set on server:
      Set-Cookie: token=abc123; HttpOnly; Secure; SameSite=Strict
      
      // ✅ CSRF protection
      // Include CSRF token in requests
      fetch('/api/delete', {
        method: 'DELETE',
        headers: {
          'X-CSRF-Token': getCsrfToken(),
        },
      });
      
      // ✅ Proper CORS setup (on server)
      Access-Control-Allow-Origin: https://trusted-domain.com
      Access-Control-Allow-Credentials: true
    `,
  };

  // 6. RENDERING CONSTRAINTS
  rendering: {
    description: `
      Browser rendering is expensive:
      - Reflow (layout recalculation): 10-20ms
      - Repaint (redrawing pixels): 5-10ms
      - Must maintain 60fps = 16.67ms per frame
    `,
    
    problem: `
      // ❌ Forced synchronous layout (layout thrashing)
      elements.forEach(el => {
        const height = el.offsetHeight; // Read (triggers layout)
        el.style.height = height + 10 + 'px'; // Write
        // Each iteration forces layout recalculation!
      });
      
      // ❌ Large DOM tree
      function renderThousandItems(items) {
        return items.map(item => <ItemCard key={item.id} {...item} />);
        // 10,000 items = slow rendering, high memory
      }
    `,
    
    solution: `
      // ✅ Batch reads and writes
      // Read all first
      const heights = elements.map(el => el.offsetHeight);
      
      // Then write all
      elements.forEach((el, i) => {
        el.style.height = heights[i] + 10 + 'px';
      });
      
      // ✅ Virtual scrolling
      import { useVirtualizer } from '@tanstack/react-virtual';
      
      function LargeList({ items }) {
        const parentRef = useRef();
        const virtualizer = useVirtualizer({
          count: items.length,
          getScrollElement: () => parentRef.current,
          estimateSize: () => 50,
        });
        
        return (
          <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
            <div style={{ height: virtualizer.getTotalSize() }}>
              {virtualizer.getVirtualItems().map(virtualRow => (
                <div key={virtualRow.index} style={{
                  position: 'absolute',
                  top: 0,
                  left: 0,
                  width: '100%',
                  height: virtualRow.size,
                  transform: \`translateY(\${virtualRow.start}px)\`,
                }}>
                  {items[virtualRow.index].name}
                </div>
              ))}
            </div>
          </div>
        );
      }
      
      // ✅ CSS containment
      .item {
        contain: layout style paint;
        /* Isolates element from rest of page */
      }
      
      // ✅ Will-change for animations
      .animated-element {
        will-change: transform;
        /* Creates composite layer, GPU-accelerated */
      }
    `,
  };
}
```

### Browser APIs and Their Constraints

```typescript
// ═══════════════════════════════════════════════════════════════
// BROWSER APIs: CAPABILITIES AND LIMITATIONS
// ═══════════════════════════════════════════════════════════════

interface BrowserAPIs {
  // 1. LocalStorage / SessionStorage
  webStorage: {
    capabilities: ['Key-value storage', 'Synchronous API', 'String values only'],
    limitations: ['5-10MB limit', 'Blocks main thread', 'No expiration', 'Same-origin only'],
    
    bestPractices: `
      // ✅ Check availability
      function isStorageAvailable(type) {
        try {
          const storage = window[type];
          const test = '__storage_test__';
          storage.setItem(test, test);
          storage.removeItem(test);
          return true;
        } catch (e) {
          return false;
        }
      }
      
      // ✅ Handle quota exceeded
      function setItem(key, value) {
        try {
          localStorage.setItem(key, value);
        } catch (e) {
          if (e.name === 'QuotaExceededError') {
            // Clear old data or notify user
            clearOldData();
            localStorage.setItem(key, value);
          }
        }
      }
    `,
  };

  // 2. IndexedDB
  indexedDB: {
    capabilities: ['Large storage (~50MB+)', 'Async API', 'Complex queries', 'Transactions'],
    limitations: ['Complex API', 'No server sync', 'Per-origin limit'],
    
    bestPractices: `
      import { openDB } from 'idb';
      
      // ✅ Proper setup with version management
      const db = await openDB('myDB', 2, {
        upgrade(db, oldVersion, newVersion, transaction) {
          if (oldVersion < 1) {
            const store = db.createObjectStore('users', { keyPath: 'id' });
            store.createIndex('email', 'email', { unique: true });
          }
          if (oldVersion < 2) {
            const store = transaction.objectStore('users');
            store.createIndex('createdAt', 'createdAt');
          }
        },
      });
      
      // ✅ Use transactions
      const tx = db.transaction('users', 'readwrite');
      await tx.store.add({ id: 1, name: 'John' });
      await tx.store.add({ id: 2, name: 'Jane' });
      await tx.done; // Commit atomically
    `,
  };

  // 3. Service Workers
  serviceWorkers: {
    capabilities: ['Offline support', 'Background sync', 'Push notifications', 'Intercept requests'],
    limitations: ['HTTPS only', 'Separate thread', 'Limited DOM access', 'Complex lifecycle'],
    
    bestPractices: `
      // ✅ Register with proper scope
      if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('/sw.js', { scope: '/' })
          .then(reg => console.log('SW registered', reg.scope))
          .catch(err => console.error('SW registration failed', err));
      }
      
      // ✅ Cache strategy
      self.addEventListener('fetch', (event) => {
        event.respondWith(
          caches.match(event.request).then((cached) => {
            // Network falling back to cache
            const fetched = fetch(event.request).then((response) => {
              const cache = await caches.open('v1');
              cache.put(event.request, response.clone());
              return response;
            });
            
            return cached || fetched;
          })
        );
      });
    `,
  };

  // 4. Web Workers
  webWorkers: {
    capabilities: ['Background processing', 'No UI blocking', 'Parallel execution'],
    limitations: ['No DOM access', 'Message passing overhead', 'Limited APIs'],
    
    bestPractices: `
      // ✅ Offload heavy computation
      // main.js
      const worker = new Worker('worker.js');
      
      worker.postMessage({ data: largeDataset });
      
      worker.onmessage = (e) => {
        displayResults(e.data);
      };
      
      // worker.js
      self.onmessage = (e) => {
        const result = processData(e.data);
        self.postMessage(result);
      };
      
      // ✅ Terminate when done
      worker.terminate();
    `,
  };

  // 5. WebSockets
  webSockets: {
    capabilities: ['Real-time communication', 'Bidirectional', 'Low latency'],
    limitations: ['Requires server support', 'Not HTTP', 'Connection limits', 'Proxy issues'],
    
    bestPractices: `
      // ✅ Reconnection logic
      class WebSocketClient {
        constructor(url) {
          this.url = url;
          this.reconnectAttempts = 0;
          this.maxReconnectAttempts = 5;
          this.reconnectDelay = 1000;
          this.connect();
        }
        
        connect() {
          this.ws = new WebSocket(this.url);
          
          this.ws.onopen = () => {
            console.log('Connected');
            this.reconnectAttempts = 0;
          };
          
          this.ws.onclose = () => {
            if (this.reconnectAttempts < this.maxReconnectAttempts) {
              setTimeout(() => {
                this.reconnectAttempts++;
                this.connect();
              }, this.reconnectDelay * Math.pow(2, this.reconnectAttempts));
            }
          };
          
          this.ws.onerror = (error) => {
            console.error('WebSocket error:', error);
            this.ws.close();
          };
        }
        
        send(data) {
          if (this.ws.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(data));
          }
        }
      }
    `,
  };
}
```

---

## Trade-offs in Frontend Development

### The Iron Triangle: UX vs Performance vs Security

```typescript
// ╔═══════════════════════════════════════════════════════════════════════╗
// ║                    THE TRADE-OFF TRIANGLE                              ║
// ╚═══════════════════════════════════════════════════════════════════════╝

/*
                   UX (User Experience)
                          △
                         ╱ ╲
                        ╱   ╲
                       ╱     ╲
                      ╱       ╲
                     ╱         ╲
                    ╱           ╲
                   ╱             ╲
                  ╱               ╲
         Performance ◄─────────────► Security

Every decision involves trade-offs between these three pillars.
Optimizing one often means compromising another.
*/

interface TradeOffExamples {
  // ═══════════════════════════════════════════════════════════════
  // 1. UX vs PERFORMANCE
  // ═══════════════════════════════════════════════════════════════
  
  uxVsPerformance: {
    scenario1: {
      name: 'Optimistic UI Updates',
      
      uxFirst: {
        approach: 'Update UI immediately, API call in background',
        code: `
          function addToCart(item) {
            // Update UI first (instant feedback)
            setCartItems(prev => [...prev, item]);
            setCartCount(prev => prev + 1);
            
            // API call in background
            api.addToCart(item).catch(() => {
              // Rollback on failure
              setCartItems(prev => prev.filter(i => i.id !== item.id));
              setCartCount(prev => prev - 1);
              showError('Failed to add to cart');
            });
          }
        `,
        pros: ['Instant feedback', 'Feels fast', 'Better UX'],
        cons: ['Extra state management', 'Rollback complexity', 'Inconsistent state'],
      },
      
      performanceFirst: {
        approach: 'Wait for API response before updating UI',
        code: `
          async function addToCart(item) {
            setLoading(true);
            
            try {
              await api.addToCart(item);
              // Only update after confirmation
              setCartItems(prev => [...prev, item]);
              setCartCount(prev => prev + 1);
            } catch (error) {
              showError('Failed to add to cart');
            } finally {
              setLoading(false);
            }
          }
        `,
        pros: ['Simpler logic', 'Always consistent', 'Fewer edge cases'],
        cons: ['Slower perceived performance', 'UI blocked during API call'],
      },
      
      decision: `
        ✅ Use optimistic updates for:
        - Frequent actions (like, favorite)
        - Low failure rate (<1%)
        - Easy rollback
        
        ✅ Wait for confirmation for:
        - Critical actions (payment, delete)
        - High failure rate
        - Irreversible actions
      `,
    },

    scenario2: {
      name: 'Animations',
      
      uxFirst: {
        approach: 'Rich animations for delightful experience',
        code: `
          // Smooth transitions everywhere
          .card {
            transition: all 0.3s ease-in-out;
            transform: translateY(0);
          }
          
          .card:hover {
            transform: translateY(-10px);
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
          }
          
          // Page transitions
          <AnimatePresence mode="wait">
            <motion.div
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -20 }}
              transition={{ duration: 0.3 }}
            >
              {children}
            </motion.div>
          </AnimatePresence>
        `,
        pros: ['Polished feel', 'Engaging', 'Professional'],
        cons: ['Adds 30-50KB bundle size', 'CPU/GPU intensive', 'Battery drain', 'Janky on low-end devices'],
      },
      
      performanceFirst: {
        approach: 'Minimal animations, optimize for speed',
        code: `
          // Only essential animations
          .button {
            transition: background-color 0.15s; /* Fast, simple */
          }
          
          // Prefer transform/opacity (GPU-accelerated)
          @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
          }
          
          // Use prefers-reduced-motion
          @media (prefers-reduced-motion: reduce) {
            * {
              animation: none !important;
              transition: none !important;
            }
          }
        `,
        pros: ['Fast performance', 'Works on all devices', 'Accessible'],
        cons: ['Less engaging', 'Can feel basic'],
      },
      
      decision: `
        ✅ Rich animations for:
        - Marketing pages
        - Onboarding flows
        - Micro-interactions (button feedback)
        
        ✅ Minimal animations for:
        - Data-heavy dashboards
        - Mobile apps
        - Low-end device support
      `,
    },

    scenario3: {
      name: 'Image Quality',
      
      uxFirst: {
        approach: 'High-quality images for visual appeal',
        code: `
          <img src="/hero-4k.jpg" alt="Hero" />
          <!-- 2.5MB image file -->
        `,
        pros: ['Beautiful', 'Professional', 'Retina-ready'],
        cons: ['Slow loading', 'High bandwidth', 'Poor LCP score'],
      },
      
      performanceFirst: {
        approach: 'Optimized images with multiple formats',
        code: `
          <picture>
            <source 
              srcset="/hero-small.webp 400w,
                      /hero-medium.webp 800w,
                      /hero-large.webp 1200w"
              type="image/webp"
              sizes="(max-width: 600px) 400px,
                     (max-width: 1200px) 800px,
                     1200px"
            />
            <img 
              src="/hero-medium.jpg" 
              alt="Hero"
              loading="lazy"
              width="1200"
              height="600"
            />
          </picture>
          <!-- 50-150KB per image -->
        `,
        pros: ['Fast loading', 'Bandwidth-efficient', 'Better Core Web Vitals'],
        cons: ['More complex', 'Build process overhead', 'Slight quality reduction'],
      },
      
      decision: `
        ✅ Use responsive images with:
        - WebP/AVIF formats
        - Lazy loading
        - Proper sizing
        - CDN with automatic optimization
      `,
    },
  };

  // ═══════════════════════════════════════════════════════════════
  // 2. PERFORMANCE vs SECURITY
  // ═══════════════════════════════════════════════════════════════
  
  performanceVsSecurity: {
    scenario1: {
      name: 'Authentication Tokens',
      
      performanceFirst: {
        approach: 'Store tokens in localStorage for easy access',
        code: `
          // Fast, simple access
          localStorage.setItem('authToken', token);
          
          // Use in API calls
          fetch('/api/data', {
            headers: {
              Authorization: \`Bearer \${localStorage.getItem('authToken')}\`
            }
          });
        `,
        pros: ['Simple implementation', 'No server roundtrip', 'Works across tabs'],
        cons: ['Vulnerable to XSS', 'No HttpOnly protection', 'Can be stolen'],
      },
      
      securityFirst: {
        approach: 'Use HttpOnly cookies',
        code: `
          // Set on server (backend)
          Set-Cookie: authToken=xyz; HttpOnly; Secure; SameSite=Strict
          
          // Automatically included in requests
          fetch('/api/data', {
            credentials: 'include' // Sends cookies
          });
        `,
        pros: ['XSS-proof', 'Secure', 'Industry standard'],
        cons: ['CSRF vulnerability (needs token)', 'No cross-domain', 'Slightly more complex setup'],
      },
      
      decision: `
        ✅ Always use HttpOnly cookies for auth tokens
        + Implement CSRF protection
        + Use short-lived tokens with refresh mechanism
      `,
    },

    scenario2: {
      name: 'Content Security Policy',
      
      performanceFirst: {
        approach: 'Relaxed CSP for flexibility',
        code: `
          Content-Security-Policy: default-src *; script-src *;
          <!-- Allows everything -->
        `,
        pros: ['No blocking', 'Third-party scripts work', 'Fast development'],
        cons: ['No XSS protection', 'Vulnerable to injection'],
      },
      
      securityFirst: {
        approach: 'Strict CSP',
        code: `
          Content-Security-Policy:
            default-src 'self';
            script-src 'self' 'nonce-random123';
            style-src 'self' 'unsafe-inline';
            img-src 'self' https://cdn.example.com;
            connect-src 'self' https://api.example.com;
        `,
        pros: ['Blocks XSS', 'Prevents injection', 'Secure'],
        cons: ['Breaks inline scripts', 'Third-party scripts need whitelisting', 'Development friction'],
      },
      
      decision: `
        ✅ Use strict CSP in production
        ✅ Use nonces for inline scripts
        ✅ Whitelist trusted CDNs only
      `,
    },

    scenario3: {
      name: 'Input Sanitization',
      
      performanceFirst: {
        approach: 'Minimal sanitization',
        code: `
          // Just display as-is
          element.innerHTML = userComment;
        `,
        pros: ['Fast', 'Simple', 'No overhead'],
        cons: ['XSS vulnerable', 'Dangerous'],
      },
      
      securityFirst: {
        approach: 'Full sanitization',
        code: `
          import DOMPurify from 'dompurify';
          
          // Sanitize everything
          const clean = DOMPurify.sanitize(userComment, {
            ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
            ALLOWED_ATTR: ['href'],
          });
          element.innerHTML = clean;
        `,
        pros: ['XSS-proof', 'Safe', 'Configurable'],
        cons: ['Adds 20KB bundle', 'Processing time', 'Potential over-sanitization'],
      },
      
      decision: `
        ✅ Always sanitize user-generated content
        ✅ Use textContent for plain text
        ✅ Whitelist approach for HTML
      `,
    },
  };

  // ═══════════════════════════════════════════════════════════════
  // 3. UX vs SECURITY
  // ═══════════════════════════════════════════════════════════════
  
  uxVsSecurity: {
    scenario1: {
      name: 'Password Requirements',
      
      uxFirst: {
        approach: 'Simple password requirements',
        code: `
          // Minimum 6 characters
          const isValid = password.length >= 6;
        `,
        pros: ['Easy to remember', 'Low friction', 'Higher conversion'],
        cons: ['Weak security', 'Vulnerable to brute force'],
      },
      
      securityFirst: {
        approach: 'Strict password requirements',
        code: `
          // At least 12 chars, uppercase, lowercase, number, special char
          const hasUpperCase = /[A-Z]/.test(password);
          const hasLowerCase = /[a-z]/.test(password);
          const hasNumbers = /\\d/.test(password);
          const hasSpecial = /[!@#$%^&*]/.test(password);
          const isLongEnough = password.length >= 12;
          
          const isValid = hasUpperCase && hasLowerCase && 
                          hasNumbers && hasSpecial && isLongEnough;
        `,
        pros: ['Strong security', 'Brute-force resistant'],
        cons: ['Hard to remember', 'Frustrating UX', 'Lower conversion'],
      },
      
      balanced: {
        approach: 'Passwordless or passkey',
        code: `
          // Use WebAuthn for biometric auth
          const credential = await navigator.credentials.create({
            publicKey: {
              challenge: new Uint8Array(32),
              rp: { name: "Example Corp" },
              user: {
                id: new Uint8Array(16),
                name: "user@example.com",
                displayName: "User Name"
              },
              pubKeyCredParams: [{ alg: -7, type: "public-key" }],
              authenticatorSelection: {
                userVerification: "required"
              }
            }
          });
        `,
        pros: ['Best security', 'Great UX (biometric)', 'Phishing-resistant'],
        cons: ['Browser support', 'Fallback needed'],
      },
      
      decision: `
        ✅ Prefer passwordless (WebAuthn/passkeys)
        ✅ If passwords: 8+ chars + complexity OR long passphrase
        ✅ Add 2FA for sensitive actions
      `,
    },

    scenario2: {
      name: 'Session Timeout',
      
      uxFirst: {
        approach: 'Long session (30 days)',
        code: `
          // Keep user logged in for 30 days
          const expiresIn = 30 * 24 * 60 * 60 * 1000;
        `,
        pros: ['Convenient', 'No frequent re-login', 'Better UX'],
        cons: ['Higher risk if device stolen', 'Compliance issues'],
      },
      
      securityFirst: {
        approach: 'Short session (15 minutes)',
        code: `
          // Auto-logout after 15 min inactivity
          const expiresIn = 15 * 60 * 1000;
        `,
        pros: ['More secure', 'Limits exposure', 'Compliance-friendly'],
        cons: ['Frustrating', 'Lost work', 'Poor UX'],
      },
      
      balanced: {
        approach: 'Sliding session with refresh tokens',
        code: `
          // Access token: 15 min
          // Refresh token: 30 days
          
          const refreshAccessToken = async () => {
            const newToken = await fetch('/auth/refresh', {
              method: 'POST',
              credentials: 'include', // Sends refresh token cookie
            });
            
            return newToken.accessToken;
          };
          
          // Intercept 401 responses
          api.interceptors.response.use(
            response => response,
            async error => {
              if (error.response.status === 401) {
                const newToken = await refreshAccessToken();
                error.config.headers.Authorization = \`Bearer \${newToken}\`;
                return api.request(error.config);
              }
              return Promise.reject(error);
            }
          );
        `,
        pros: ['Good security', 'Seamless UX', 'Automatic refresh'],
        cons: ['More complex', 'Requires backend support'],
      },
      
      decision: `
        ✅ Use short-lived access tokens (15 min)
        ✅ Long-lived refresh tokens (30 days)
        ✅ Automatic background refresh
        ✅ Prompt before important actions even if logged in
      `,
    },

    scenario3: {
      name: 'CAPTCHA',
      
      uxFirst: {
        approach: 'No CAPTCHA',
        code: `
          // Just submit form
          <form onSubmit={handleSubmit}>
            <button>Submit</button>
          </form>
        `,
        pros: ['Smooth UX', 'No friction', 'Fast'],
        cons: ['Spam vulnerable', 'Bot abuse'],
      },
      
      securityFirst: {
        approach: 'Traditional CAPTCHA',
        code: `
          // Google reCAPTCHA v2
          <ReCAPTCHA
            sitekey="your-site-key"
            onChange={setCaptchaToken}
          />
        `,
        pros: ['Blocks bots', 'Proven solution'],
        cons: ['Annoying', 'Accessibility issues', 'Privacy concerns', 'Conversion drop'],
      },
      
      balanced: {
        approach: 'Invisible CAPTCHA or hCaptcha',
        code: `
          // reCAPTCHA v3 (invisible, risk score)
          const token = await grecaptcha.execute('your-site-key', {
            action: 'submit'
          });
          
          // Backend validates and checks score
          const score = await validateRecaptcha(token);
          if (score < 0.5) {
            // Show additional verification
          }
        `,
        pros: ['No user interaction', 'Good UX', 'Still blocks bots'],
        cons: ['Privacy concerns', 'False positives', 'Requires backend'],
      },
      
      decision: `
        ✅ Use invisible CAPTCHA (v3) for most forms
        ✅ Fallback to visible CAPTCHA for low scores
        ✅ Consider hCaptcha for privacy
        ✅ Rate limiting as first line of defense
      `,
    },
  };

  // ═══════════════════════════════════════════════════════════════
  // 4. BALANCING ALL THREE
  // ═══════════════════════════════════════════════════════════════
  
  balancedApproach: {
    framework: `
      When making trade-off decisions, ask:
      
      1. CONTEXT
         - What type of application? (B2B dashboard vs consumer app)
         - What's the user base? (Tech-savvy vs general public)
         - What's the risk level? (Banking vs blog)
      
      2. METRICS
         - What's acceptable performance? (LCP < 2.5s)
         - What's acceptable security level? (OWASP Top 10)
         - What's acceptable UX? (NPS > 50)
      
      3. PRIORITIES
         - What's non-negotiable? (Security for banking, UX for games)
         - What can we compromise? (Some animations, some convenience)
      
      4. ITERATION
         - Start with secure baseline
         - Optimize performance
         - Layer in UX improvements
         - Measure and iterate
    `,
    
    realWorldExample: {
      scenario: 'E-commerce Checkout Flow',
      
      decisions: `
        1. FORM VALIDATION
           ✅ Client-side validation for UX (instant feedback)
           ✅ Server-side validation for security (never trust client)
           ✅ Debounce for performance (avoid excessive API calls)
        
        2. PAYMENT PROCESSING
           ✅ Use payment iframe for security (PCI compliance)
           ✅ Show progress indicator for UX (transparency)
           ✅ Prevent double-submit for reliability
           ❌ Don't optimize away security (use HTTPS, tokenization)
        
        3. SESSION MANAGEMENT
           ✅ 30-min checkout session for UX
           ✅ Re-verify auth before payment for security
           ✅ Save cart in database for reliability
        
        4. ERROR HANDLING
           ✅ Clear error messages for UX
           ✅ Don't leak sensitive info for security
           ✅ Retry logic for reliability
        
        5. PERFORMANCE
           ✅ Lazy load non-critical scripts
           ✅ Preload payment provider SDK
           ✅ Optimize images
           ❌ Don't skip encryption for speed
      `,
    },
  };
}
```

### Decision Framework

```typescript
// ═══════════════════════════════════════════════════════════════
// TRADE-OFF DECISION FRAMEWORK
// ═══════════════════════════════════════════════════════════════

interface TradeOffDecisionFramework {
  step1_identify: {
    description: 'Identify the trade-off',
    questions: [
      'What are we optimizing for?',
      'What are we sacrificing?',
      'Is this trade-off necessary?',
    ],
    example: `
      Trade-off: Adding animations
      Optimizing: User experience
      Sacrificing: Performance (bundle size, CPU)
      Necessary? Depends on application type
    `,
  };

  step2_measure: {
    description: 'Measure the impact',
    metrics: {
      ux: ['Time on page', 'Bounce rate', 'Conversion rate', 'NPS'],
      performance: ['LCP', 'FID', 'CLS', 'Bundle size', 'API latency'],
      security: ['Vulnerability count', 'Incident rate', 'Audit score'],
    },
    example: `
      Before: Bundle 150KB, LCP 2.1s, Conversion 3.2%
      After adding animations: Bundle 200KB, LCP 2.8s, Conversion 3.5%
      
      Trade-off worth it? Conversion up 9%, but LCP slower.
      Decision: Add animations but optimize elsewhere to maintain LCP < 2.5s
    `,
  };

  step3_prioritize: {
    description: 'Prioritize based on context',
    
    contexts: {
      banking: {
        priority: ['Security', 'Performance', 'UX'],
        rationale: 'Security is non-negotiable for financial data',
      },
      socialMedia: {
        priority: ['UX', 'Performance', 'Security'],
        rationale: 'UX drives engagement, but security still important',
      },
      dashboard: {
        priority: ['Performance', 'UX', 'Security'],
        rationale: 'Users need fast data access, internal use reduces security risk',
      },
      ecommerce: {
        priority: ['UX', 'Security', 'Performance'],
        rationale: 'Conversion depends on UX, payment needs security',
      },
    },
  };

  step4_mitigate: {
    description: 'Mitigate the trade-off',
    strategies: [
      'Progressive enhancement (start secure/fast, layer in UX)',
      'Feature flags (enable for subset of users)',
      'A/B testing (measure real impact)',
      'Optimization (find ways to have both)',
      'Fallbacks (degrade gracefully)',
    ],
    example: `
      Trade-off: Rich animations hurt performance on mobile
      
      Mitigation strategies:
      1. Detect device capability
      2. Disable animations on low-end devices
      3. Use CSS animations (GPU-accelerated) not JS
      4. Respect prefers-reduced-motion
      5. Lazy load animation libraries
      
      Code:
      const shouldAnimate = 
        !window.matchMedia('(prefers-reduced-motion: reduce)').matches &&
        navigator.hardwareConcurrency > 4 &&
        !navigator.connection?.saveData;
    `,
  };

  step5_document: {
    description: 'Document the decision',
    template: `
      # Trade-Off Decision: [Feature Name]
      
      ## Context
      - Application: [E-commerce checkout]
      - Users: [General consumers, mobile-first]
      - Date: [2024-02-01]
      
      ## Decision
      We chose [optimistic UI updates] over [wait for API confirmation]
      
      ## Trade-offs
      Optimizing: User experience (instant feedback)
      Sacrificing: Code complexity, potential inconsistency
      
      ## Rationale
      - 95% success rate on add-to-cart API
      - User testing showed 40% drop in perceived latency
      - Easy rollback mechanism
      
      ## Mitigation
      - Implemented error boundary
      - Added rollback logic
      - Log failures to monitor
      - Alert if failure rate > 5%
      
      ## Metrics
      - Conversion rate: 3.2% → 3.5% (↑9%)
      - Perceived speed: 4.2/5 → 4.7/5 (↑12%)
      - Error rate: 0.3% (within threshold)
      
      ## Review Date
      [2024-05-01] - Reassess if error rate increases
    `,
  };
}
```

---

*[Part 2 of this guide will continue in the next message with Real-World Examples, Best Practices, and Interview Questions]*