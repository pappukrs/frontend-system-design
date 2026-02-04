
<!-- 5.2 Deployment Strategies
CSR vs SSR vs SSG
ISR
Edge rendering
Blue-green
Canary releases -->




# 5.2 Deployment Strategies - Complete Deep Dive

**Duration:** 1-1.5 weeks | **Interview Weightage:** Critical for Senior+ roles

---

## Table of Contents
1. [Rendering Strategies (CSR vs SSR vs SSG)](#rendering-strategies)
2. [ISR (Incremental Static Regeneration)](#isr)
3. [Edge Rendering](#edge-rendering)
4. [Blue-Green Deployments](#blue-green)
5. [Canary Releases](#canary-releases)
6. [Interview Questions](#interview-questions)
7. [Daily Best Practices](#best-practices)

---

<a name="rendering-strategies"></a>
## 🔴 Rendering Strategies: CSR vs SSR vs SSG

### **Understanding the Fundamentals**

```
User Request Flow:

CSR (Client-Side Rendering):
Browser → Server (HTML shell) → Browser downloads JS → Renders UI
└─ Fast server, slow initial load, poor SEO

SSR (Server-Side Rendering):
Browser → Server generates HTML → Browser displays → Hydrates
└─ Slow server, fast initial load, great SEO

SSG (Static Site Generation):
Build time → Pre-render HTML → CDN → Browser displays instantly
└─ Fast everything, but content can be stale
```

---

### **✅ CSR (Client-Side Rendering)**

#### **What is CSR?**

The browser downloads a minimal HTML shell, then JavaScript renders the entire UI on the client.

```html
<!-- What the server sends (CSR) -->
<!DOCTYPE html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <div id="root"></div>
    <!-- Empty! -->
    <script src="/bundle.js"></script>
  </body>
</html>

<!-- Browser downloads bundle.js, then renders:
<div id="root">
  <h1>Welcome to My App</h1>
  <p>Content rendered by JavaScript</p>
</div>
-->
```

#### **CSR Implementation**

```javascript
// Standard Create React App (CSR)
// src/index.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);

root.render(<App />);

// App.tsx
function App() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // Data fetched on client side
    fetch('/api/data')
      .then(res => res.json())
      .then(setData);
  }, []);
  
  if (!data) return <div>Loading...</div>;
  
  return <div>{data.content}</div>;
}
```

#### **CSR Pros & Cons**

```typescript
interface RenderingStrategy {
  pros: string[];
  cons: string[];
  useCases: string[];
}

const CSR: RenderingStrategy = {
  pros: [
    '✅ Simple deployment (static files)',
    '✅ Cheap hosting (any CDN)',
    '✅ Rich interactions after load',
    '✅ No server costs',
    '✅ Easy to scale',
  ],
  
  cons: [
    '❌ Slow First Contentful Paint',
    '❌ Poor SEO (if not handled)',
    '❌ Blank screen while loading',
    '❌ Large JavaScript bundle',
    '❌ Poor performance on slow devices',
  ],
  
  useCases: [
    'Admin dashboards',
    'Internal tools',
    'Apps behind authentication',
    'SPAs with minimal SEO needs',
  ],
};
```

#### **Optimizing CSR**

```javascript
// 1. Code Splitting
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}

// 2. Preloading Critical Data
function App() {
  // Start fetching data early
  const dataPromise = useMemo(() => fetchCriticalData(), []);
  
  return <Suspense fallback={<Loading />}>
    <DataRenderer promise={dataPromise} />
  </Suspense>;
}

// 3. App Shell Architecture
// Cache app shell for instant loads
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}

// sw.js - Service Worker
const CACHE_NAME = 'app-shell-v1';
const SHELL_FILES = [
  '/',
  '/index.html',
  '/static/css/main.css',
  '/static/js/main.js',
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(SHELL_FILES))
  );
});
```

---

### **✅ SSR (Server-Side Rendering)**

#### **What is SSR?**

The server generates fully-rendered HTML for each request, then the browser "hydrates" it to make it interactive.

```html
<!-- What the server sends (SSR) -->
<!DOCTYPE html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <div id="root">
      <!-- Fully rendered HTML -->
      <h1>Welcome to My App</h1>
      <p>This content was rendered on the server</p>
      <ul>
        <li>Item 1</li>
        <li>Item 2</li>
      </ul>
    </div>
    <script src="/bundle.js"></script>
  </body>
</html>

<!-- Browser downloads JS and "hydrates" (attaches event listeners) -->
```

#### **SSR Implementation (Next.js)**

```typescript
// pages/index.tsx - Next.js SSR
import { GetServerSideProps } from 'next';

interface HomeProps {
  data: {
    title: string;
    items: string[];
  };
}

export default function Home({ data }: HomeProps) {
  return (
    <div>
      <h1>{data.title}</h1>
      <ul>
        {data.items.map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
    </div>
  );
}

// This runs on EVERY request
export const getServerSideProps: GetServerSideProps<HomeProps> = async (context) => {
  // Fetch data on the server
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  
  // Access request context
  const { req, res: response, query, params } = context;
  
  // Can set headers
  response.setHeader('Cache-Control', 'public, s-maxage=10, stale-while-revalidate=59');
  
  return {
    props: { data },
  };
};
```

#### **Custom SSR Implementation (Express + React)**

```javascript
// server/index.js
import express from 'express';
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter } from 'react-router-dom/server';
import App from '../src/App';

const app = express();

app.get('*', async (req, res) => {
  // 1. Fetch data on server
  const data = await fetchData(req.url);
  
  // 2. Create context for router
  const context = {};
  
  // 3. Render React to string
  const html = renderToString(
    <StaticRouter location={req.url} context={context}>
      <App initialData={data} />
    </StaticRouter>
  );
  
  // 4. Send full HTML
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>My SSR App</title>
        <link rel="stylesheet" href="/styles.css">
      </head>
      <body>
        <div id="root">${html}</div>
        <script>
          window.__INITIAL_DATA__ = ${JSON.stringify(data)};
        </script>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
});

app.listen(3000);

// client/index.js - Hydration
import { hydrateRoot } from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

// Get initial data from server
const initialData = window.__INITIAL_DATA__;

hydrateRoot(
  document.getElementById('root'),
  <BrowserRouter>
    <App initialData={initialData} />
  </BrowserRouter>
);
```

#### **SSR Pros & Cons**

```typescript
const SSR: RenderingStrategy = {
  pros: [
    '✅ Fast First Contentful Paint',
    '✅ Excellent SEO',
    '✅ Works without JavaScript',
    '✅ Better performance on slow devices',
    '✅ Social media previews work',
  ],
  
  cons: [
    '❌ Slower server response (rendering on each request)',
    '❌ Higher server costs',
    '❌ Complex caching strategies',
    '❌ Increased Time to Interactive (TTI)',
    '❌ Server infrastructure required',
  ],
  
  useCases: [
    'E-commerce product pages',
    'News/blog sites',
    'Marketing pages',
    'Content-heavy sites needing SEO',
    'Personalized dashboards',
  ],
};
```

#### **SSR Performance Optimization**

```typescript
// 1. Streaming SSR (React 18)
import { renderToPipeableStream } from 'react-dom/server';

app.get('/', (req, res) => {
  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/bundle.js'],
    onShellReady() {
      res.statusCode = 200;
      res.setHeader('Content-Type', 'text/html');
      pipe(res); // Start streaming immediately
    },
  });
});

// 2. Component-level caching
import { cache } from 'react';

const getCachedData = cache(async (id: string) => {
  const data = await fetch(`/api/data/${id}`);
  return data.json();
});

// 3. Edge caching with stale-while-revalidate
export const getServerSideProps: GetServerSideProps = async ({ res }) => {
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=10, stale-while-revalidate=59'
  );
  
  const data = await fetchData();
  
  return { props: { data } };
};

// 4. Selective Hydration (React 18)
import { Suspense } from 'react';

function App() {
  return (
    <div>
      <Header /> {/* Hydrates immediately */}
      
      <Suspense fallback={<Spinner />}>
        <Comments /> {/* Hydrates when ready */}
      </Suspense>
      
      <Suspense fallback={<Spinner />}>
        <Sidebar /> {/* Hydrates independently */}
      </Suspense>
    </div>
  );
}
```

---

### **✅ SSG (Static Site Generation)**

#### **What is SSG?**

Pages are pre-rendered at **build time**, generating static HTML files that are served from a CDN.

```bash
# Build process (SSG)
npm run build

# Generates static files:
out/
├── index.html          # Pre-rendered
├── about.html          # Pre-rendered
├── blog/
│   ├── post-1.html    # Pre-rendered
│   └── post-2.html    # Pre-rendered
└── _next/
    └── static/

# Deploy to CDN (Vercel, Netlify, CloudFlare)
# Users get instant HTML from CDN edge
```

#### **SSG Implementation (Next.js)**

```typescript
// pages/blog/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface PostProps {
  post: {
    title: string;
    content: string;
    date: string;
  };
}

export default function BlogPost({ post }: PostProps) {
  return (
    <article>
      <h1>{post.title}</h1>
      <time>{post.date}</time>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Runs at BUILD time - generates static paths
export const getStaticPaths: GetStaticPaths = async () => {
  // Fetch all blog post slugs
  const posts = await getAllPosts();
  
  const paths = posts.map((post) => ({
    params: { slug: post.slug },
  }));
  
  return {
    paths,
    fallback: false, // or 'blocking' or true
  };
};

// Runs at BUILD time - generates static props
export const getStaticProps: GetStaticProps<PostProps> = async ({ params }) => {
  const post = await getPostBySlug(params!.slug as string);
  
  return {
    props: { post },
    revalidate: 60, // ISR: Regenerate every 60 seconds
  };
};
```

#### **SSG with Gatsby**

```javascript
// gatsby-node.js
exports.createPages = async ({ graphql, actions }) => {
  const { createPage } = actions;
  
  // Query all blog posts
  const result = await graphql(`
    query {
      allMarkdownRemark {
        edges {
          node {
            frontmatter {
              slug
            }
          }
        }
      }
    }
  `);
  
  // Create pages for each post
  result.data.allMarkdownRemark.edges.forEach(({ node }) => {
    createPage({
      path: `/blog/${node.frontmatter.slug}`,
      component: require.resolve('./src/templates/blog-post.js'),
      context: {
        slug: node.frontmatter.slug,
      },
    });
  });
};

// src/templates/blog-post.js
import React from 'react';
import { graphql } from 'gatsby';

export default function BlogPost({ data }) {
  const { markdownRemark } = data;
  
  return (
    <article>
      <h1>{markdownRemark.frontmatter.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: markdownRemark.html }} />
    </article>
  );
}

export const query = graphql`
  query($slug: String!) {
    markdownRemark(frontmatter: { slug: { eq: $slug } }) {
      html
      frontmatter {
        title
        date
      }
    }
  }
`;
```

#### **SSG Pros & Cons**

```typescript
const SSG: RenderingStrategy = {
  pros: [
    '✅ Fastest possible page loads',
    '✅ Perfect SEO',
    '✅ Cheapest hosting (just CDN)',
    '✅ Infinite scalability',
    '✅ No server needed',
    '✅ Works offline (with service worker)',
  ],
  
  cons: [
    '❌ Build time increases with pages',
    '❌ Content can be stale',
    '❌ Not suitable for personalized content',
    '❌ Need rebuild for content updates',
    '❌ Dynamic routes require pre-generation',
  ],
  
  useCases: [
    'Blogs',
    'Documentation sites',
    'Marketing pages',
    'Portfolio sites',
    'E-commerce product catalogs (with ISR)',
  ],
};
```

---

### **✅ Comparison Matrix**

```typescript
interface StrategyComparison {
  metric: string;
  CSR: string;
  SSR: string;
  SSG: string;
}

const comparison: StrategyComparison[] = [
  {
    metric: 'First Contentful Paint',
    CSR: '❌ Slow (2-5s)',
    SSR: '✅ Fast (<1s)',
    SSG: '✅ Fastest (<0.5s)',
  },
  {
    metric: 'Time to Interactive',
    CSR: '⚠️ Slow (same as FCP)',
    SSR: '⚠️ Moderate (hydration needed)',
    SSG: '✅ Fast (pre-rendered)',
  },
  {
    metric: 'SEO',
    CSR: '❌ Poor (without SSR workarounds)',
    SSR: '✅ Excellent',
    SSG: '✅ Excellent',
  },
  {
    metric: 'Server Cost',
    CSR: '✅ None (static)',
    SSR: '❌ High (per request)',
    SSG: '✅ None (static)',
  },
  {
    metric: 'Hosting Cost',
    CSR: '✅ Very Low (CDN)',
    SSR: '❌ Moderate (server)',
    SSG: '✅ Very Low (CDN)',
  },
  {
    metric: 'Scalability',
    CSR: '✅ Infinite (CDN)',
    SSR: '⚠️ Limited (server capacity)',
    SSG: '✅ Infinite (CDN)',
  },
  {
    metric: 'Content Freshness',
    CSR: '✅ Real-time',
    SSR: '✅ Real-time',
    SSG: '❌ Stale (until rebuild)',
  },
  {
    metric: 'Build Time',
    CSR: '✅ Fast (bundle only)',
    SSR: '✅ Fast (no pre-render)',
    SSG: '❌ Slow (pre-render all)',
  },
  {
    metric: 'Personalization',
    CSR: '✅ Easy',
    SSR: '✅ Easy',
    SSG: '❌ Difficult',
  },
];

// Decision Matrix
function chooseStrategy(requirements: {
  needsSEO: boolean;
  needsPersonalization: boolean;
  trafficVolume: 'low' | 'medium' | 'high';
  contentUpdateFrequency: 'rarely' | 'often' | 'realtime';
  budget: 'low' | 'medium' | 'high';
}) {
  // SEO-critical + static content = SSG
  if (requirements.needsSEO && requirements.contentUpdateFrequency === 'rarely') {
    return 'SSG (or SSG + ISR)';
  }
  
  // SEO-critical + dynamic content = SSR
  if (requirements.needsSEO && requirements.contentUpdateFrequency === 'realtime') {
    return 'SSR';
  }
  
  // No SEO + personalized = CSR
  if (!requirements.needsSEO && requirements.needsPersonalization) {
    return 'CSR';
  }
  
  // High traffic + low budget = SSG/CSR
  if (requirements.trafficVolume === 'high' && requirements.budget === 'low') {
    return 'SSG or CSR';
  }
  
  return 'Hybrid (SSG + CSR)';
}
```

---

### **✅ Hybrid Approaches**

```typescript
// Next.js - Mix strategies per page
// pages/index.tsx - SSG for landing page
export const getStaticProps = async () => {
  return { props: { data: await fetchStaticData() } };
};

// pages/dashboard.tsx - CSR for authenticated dashboard
export default function Dashboard() {
  const { data } = useSWR('/api/user', fetcher);
  return <div>{data?.name}</div>;
}

// pages/product/[id].tsx - SSR for dynamic product pages
export const getServerSideProps = async ({ params }) => {
  return { props: { product: await fetchProduct(params.id) } };
};

// pages/blog/[slug].tsx - SSG + ISR for blog posts
export const getStaticProps = async ({ params }) => {
  return {
    props: { post: await fetchPost(params.slug) },
    revalidate: 60, // ISR
  };
};

// Real-world example: E-commerce site
const ecommerceStrategy = {
  '/': 'SSG', // Home page - static
  '/products': 'SSG + CSR', // Product list - static shell + client-side filtering
  '/product/[id]': 'SSG + ISR', // Product pages - static + revalidation
  '/cart': 'CSR', // Cart - fully client-side
  '/checkout': 'SSR', // Checkout - server-rendered for security
  '/account': 'SSR', // Account - personalized, server-rendered
  '/blog/[slug]': 'SSG', // Blog - fully static
};
```

---

<a name="isr"></a>
## 🔴 ISR (Incremental Static Regeneration)

### **What is ISR?**

ISR combines the best of SSG and SSR: pages are statically generated at build time, but can be **regenerated in the background** without a full rebuild.

```
Traditional SSG:
Build → Generate ALL pages → Deploy → Stale until next build

ISR:
Build → Generate pages → Deploy → Regenerate on-demand/schedule
                                 ↓
                          Fresh content automatically
```

---

### **✅ ISR Implementation**

#### **Basic ISR (Next.js)**

```typescript
// pages/posts/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Post {
  slug: string;
  title: string;
  content: string;
  updatedAt: string;
}

export default function BlogPost({ post }: { post: Post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <time>Last updated: {post.updatedAt}</time>
      <div>{post.content}</div>
    </article>
  );
}

export const getStaticPaths: GetStaticPaths = async () => {
  // Pre-generate most popular posts at build time
  const popularPosts = await getPopularPosts(10);
  
  return {
    paths: popularPosts.map(post => ({ params: { slug: post.slug } })),
    fallback: 'blocking', // Generate other posts on-demand
  };
};

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const post = await getPost(params!.slug as string);
  
  return {
    props: { post },
    revalidate: 60, // ✅ ISR: Regenerate after 60 seconds
  };
};

// How it works:
// 1. First request after 60s triggers background regeneration
// 2. User still sees old page (stale-while-revalidate)
// 3. Next user sees fresh content
```

#### **On-Demand ISR (Next.js 12.2+)**

```typescript
// pages/api/revalidate.ts - API route to trigger revalidation
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // Verify secret to prevent unauthorized revalidation
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }
  
  try {
    // Revalidate specific path
    await res.revalidate(`/posts/${req.query.slug}`);
    
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// Webhook from CMS triggers revalidation
// POST /api/revalidate?slug=my-post&secret=xxx
```

#### **CMS Integration with ISR**

```typescript
// Contentful webhook → Next.js revalidation
// pages/api/contentful-webhook.ts
import { NextApiRequest, NextApiResponse } from 'next';
import crypto from 'crypto';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // Verify Contentful signature
  const signature = req.headers['x-contentful-signature'];
  const body = JSON.stringify(req.body);
  
  const hash = crypto
    .createHmac('sha256', process.env.CONTENTFUL_WEBHOOK_SECRET!)
    .update(body)
    .digest('base64');
  
  if (signature !== hash) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Parse webhook payload
  const { sys, fields } = req.body;
  const contentType = sys.contentType.sys.id;
  
  // Revalidate based on content type
  if (contentType === 'blogPost') {
    const slug = fields.slug['en-US'];
    await res.revalidate(`/blog/${slug}`);
    await res.revalidate('/blog'); // Also revalidate blog index
  }
  
  if (contentType === 'product') {
    const id = fields.id['en-US'];
    await res.revalidate(`/product/${id}`);
    await res.revalidate('/products');
  }
  
  return res.json({ revalidated: true });
}
```

---

### **✅ ISR Fallback Strategies**

```typescript
// pages/posts/[slug].tsx
export const getStaticPaths: GetStaticPaths = async () => {
  return {
    paths: [
      { params: { slug: 'popular-post-1' } },
      { params: { slug: 'popular-post-2' } },
    ],
    fallback: 'blocking', // Three options: false, true, 'blocking'
  };
};

// OPTION 1: fallback: false
// - Only pre-generated paths exist
// - Other paths return 404
// - Use when: Small, known set of pages
fallback: false

// OPTION 2: fallback: true
// - Show loading state while generating
// - User sees skeleton/spinner
// - Page cached after first generation
fallback: true

export default function BlogPost({ post }: { post: Post }) {
  const router = useRouter();
  
  // Show loading state for fallback
  if (router.isFallback) {
    return <LoadingSkeleton />;
  }
  
  return <article>{post.content}</article>;
}

// OPTION 3: fallback: 'blocking'
// - Wait for page generation (like SSR)
// - No loading state needed
// - Slower first request, but simpler
fallback: 'blocking'

// Use when: Better UX, no loading flicker
```

---

### **✅ ISR Best Practices**

```typescript
// 1. Smart revalidation intervals
const getRevalidateTime = (contentType: string) => {
  const intervals = {
    'blog-post': 3600,        // 1 hour (rarely changes)
    'product-price': 60,      // 1 minute (changes often)
    'homepage': 300,          // 5 minutes (moderate changes)
    'documentation': 86400,   // 24 hours (very stable)
  };
  
  return intervals[contentType] || 600; // Default 10 minutes
};

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const post = await getPost(params!.slug as string);
  
  return {
    props: { post },
    revalidate: getRevalidateTime('blog-post'),
  };
};

// 2. Cascade revalidation
// pages/api/revalidate-blog.ts
export default async function handler(req, res) {
  const { slug } = req.query;
  
  // Revalidate post page
  await res.revalidate(`/blog/${slug}`);
  
  // Also revalidate related pages
  await res.revalidate('/blog'); // Index page
  await res.revalidate('/'); // Homepage (if it shows recent posts)
  await res.revalidate(`/author/${post.authorId}`); // Author page
  
  return res.json({ revalidated: true });
}

// 3. Conditional revalidation
export const getStaticProps: GetStaticProps = async ({ params }) => {
  const post = await getPost(params!.slug as string);
  
  // Don't revalidate if post is archived
  const revalidate = post.status === 'archived' ? false : 60;
  
  return {
    props: { post },
    revalidate,
  };
};

// 4. Error handling
export const getStaticProps: GetStaticProps = async ({ params }) => {
  try {
    const post = await getPost(params!.slug as string);
    
    return {
      props: { post },
      revalidate: 60,
    };
  } catch (error) {
    // Return 404 if post doesn't exist
    if (error.statusCode === 404) {
      return { notFound: true };
    }
    
    // Return old cached version if API fails
    return {
      props: { post: null },
      revalidate: 10, // Retry sooner
    };
  }
};
```

---

### **✅ ISR Monitoring & Analytics**

```typescript
// utils/isr-analytics.ts
export async function trackISRRegeneration(
  page: string,
  success: boolean,
  duration: number
) {
  await fetch('/api/analytics/isr', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      page,
      success,
      duration,
      timestamp: Date.now(),
    }),
  });
}

// pages/posts/[slug].tsx
export const getStaticProps: GetStaticProps = async ({ params }) => {
  const startTime = Date.now();
  
  try {
    const post = await getPost(params!.slug as string);
    const duration = Date.now() - startTime;
    
    // Track successful regeneration
    await trackISRRegeneration(`/posts/${params!.slug}`, true, duration);
    
    return {
      props: { post },
      revalidate: 60,
    };
  } catch (error) {
    const duration = Date.now() - startTime;
    await trackISRRegeneration(`/posts/${params!.slug}`, false, duration);
    
    throw error;
  }
};

// Dashboard to monitor ISR health
// pages/admin/isr-stats.tsx
export default function ISRStats({ stats }) {
  return (
    <div>
      <h1>ISR Statistics</h1>
      <table>
        <thead>
          <tr>
            <th>Page</th>
            <th>Regenerations (24h)</th>
            <th>Success Rate</th>
            <th>Avg Duration</th>
          </tr>
        </thead>
        <tbody>
          {stats.map(stat => (
            <tr key={stat.page}>
              <td>{stat.page}</td>
              <td>{stat.count}</td>
              <td>{stat.successRate}%</td>
              <td>{stat.avgDuration}ms</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

<a name="edge-rendering"></a>
## 🔴 Edge Rendering

### **What is Edge Rendering?**

Edge rendering runs your server-side code **closer to the user** on edge nodes (CDN locations) instead of a centralized server.

```
Traditional SSR:
User (Tokyo) → Server (US) → Response
               └─ 200ms latency

Edge Rendering:
User (Tokyo) → Edge (Tokyo) → Response
               └─ 10ms latency
```

---

### **✅ Edge Rendering Platforms**

```typescript
// Platform comparison
const edgePlatforms = {
  vercel: {
    name: 'Vercel Edge Functions',
    runtime: 'V8 isolate',
    languages: ['TypeScript', 'JavaScript'],
    coldStart: '0ms',
    locations: '70+ regions',
    pricing: 'Usage-based',
  },
  
  cloudflare: {
    name: 'Cloudflare Workers',
    runtime: 'V8 isolate',
    languages: ['TypeScript', 'JavaScript', 'Rust', 'C++'],
    coldStart: '0ms',
    locations: '275+ cities',
    pricing: 'Free tier + usage',
  },
  
  netlify: {
    name: 'Netlify Edge Functions',
    runtime: 'Deno',
    languages: ['TypeScript', 'JavaScript'],
    coldStart: '<1ms',
    locations: 'Global CDN',
    pricing: 'Usage-based',
  },
  
  aws: {
    name: 'Lambda@Edge',
    runtime: 'Node.js / Python',
    languages: ['Node.js', 'Python'],
    coldStart: '50-100ms',
    locations: '400+ edge locations',
    pricing: 'Per request + duration',
  },
};
```

---

### **✅ Edge Functions Implementation**

#### **Vercel Edge Functions**

```typescript
// app/api/hello/route.ts (Next.js 13+ App Router)
import { NextRequest, NextResponse } from 'next/server';

// ✅ Edge Runtime
export const runtime = 'edge';

export async function GET(request: NextRequest) {
  // Access geolocation data
  const country = request.geo?.country || 'Unknown';
  const city = request.geo?.city || 'Unknown';
  
  // Access request headers
  const userAgent = request.headers.get('user-agent');
  
  return NextResponse.json({
    message: 'Hello from the edge!',
    location: { country, city },
    userAgent,
    timestamp: new Date().toISOString(),
  });
}

// Example: Personalized content based on location
export async function POST(request: NextRequest) {
  const { country } = request.geo || {};
  
  // Fetch region-specific content
  const content = await fetch(`https://api.example.com/content/${country}`);
  
  return NextResponse.json(await content.json());
}
```

#### **Cloudflare Workers**

```typescript
// worker.ts
export interface Env {
  KV_NAMESPACE: KVNamespace;
  DB: D1Database;
}

export default {
  async fetch(
    request: Request,
    env: Env,
    ctx: ExecutionContext
  ): Promise<Response> {
    const url = new URL(request.url);
    
    // Geolocation from Cloudflare
    const country = request.cf?.country;
    const city = request.cf?.city;
    
    // Example: A/B testing at the edge
    const variant = Math.random() < 0.5 ? 'A' : 'B';
    
    // Cache at edge
    const cacheKey = new Request(url.toString(), request);
    const cache = caches.default;
    
    let response = await cache.match(cacheKey);
    
    if (!response) {
      // Fetch from origin
      response = await fetch(request);
      
      // Clone and cache
      ctx.waitUntil(cache.put(cacheKey, response.clone()));
    }
    
    // Add custom headers
    const newResponse = new Response(response.body, response);
    newResponse.headers.set('X-Variant', variant);
    newResponse.headers.set('X-Country', country || 'unknown');
    
    return newResponse;
  },
};

// KV Storage at edge
export async function handleKV(request: Request, env: Env) {
  const url = new URL(request.url);
  const key = url.pathname.slice(1);
  
  if (request.method === 'GET') {
    const value = await env.KV_NAMESPACE.get(key);
    return new Response(value);
  }
  
  if (request.method === 'PUT') {
    const value = await request.text();
    await env.KV_NAMESPACE.put(key, value);
    return new Response('Stored', { status: 201 });
  }
}
```

#### **Netlify Edge Functions**

```typescript
// netlify/edge-functions/hello.ts
import type { Context } from "@netlify/edge-functions";

export default async (request: Request, context: Context) => {
  // Access geolocation
  const { geo } = context;
  
  // Cookies
  const sessionId = context.cookies.get('sessionId');
  
  // Rewrite/redirect at edge
  if (geo.country?.code === 'CN') {
    return context.rewrite('/cn');
  }
  
  // Transform response
  const response = await context.next();
  const html = await response.text();
  
  // Inject country-specific content
  const modified = html.replace(
    '</body>',
    `<script>window.COUNTRY = '${geo.country?.code}';</script></body>`
  );
  
  return new Response(modified, {
    headers: response.headers,
  });
};

export const config = {
  path: "/products/*",
};
```

---

### **✅ Edge Use Cases**

```typescript
// 1. GEOLOCATION-BASED ROUTING
// middleware.ts (Next.js)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const country = request.geo?.country;
  
  // Redirect to country-specific version
  if (country === 'DE' && !request.nextUrl.pathname.startsWith('/de')) {
    return NextResponse.redirect(new URL('/de', request.url));
  }
  
  if (country === 'FR' && !request.nextUrl.pathname.startsWith('/fr')) {
    return NextResponse.redirect(new URL('/fr', request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};

// 2. A/B TESTING AT EDGE
export function middleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // Get or assign variant
  let variant = request.cookies.get('ab-test-variant')?.value;
  
  if (!variant) {
    variant = Math.random() < 0.5 ? 'A' : 'B';
    response.cookies.set('ab-test-variant', variant, {
      maxAge: 60 * 60 * 24 * 30, // 30 days
    });
  }
  
  // Rewrite to variant page
  if (variant === 'B' && request.nextUrl.pathname === '/') {
    return NextResponse.rewrite(new URL('/variant-b', request.url));
  }
  
  return response;
}

// 3. AUTHENTICATION AT EDGE
export async function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token')?.value;
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  // Verify JWT at edge (fast!)
  try {
    await verifyJWT(token);
    return NextResponse.next();
  } catch (error) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
}

// 4. RATE LIMITING AT EDGE
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'),
});

export async function middleware(request: NextRequest) {
  const ip = request.ip ?? '127.0.0.1';
  const { success } = await ratelimit.limit(ip);
  
  if (!success) {
    return new NextResponse('Too Many Requests', { status: 429 });
  }
  
  return NextResponse.next();
}

// 5. IMAGE OPTIMIZATION AT EDGE
export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  
  if (pathname.startsWith('/images/')) {
    const url = new URL(request.url);
    
    // Add WebP conversion
    if (request.headers.get('accept')?.includes('image/webp')) {
      url.searchParams.set('format', 'webp');
    }
    
    // Add device-specific quality
    const viewport = request.headers.get('viewport-width');
    if (viewport && parseInt(viewport) < 768) {
      url.searchParams.set('quality', '60'); // Lower quality for mobile
    }
    
    return NextResponse.rewrite(url);
  }
  
  return NextResponse.next();
}

// 6. FEATURE FLAGS AT EDGE
const features = {
  newUI: ['US', 'CA'], // Enabled for US and Canada
  darkMode: ['*'], // Enabled globally
  betaFeature: [], // Disabled
};

export function middleware(request: NextRequest) {
  const country = request.geo?.country || 'US';
  const response = NextResponse.next();
  
  // Check feature flags
  const enabledFeatures = Object.entries(features)
    .filter(([_, countries]) => 
      countries.includes('*') || countries.includes(country)
    )
    .map(([feature]) => feature);
  
  // Inject feature flags
  response.headers.set('X-Features', JSON.stringify(enabledFeatures));
  
  return response;
}
```

---

### **✅ Edge Rendering Limitations**

```typescript
// ❌ NOT available in Edge Runtime
const limitations = {
  nodeAPIs: [
    'fs', // No file system access
    'child_process', // No spawning processes
    'net', // No raw TCP/UDP
  ],
  
  npm: [
    'Cannot use packages that depend on Node.js APIs',
    'Limited to packages that work in browser-like environment',
  ],
  
  compute: [
    'Max execution time: 30s (Vercel), 50ms (Cloudflare Workers)',
    'Memory limits: Lower than traditional servers',
    'CPU time limits',
  ],
  
  size: [
    'Bundle size limits (1MB compressed for Vercel)',
    'Cannot import large dependencies',
  ],
};

// ✅ Workarounds
// 1. Proxy to traditional server for heavy computation
export async function middleware(request: NextRequest) {
  if (requiresHeavyComputation(request)) {
    // Proxy to traditional server
    return fetch('https://api-server.example.com', {
      method: request.method,
      headers: request.headers,
      body: request.body,
    });
  }
  
  // Handle at edge
  return NextResponse.next();
}

// 2. Use edge-compatible alternatives
// Instead of: import bcrypt from 'bcrypt' (Node.js only)
// Use: import bcrypt from 'bcryptjs' (JavaScript implementation)

// 3. Conditional imports
const isEdge = process.env.NEXT_RUNTIME === 'edge';

const crypto = isEdge
  ? await import('crypto-js')
  : await import('crypto'); // Node.js crypto
```

---

<a name="blue-green"></a>
## 🔴 Blue-Green Deployments

### **What is Blue-Green Deployment?**

Blue-Green deployment maintains **two identical production environments** (Blue and Green). Traffic switches instantly between them.

```
Blue Environment (Current Production)
   ↓ Traffic (100%)
   
Deploy new version to Green Environment (Idle)
   ↓ Test Green
   ↓ Switch traffic
   
Green Environment (New Production)
   ↓ Traffic (100%)
   
Blue Environment (Idle, ready for rollback)
```

---

### **✅ Blue-Green Implementation**

#### **Infrastructure Setup**

```yaml
# docker-compose.yml - Blue/Green containers
version: '3.8'

services:
  # Blue environment
  app-blue:
    image: myapp:v1.0.0
    container_name: app-blue
    ports:
      - "3001:3000"
    environment:
      - NODE_ENV=production
      - VERSION=blue
    networks:
      - app-network

  # Green environment
  app-green:
    image: myapp:v1.1.0
    container_name: app-green
    ports:
      - "3002:3000"
    environment:
      - NODE_ENV=production
      - VERSION=green
    networks:
      - app-network

  # Load balancer
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - app-network
    depends_on:
      - app-blue
      - app-green

networks:
  app-network:
    driver: bridge
```

```nginx
# nginx.conf - Load balancer configuration
http {
    upstream app {
        # Initially route to blue
        server app-blue:3000 weight=100;
        server app-green:3000 weight=0;
    }
    
    server {
        listen 80;
        
        location / {
            proxy_pass http://app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "OK\n";
        }
    }
}
```

#### **Deployment Script**

```bash
#!/bin/bash
# deploy-blue-green.sh

CURRENT_ENV=$(docker ps --filter "name=app-" --filter "label=active=true" --format "{{.Names}}" | cut -d'-' -f2)

if [ "$CURRENT_ENV" = "blue" ]; then
    NEW_ENV="green"
    OLD_ENV="blue"
else
    NEW_ENV="blue"
    OLD_ENV="green"
fi

echo "Current environment: $OLD_ENV"
echo "Deploying to: $NEW_ENV"

# 1. Build new version
echo "Building new Docker image..."
docker build -t myapp:latest .
docker tag myapp:latest myapp:$NEW_ENV

# 2. Stop and update inactive environment
echo "Updating $NEW_ENV environment..."
docker-compose stop app-$NEW_ENV
docker-compose rm -f app-$NEW_ENV
docker-compose up -d app-$NEW_ENV

# 3. Wait for new environment to be ready
echo "Waiting for $NEW_ENV to be ready..."
for i in {1..30}; do
    if curl -f http://localhost:300${NEW_ENV#*e}/health > /dev/null 2>&1; then
        echo "$NEW_ENV is ready!"
        break
    fi
    echo "Attempt $i/30..."
    sleep 2
done

# 4. Run smoke tests
echo "Running smoke tests on $NEW_ENV..."
if ! ./run-smoke-tests.sh "http://localhost:300${NEW_ENV#*e}"; then
    echo "Smoke tests failed! Aborting deployment."
    exit 1
fi

# 5. Switch traffic
echo "Switching traffic to $NEW_ENV..."
cat > nginx.conf <<EOF
http {
    upstream app {
        server app-$NEW_ENV:3000 weight=100;
        server app-$OLD_ENV:3000 weight=0;
    }
}
EOF

docker-compose exec nginx nginx -s reload

echo "Traffic switched to $NEW_ENV"

# 6. Monitor for issues
echo "Monitoring for 5 minutes..."
sleep 300

# 7. Check error rates
ERROR_RATE=$(check_error_rate)
if [ "$ERROR_RATE" -gt 5 ]; then
    echo "Error rate too high! Rolling back..."
    ./rollback-blue-green.sh
    exit 1
fi

# 8. Keep old environment for quick rollback
echo "Deployment successful! $OLD_ENV kept for rollback if needed."
echo "To fully decommission $OLD_ENV, run: docker-compose stop app-$OLD_ENV"
```

#### **Rollback Script**

```bash
#!/bin/bash
# rollback-blue-green.sh

CURRENT_ENV=$(get_current_environment)

if [ "$CURRENT_ENV" = "blue" ]; then
    ROLLBACK_TO="green"
else
    ROLLBACK_TO="blue"
fi

echo "Rolling back to $ROLLBACK_TO..."

# Switch traffic back
cat > nginx.conf <<EOF
http {
    upstream app {
        server app-$ROLLBACK_TO:3000 weight=100;
        server app-$CURRENT_ENV:3000 weight=0;
    }
}
EOF

docker-compose exec nginx nginx -s reload

echo "Rollback complete! Traffic restored to $ROLLBACK_TO"
```

---

### **✅ Blue-Green with Kubernetes**

```yaml
# blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: VERSION
          value: "blue"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
# green-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v1.1.0
        ports:
        - containerPort: 3000
        env:
        - name: VERSION
          value: "green"

---
# service.yaml - Switch traffic by updating selector
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Change to 'green' to switch
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer
```

```bash
# Deployment script for Kubernetes
#!/bin/bash

# 1. Deploy green environment
kubectl apply -f green-deployment.yaml

# 2. Wait for green to be ready
kubectl wait --for=condition=available --timeout=300s deployment/myapp-green

# 3. Run tests against green
kubectl port-forward deployment/myapp-green 8080:3000 &
PF_PID=$!
sleep 5

if ! ./run-tests.sh http://localhost:8080; then
    echo "Tests failed! Keeping blue active."
    kill $PF_PID
    exit 1
fi
kill $PF_PID

# 4. Switch service to green
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'

echo "Traffic switched to green"

# 5. Monitor and optionally scale down blue
sleep 300  # Wait 5 minutes
kubectl scale deployment myapp-blue --replicas=1  # Keep 1 pod for quick rollback
```

---

### **✅ Blue-Green with Vercel/Netlify**

```javascript
// vercel.json - Preview deployments as "Green"
{
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/next"
    }
  ],
  "env": {
    "ENVIRONMENT": "production"
  },
  "build": {
    "env": {
      "NEXT_PUBLIC_ENV": "production"
    }
  }
}

// Deploy workflow
// 1. Push to branch → Creates preview deployment (Green)
// 2. Test preview URL
// 3. Merge to main → Promotes to production (switches Blue → Green)
// 4. Previous production becomes available for instant rollback
```

```bash
# GitHub Actions - Blue-Green with Vercel
name: Blue-Green Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # 1. Deploy to Vercel (Green)
      - name: Deploy to Vercel
        id: deploy
        run: |
          DEPLOYMENT_URL=$(vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }})
          echo "url=$DEPLOYMENT_URL" >> $GITHUB_OUTPUT
      
      # 2. Run smoke tests on new deployment
      - name: Smoke Tests
        run: |
          npm install
          npm run test:smoke -- ${{ steps.deploy.outputs.url }}
      
      # 3. If tests pass, traffic automatically switches
      # 4. Previous deployment remains for rollback
      
      - name: Notify Success
        if: success()
        run: |
          echo "Deployment successful: ${{ steps.deploy.outputs.url }}"
          
      - name: Rollback on Failure
        if: failure()
        run: |
          # Vercel keeps previous deployments
          # Can instantly rollback via dashboard or CLI
          echo "Deployment failed. Previous version still active."
```

---

<a name="canary-releases"></a>
## 🔴 Canary Releases

### **What is Canary Release?**

Canary release gradually rolls out changes to a **small subset of users** before full deployment, allowing you to detect issues early.

```
Traditional Deployment:
Old Version (100%) → New Version (100%)
└─ All users affected by bugs

Canary Release:
Old Version (100%)
  ↓
Old (95%) + Canary (5%)
  ↓ Monitor metrics
Old (80%) + Canary (20%)
  ↓ All good?
Old (50%) + Canary (50%)
  ↓ Continue...
New Version (100%)
```

---

### **✅ Canary Implementation**

#### **Load Balancer Canary (NGINX)**

```nginx
# nginx.conf - Weighted traffic splitting
http {
    upstream app-stable {
        server app-v1:3000;
    }
    
    upstream app-canary {
        server app-v2:3000;
    }
    
    # Split module for percentage-based routing
    split_clients "${remote_addr}${http_user_agent}" $variant {
        5% "canary";     # 5% of traffic to canary
        * "stable";      # 95% to stable
    }
    
    server {
        listen 80;
        
        location / {
            proxy_pass http://app-$variant;
            proxy_set_header X-Variant $variant;
            proxy_set_header Host $host;
        }
    }
}
```

#### **Canary with Feature Flags**

```typescript
// utils/feature-flags.ts
interface FeatureFlag {
  enabled: boolean;
  rollout: number; // Percentage 0-100
  userGroups?: string[]; // Specific user groups
}

class FeatureFlagService {
  private flags: Map<string, FeatureFlag> = new Map();
  
  constructor() {
    this.flags.set('new-checkout', {
      enabled: true,
      rollout: 10, // 10% rollout
      userGroups: ['beta-testers'],
    });
  }
  
  isEnabled(flagName: string, userId: string, userGroups: string[] = []): boolean {
    const flag = this.flags.get(flagName);
    
    if (!flag || !flag.enabled) {
      return false;
    }
    
    // Check if user is in specific group
    if (flag.userGroups && userGroups.some(g => flag.userGroups!.includes(g))) {
      return true;
    }
    
    // Consistent hashing for percentage rollout
    const hash = this.hashUserId(userId);
    const threshold = (flag.rollout / 100) * Number.MAX_SAFE_INTEGER;
    
    return hash < threshold;
  }
  
  private hashUserId(userId: string): number {
    let hash = 0;
    for (let i = 0; i < userId.length; i++) {
      hash = ((hash << 5) - hash) + userId.charCodeAt(i);
      hash = hash & hash; // Convert to 32-bit integer
    }
    return Math.abs(hash);
  }
  
  updateRollout(flagName: string, rollout: number) {
    const flag = this.flags.get(flagName);
    if (flag) {
      flag.rollout = rollout;
    }
  }
}

export const featureFlags = new FeatureFlagService();

// Usage in components
import { featureFlags } from '@/utils/feature-flags';

function CheckoutPage() {
  const { user } = useAuth();
  const useNewCheckout = featureFlags.isEnabled(
    'new-checkout',
    user.id,
    user.groups
  );
  
  if (useNewCheckout) {
    return <NewCheckout />;
  }
  
  return <OldCheckout />;
}
```

#### **Progressive Rollout Script**

```bash
#!/bin/bash
# canary-rollout.sh - Gradually increase canary traffic

CANARY_PERCENTAGES=(5 10 25 50 75 100)
MONITORING_DURATION=300  # 5 minutes between increases

for PERCENTAGE in "${CANARY_PERCENTAGES[@]}"; do
    echo "Increasing canary traffic to $PERCENTAGE%..."
    
    # Update load balancer configuration
    update_traffic_split $PERCENTAGE
    
    # Wait for metrics to stabilize
    echo "Monitoring for $MONITORING_DURATION seconds..."
    sleep $MONITORING_DURATION
    
    # Check metrics
    ERROR_RATE=$(get_error_rate "canary")
    LATENCY=$(get_avg_latency "canary")
    
    echo "Canary metrics - Error rate: $ERROR_RATE%, Latency: ${LATENCY}ms"
    
    # Compare with stable version
    STABLE_ERROR_RATE=$(get_error_rate "stable")
    STABLE_LATENCY=$(get_avg_latency "stable")
    
    # Abort if canary performs significantly worse
    if (( $(echo "$ERROR_RATE > $STABLE_ERROR_RATE * 1.5" | bc -l) )); then
        echo "⚠️  Error rate too high! Rolling back..."
        rollback_canary
        exit 1
    fi
    
    if (( $(echo "$LATENCY > $STABLE_LATENCY * 1.3" | bc -l) )); then
        echo "⚠️  Latency too high! Rolling back..."
        rollback_canary
        exit 1
    fi
    
    echo "✅ Metrics healthy at $PERCENTAGE%"
done

echo "🎉 Canary successfully deployed to 100%!"
```

---

### **✅ Canary with Kubernetes**

```yaml
# canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1  # Start with 1 pod (5% if stable has 19 pods)
  selector:
    matchLabels:
      app: myapp
      version: canary
  template:
    metadata:
      labels:
        app: myapp
        version: canary
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0.0
        ports:
        - containerPort: 3000

---
# stable-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 19  # 95% of traffic
  selector:
    matchLabels:
      app: myapp
      version: stable
  template:
    metadata:
      labels:
        app: myapp
        version: stable
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0.0
        ports:
        - containerPort: 3000

---
# service.yaml - Routes to both versions
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp  # Matches both stable and canary
  ports:
  - port: 80
    targetPort: 3000
```

```bash
# Progressive canary script for Kubernetes
#!/bin/bash

STABLE_REPLICAS=20
CANARY_PERCENTAGES=(5 10 25 50 100)

for PCT in "${CANARY_PERCENTAGES[@]}"; do
    CANARY_REPLICAS=$((STABLE_REPLICAS * PCT / 100))
    STABLE_COUNT=$((STABLE_REPLICAS - CANARY_REPLICAS))
    
    echo "Scaling to $PCT% canary ($CANARY_REPLICAS pods)"
    
    kubectl scale deployment myapp-canary --replicas=$CANARY_REPLICAS
    kubectl scale deployment myapp-stable --replicas=$STABLE_COUNT
    
    # Wait for scaling
    kubectl wait --for=condition=available deployment/myapp-canary --timeout=300s
    
    # Monitor
    sleep 300
    
    # Check metrics from Prometheus
    ERROR_RATE=$(get_prometheus_metric 'error_rate{version="canary"}')
    
    if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
        echo "Error rate too high! Rolling back..."
        kubectl scale deployment myapp-canary --replicas=0
        kubectl scale deployment myapp-stable --replicas=$STABLE_REPLICAS
        exit 1
    fi
    
    echo "✅ $PCT% rollout successful"
done

# Full rollout
kubectl scale deployment myapp-canary --replicas=$STABLE_REPLICAS
kubectl scale deployment myapp-stable --replicas=0

# After validation, delete old deployment
kubectl delete deployment myapp-stable
kubectl label deployment myapp-canary version=stable
```

---

### **✅ Canary with Monitoring**

```typescript
// monitoring/canary-metrics.ts
interface CanaryMetrics {
  version: 'stable' | 'canary';
  errorRate: number;
  latency: {
    p50: number;
    p95: number;
    p99: number;
  };
  requestCount: number;
  timestamp: number;
}

class CanaryMonitor {
  private metrics: Map<string, CanaryMetrics[]> = new Map();
  private alertThresholds = {
    errorRate: 0.05, // 5% error rate
    latencyP99: 1000, // 1000ms p99 latency
    relativeErrorIncrease: 1.5, // 50% increase vs stable
    relativeLatencyIncrease: 1.3, // 30% increase vs stable
  };
  
  async recordMetric(metric: CanaryMetrics) {
    const key = metric.version;
    const history = this.metrics.get(key) || [];
    history.push(metric);
    
    // Keep last hour of metrics
    const oneHourAgo = Date.now() - 3600000;
    this.metrics.set(
      key,
      history.filter(m => m.timestamp > oneHourAgo)
    );
    
    // Check for issues
    await this.checkHealth();
  }
  
  private async checkHealth() {
    const canaryMetrics = this.getRecentMetrics('canary');
    const stableMetrics = this.getRecentMetrics('stable');
    
    if (!canaryMetrics || !stableMetrics) return;
    
    const issues: string[] = [];
    
    // Check absolute thresholds
    if (canaryMetrics.errorRate > this.alertThresholds.errorRate) {
      issues.push(`High error rate: ${canaryMetrics.errorRate * 100}%`);
    }
    
    if (canaryMetrics.latency.p99 > this.alertThresholds.latencyP99) {
      issues.push(`High p99 latency: ${canaryMetrics.latency.p99}ms`);
    }
    
    // Check relative thresholds
    const errorRateIncrease = canaryMetrics.errorRate / stableMetrics.errorRate;
    if (errorRateIncrease > this.alertThresholds.relativeErrorIncrease) {
      issues.push(`Error rate ${errorRateIncrease.toFixed(2)}x higher than stable`);
    }
    
    const latencyIncrease = canaryMetrics.latency.p99 / stableMetrics.latency.p99;
    if (latencyIncrease > this.alertThresholds.relativeLatencyIncrease) {
      issues.push(`Latency ${latencyIncrease.toFixed(2)}x higher than stable`);
    }
    
    if (issues.length > 0) {
      await this.alert(issues);
    }
  }
  
  private getRecentMetrics(version: 'stable' | 'canary'): CanaryMetrics | null {
    const metrics = this.metrics.get(version) || [];
    if (metrics.length === 0) return null;
    
    // Average last 5 minutes
    const fiveMinutesAgo = Date.now() - 300000;
    const recent = metrics.filter(m => m.timestamp > fiveMinutesAgo);
    
    if (recent.length === 0) return null;
    
    return {
      version,
      errorRate: recent.reduce((sum, m) => sum + m.errorRate, 0) / recent.length,
      latency: {
        p50: recent.reduce((sum, m) => sum + m.latency.p50, 0) / recent.length,
        p95: recent.reduce((sum, m) => sum + m.latency.p95, 0) / recent.length,
        p99: recent.reduce((sum, m) => sum + m.latency.p99, 0) / recent.length,
      },
      requestCount: recent.reduce((sum, m) => sum + m.requestCount, 0),
      timestamp: Date.now(),
    };
  }
  
  private async alert(issues: string[]) {
    console.error('🚨 CANARY ALERT:', issues);
    
    // Send to monitoring service
    await fetch('/api/alerts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        severity: 'critical',
        service: 'canary-deployment',
        issues,
        timestamp: Date.now(),
      }),
    });
    
    // Optionally auto-rollback
    if (issues.some(i => i.includes('error rate'))) {
      await this.triggerRollback();
    }
  }
  
  private async triggerRollback() {
    console.log('🔄 Auto-rolling back canary deployment...');
    
    await fetch('/api/deployment/rollback', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ reason: 'automated-health-check-failure' }),
    });
  }
}

export const canaryMonitor = new CanaryMonitor();

// Middleware to track metrics
export function trackCanaryMetrics(req, res, next) {
  const start = Date.now();
  const version = req.headers['x-version'] || 'stable';
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    const isError = res.statusCode >= 400;
    
    canaryMonitor.recordMetric({
      version: version as 'stable' | 'canary',
      errorRate: isError ? 1 : 0,
      latency: { p50: duration, p95: duration, p99: duration },
      requestCount: 1,
      timestamp: Date.now(),
    });
  });
  
  next();
}
```

---

<a name="interview-questions"></a>
## 🎯 Common Interview Questions

### **Q1: When would you use SSR vs SSG vs CSR?**

**Answer:**

**Use SSR when:**
- SEO is critical AND content changes frequently
- Personalized content (user-specific data)
- Real-time data requirements
- Example: E-commerce product pages, news sites

**Use SSG when:**
- SEO is critical AND content rarely changes
- Content same for all users
- Can rebuild when content updates
- Example: Blogs, documentation, marketing pages

**Use CSR when:**
- SEO not important (behind auth)
- Highly interactive applications
- Content doesn't need to be indexed
- Example: Dashboards, admin panels, internal tools

**My recommendation for most projects:**
- Start with SSG + ISR (best performance + fresh content)
- Add SSR only where needed (user-specific pages)
- Use CSR for authenticated sections

**Real-world example:**
```
E-commerce site:
/ (Homepage) → SSG
/products → SSG + ISR (revalidate: 60)
/product/[id] → SSG + ISR
/cart → CSR
/checkout → SSR (personalized, secure)
/account → SSR
```

---

### **Q2: Explain ISR and its benefits**

**Answer:**

ISR (Incremental Static Regeneration) allows you to:
1. Generate static pages at build time
2. Regenerate them in the background after deployment
3. Serve stale content while regenerating

**How it works:**
```typescript
export const getStaticProps = async () => {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60, // Regenerate after 60 seconds
  };
};

// Timeline:
// t=0: Page generated at build
// t=30: User request → Serve static page (fast!)
// t=65: First request after 60s → Serve old page, trigger regeneration
// t=66: Regeneration complete
// t=70: Next user gets fresh content
```

**Benefits:**
- Static page performance
- Fresh content without rebuilds
- Handles high traffic (pages cached at CDN)
- Reduces build times (don't pre-render everything)

**Use cases:**
- Product catalogs (prices change)
- Blog posts (occasional updates)
- News articles (initially static, update for corrections)

**Limitation:** Content can be stale for `revalidate` seconds

---

### **Q3: What is edge rendering and when would you use it?**

**Answer:**

Edge rendering runs server-side code on CDN edge nodes **close to users** instead of a centralized server.

**Benefits:**
- Lower latency (10-50ms vs 200ms)
- Global distribution
- Zero cold starts (V8 isolates)
- Better performance worldwide

**When to use:**
1. **Geo-personalization**
```typescript
// Redirect based on location
if (request.geo.country === 'DE') {
  return redirect('/de');
}
```

2. **A/B testing**
```typescript
// Assign variant at edge (no flicker)
const variant = Math.random() < 0.5 ? 'A' : 'B';
return rewrite(`/variant-${variant}`);
```

3. **Authentication checks**
```typescript
// Verify JWT at edge (fast!)
if (!await verifyToken(request.cookies.token)) {
  return redirect('/login');
}
```

4. **Rate limiting**
```typescript
// Block at edge before hitting origin
if (await rateLimit.exceeded(ip)) {
  return new Response('Too many requests', { status: 429 });
}
```

**Limitations:**
- No Node.js APIs (fs, child_process)
- Limited execution time (30s-50ms depending on platform)
- Smaller bundle size limits

**My approach:** Use edge for routing/auth/personalization, proxy to traditional server for heavy computation.

---

### **Q4: Explain blue-green vs canary deployments**

**Answer:**

**Blue-Green:**
- Two identical environments
- Instant switch (100% → 0%, 0% → 100%)
- Quick rollback
- Higher cost (2x infrastructure)

```
Blue (100%) → Deploy to Green → Test Green → Switch 100% to Green
```

**Canary:**
- Gradual rollout
- Progressive traffic shift (5% → 10% → 25% → 50% → 100%)
- Early problem detection
- Same infrastructure

```
Old (100%) → Old (95%) + Canary (5%) → Monitor → Increase gradually
```

**When to use:**

**Blue-Green:**
- ✅ Major version changes
- ✅ Database migrations
- ✅ Need instant rollback
- ✅ Can afford 2x infrastructure
- ❌ Not for finding subtle bugs (100% switch)

**Canary:**
- ✅ Continuous deployments
- ✅ Testing with real users
- ✅ Minimize blast radius
- ✅ Cost-effective
- ❌ Slower rollout

**My preference:** Canary for most deployments (safer), blue-green for major releases.

---

### **Q5: How do you ensure zero-downtime deployments?**

**Answer:**

**My zero-downtime strategy:**

1. **Health Checks**
```typescript
app.get('/health', (req, res) => {
  const checks = {
    database: await db.ping(),
    cache: await redis.ping(),
    api: await externalAPI.ping(),
  };
  
  const healthy = Object.values(checks).every(Boolean);
  res.status(healthy ? 200 : 503).json(checks);
});
```

2. **Graceful Shutdown**
```typescript
let isShuttingDown = false;

process.on('SIGTERM', () => {
  isShuttingDown = true;
  
  // Stop accepting new requests
  server.close(() => {
    // Wait for existing requests to complete
    setTimeout(() => {
      process.exit(0);
    }, 30000); // 30s grace period
  });
});

app.use((req, res, next) => {
  if (isShuttingDown) {
    res.status(503).send('Server shutting down');
  } else {
    next();
  }
});
```

3. **Rolling Updates**
```yaml
# Kubernetes
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # Only 1 pod down at a time
      maxSurge: 1        # Only 1 extra pod during update
```

4. **Database Migrations**
```typescript
// Backward-compatible migrations
// Step 1: Add new column (old code still works)
ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT false;

// Step 2: Deploy new code that uses new column

// Step 3: Backfill data
UPDATE users SET email_verified = true WHERE email_confirmed_at IS NOT NULL;

// Step 4: Deploy code that requires new column

// Step 5: Remove old column (if needed)
ALTER TABLE users DROP COLUMN email_confirmed_at;
```

5. **Load Balancer Health Checks**
```
Load Balancer → Health check every 5s → Remove unhealthy instances
```

6. **Connection Draining**
```nginx
# NGINX
upstream app {
    server app1:3000;
    server app2:3000 down;  # Drain connections before removing
}
```

---

<a name="best-practices"></a>
## 📚 Daily Best Practices

### **Deployment Checklist**

```bash
# Pre-deployment checklist
[ ] Health checks implemented
[ ] Graceful shutdown configured
[ ] Database migrations backward-compatible
[ ] Rollback plan documented
[ ] Monitoring/alerts configured
[ ] Load testing completed
[ ] Security scan passed
[ ] Dependencies updated
[ ] Environment variables verified
[ ] Backup created

# Post-deployment monitoring (first 30 minutes)
[ ] Error rate < 1%
[ ] Latency p99 < 500ms
[ ] CPU usage < 70%
[ ] Memory usage < 80%
[ ] No increase in 5xx errors
[ ] Database connections stable
[ ] Cache hit rate normal
```

### **Progressive Learning Path**

**Week 1: Rendering Strategies**
- [ ] Build CSR app (Create React App)
- [ ] Convert to SSR (Next.js getServerSideProps)
- [ ] Convert to SSG (Next.js getStaticProps)
- [ ] Compare performance (Lighthouse)
- [ ] Implement ISR with on-demand revalidation

**Week 2: Edge & Advanced**
- [ ] Deploy to Vercel Edge Functions
- [ ] Implement geo-routing at edge
- [ ] Add A/B testing at edge
- [ ] Build authentication middleware at edge
- [ ] Compare edge vs server performance

**Week 3: Deployment Strategies**
- [ ] Set up blue-green with Docker
- [ ] Implement canary with feature flags
- [ ] Configure Kubernetes rolling updates
- [ ] Create automated rollback scripts
- [ ] Set up monitoring dashboards

**Week 4: Production Readiness**
- [ ] Implement health checks
- [ ] Add graceful shutdown
- [ ] Create deployment pipeline
- [ ] Set up alerts
- [ ] Document runbooks

---

## 🚀 Final Project

**Build a production-ready e-commerce site with:**

✅ Homepage - SSG (revalidate: 300)
✅ Product listings - SSG + ISR (revalidate: 60)
✅ Product pages - SSG + ISR with on-demand revalidation
✅ Cart - CSR with optimistic updates
✅ Checkout - SSR (secure, personalized)
✅ Account pages - SSR
✅ Edge middleware for geo-routing and auth
✅ Blue-green deployment setup
✅ Canary release pipeline with automated rollback
✅ Comprehensive monitoring

**Success Metrics:**
- Lighthouse score > 95
- First Contentful Paint < 1s
- Time to Interactive < 2s
- Zero-downtime deployments
- < 1% error rate during canary rollout

---

**✅ You are now a deployment expert!** 🚀