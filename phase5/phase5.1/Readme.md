<!-- 5.1 Build Systems
Webpack
Vite / esbuild
Tree shaking
Asset optimization
Env configs -->


<!--  -->



# 5.1 Build Systems - Complete Deep Dive

**Duration:** 1-1.5 weeks | **Interview Weightage:** Critical for Senior roles

---

## Table of Contents
1. [Build Systems Fundamentals](#fundamentals)
2. [Webpack](#webpack)
3. [Vite & esbuild](#vite-esbuild)
4. [Tree Shaking](#tree-shaking)
5. [Asset Optimization](#asset-optimization)
6. [Environment Configuration](#env-config)
7. [Interview Questions](#interview-questions)
8. [Daily Best Practices](#best-practices)

---

<a name="fundamentals"></a>
## 🔴 Build Systems Fundamentals

### **What is a Build System?**

A build system transforms your source code into optimized production-ready files that browsers can execute efficiently.

```
Source Code          Build Process           Production Bundle
────────────         ──────────────         ─────────────────
TypeScript    ──┐
JSX/TSX       ──┤
SCSS/CSS      ──┤──► Build Tool ──► Minified, optimized
Images        ──┤    (Webpack/Vite)    JavaScript, CSS,
SVG           ──┤                       and assets
Fonts         ──┘
```

### **Why Build Systems?**

```javascript
// ❌ Without build system - Problems:
// 1. Browser doesn't understand TypeScript
import { calculateTotal } from './utils.ts'; // ❌ Browser error

// 2. Inefficient loading - 100s of small files
<script src="file1.js"></script>
<script src="file2.js"></script>
// ... 100 more scripts

// 3. No optimization - Large file sizes
// Unminified, includes unused code, no compression

// 4. No module system
var MyApp = (function() {
  // Manual module pattern
})();

// ✅ With build system - Solutions:
// 1. TypeScript compiled to JavaScript
// 2. All files bundled into a few optimized chunks
// 3. Minified, tree-shaken, compressed
// 4. Modern ES modules with code splitting
```

### **Build System Evolution**

```
2010-2015: Task Runners
├── Grunt (2012)
├── Gulp (2013)
└── Problems: Configuration complexity

2015-2020: Bundlers
├── Webpack (2014) - Dominated the space
├── Rollup (2015) - Better for libraries
├── Parcel (2017) - Zero config
└── Problems: Slow dev servers, complex config

2020-Present: Modern Bundlers
├── Vite (2020) - Lightning fast HMR
├── esbuild (2020) - Written in Go, extremely fast
├── Turbopack (2022) - Rust-based, by Vercel
└── Solutions: Native ESM, fast rebuilds
```

---

<a name="webpack"></a>
## 🔴 Webpack

### **What is Webpack?**

Webpack is a **static module bundler** that creates a dependency graph of your application and bundles modules into optimized files.

### **Core Concepts**

```javascript
// webpack.config.js - Core Concepts
module.exports = {
  // 1. ENTRY - Where to start building the dependency graph
  entry: './src/index.js',
  
  // 2. OUTPUT - Where to emit the bundles
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  
  // 3. LOADERS - Transform files (e.g., TypeScript → JavaScript)
  module: {
    rules: [
      { test: /\.tsx?$/, use: 'ts-loader' },
    ],
  },
  
  // 4. PLUGINS - Optimize, inject HTML, extract CSS, etc.
  plugins: [
    new HtmlWebpackPlugin(),
  ],
  
  // 5. MODE - Development or production
  mode: 'production',
};
```

---

### **✅ Webpack Configuration Levels**

#### **Level 1: Basic Configuration**

```javascript
// webpack.config.js (Basic)
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    clean: true, // Clean dist folder before build
  },
  
  module: {
    rules: [
      // JavaScript/TypeScript
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-env',
              '@babel/preset-react',
              '@babel/preset-typescript',
            ],
          },
        },
      },
      
      // CSS
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      
      // Images
      {
        test: /\.(png|jpg|gif|svg)$/,
        type: 'asset/resource',
      },
    ],
  },
  
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
  ],
  
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
  },
  
  mode: 'development',
  
  devServer: {
    static: './dist',
    port: 3000,
    hot: true,
  },
};
```

#### **Level 2: Production-Optimized Configuration**

```javascript
// webpack.config.js (Production)
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const CompressionPlugin = require('compression-webpack-plugin');

const isDevelopment = process.env.NODE_ENV !== 'production';

module.exports = {
  mode: isDevelopment ? 'development' : 'production',
  
  entry: {
    main: './src/index.tsx',
  },
  
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: isDevelopment 
      ? '[name].js' 
      : '[name].[contenthash:8].js',
    chunkFilename: isDevelopment
      ? '[name].chunk.js'
      : '[name].[contenthash:8].chunk.js',
    assetModuleFilename: 'assets/[name].[hash][ext]',
    publicPath: '/',
    clean: true,
  },
  
  module: {
    rules: [
      // TypeScript/React
      {
        test: /\.(ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', {
                targets: { browsers: 'last 2 versions' },
                modules: false, // Keep ES modules for tree shaking
              }],
              '@babel/preset-react',
              '@babel/preset-typescript',
            ],
            plugins: [
              '@babel/plugin-transform-runtime',
              isDevelopment && 'react-refresh/babel',
            ].filter(Boolean),
          },
        },
      },
      
      // CSS/SCSS
      {
        test: /\.(css|scss)$/,
        use: [
          isDevelopment ? 'style-loader' : MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: {
                auto: true, // Enable CSS modules for .module.css files
                localIdentName: isDevelopment
                  ? '[path][name]__[local]'
                  : '[hash:base64:8]',
              },
              importLoaders: 2,
              sourceMap: isDevelopment,
            },
          },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  'autoprefixer',
                  !isDevelopment && 'cssnano',
                ].filter(Boolean),
              },
            },
          },
          'sass-loader',
        ],
      },
      
      // Images
      {
        test: /\.(png|jpg|jpeg|gif|webp)$/,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024, // 8kb - inline smaller images
          },
        },
      },
      
      // SVG
      {
        test: /\.svg$/,
        use: ['@svgr/webpack', 'url-loader'],
      },
      
      // Fonts
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        type: 'asset/resource',
      },
    ],
  },
  
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      favicon: './public/favicon.ico',
      minify: !isDevelopment && {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
        removeEmptyAttributes: true,
        removeStyleLinkTypeAttributes: true,
        keepClosingSlash: true,
        minifyJS: true,
        minifyCSS: true,
        minifyURLs: true,
      },
    }),
    
    !isDevelopment && new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
      chunkFilename: 'css/[name].[contenthash:8].chunk.css',
    }),
    
    !isDevelopment && new CompressionPlugin({
      filename: '[path][base].gz',
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 10240, // Only compress files > 10KB
      minRatio: 0.8,
    }),
    
    process.env.ANALYZE && new BundleAnalyzerPlugin(),
    
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(
        isDevelopment ? 'development' : 'production'
      ),
    }),
  ].filter(Boolean),
  
  optimization: {
    minimize: !isDevelopment,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // Remove console.log in production
            drop_debugger: true,
          },
          format: {
            comments: false, // Remove comments
          },
        },
        extractComments: false,
      }),
      new CssMinimizerPlugin(),
    ],
    
    // Code splitting
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Vendor bundle - third-party libraries
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
          reuseExistingChunk: true,
        },
        // Common code shared between chunks
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
          enforce: true,
        },
        // React and related libraries
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
          name: 'react',
          priority: 20,
        },
      },
    },
    
    // Runtime chunk for better caching
    runtimeChunk: {
      name: 'runtime',
    },
  },
  
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@hooks': path.resolve(__dirname, 'src/hooks'),
      '@assets': path.resolve(__dirname, 'src/assets'),
    },
  },
  
  devServer: {
    static: {
      directory: path.join(__dirname, 'public'),
    },
    port: 3000,
    hot: true,
    open: true,
    historyApiFallback: true, // SPA routing support
    compress: true,
    client: {
      overlay: {
        errors: true,
        warnings: false,
      },
    },
  },
  
  devtool: isDevelopment ? 'eval-source-map' : 'source-map',
  
  performance: {
    hints: !isDevelopment ? 'warning' : false,
    maxEntrypointSize: 512000, // 500KB
    maxAssetSize: 512000,
  },
  
  stats: {
    colors: true,
    modules: false,
    children: false,
    chunks: false,
    chunkModules: false,
  },
};
```

#### **Level 3: Advanced Multi-Entry Configuration**

```javascript
// webpack.config.js (Advanced - Micro-frontends)
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  mode: 'production',
  
  entry: {
    main: './src/index.tsx',
    admin: './src/admin/index.tsx',
  },
  
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
    publicPath: 'auto',
    clean: true,
  },
  
  module: {
    rules: [
      {
        test: /\.(ts|tsx)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true, // Cache babel transformations
            },
          },
          {
            loader: 'ts-loader',
            options: {
              transpileOnly: true, // Faster builds
              experimentalWatchApi: true,
            },
          },
        ],
      },
    ],
  },
  
  plugins: [
    // Multiple HTML entry points
    new HtmlWebpackPlugin({
      template: './public/index.html',
      filename: 'index.html',
      chunks: ['main', 'runtime', 'vendors'],
    }),
    
    new HtmlWebpackPlugin({
      template: './public/admin.html',
      filename: 'admin.html',
      chunks: ['admin', 'runtime', 'vendors'],
    }),
    
    // Module Federation for micro-frontends
    new ModuleFederationPlugin({
      name: 'host',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/components/Button',
        './Header': './src/components/Header',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
  
  optimization: {
    splitChunks: {
      chunks: 'all',
      maxInitialRequests: 25,
      maxAsyncRequests: 25,
      cacheGroups: {
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true,
          name(module) {
            // Get package name
            const packageName = module.context.match(
              /[\\/]node_modules[\\/](.*?)([\\/]|$)/
            )[1];
            return `npm.${packageName.replace('@', '')}`;
          },
        },
      },
    },
  },
};
```

---

### **✅ Webpack Performance Optimization**

#### **1. Build Performance**

```javascript
// webpack.config.js - Speed up builds
module.exports = {
  // Use cache
  cache: {
    type: 'filesystem',
    cacheDirectory: path.resolve(__dirname, '.webpack_cache'),
    buildDependencies: {
      config: [__filename],
    },
  },
  
  module: {
    rules: [
      {
        test: /\.(ts|tsx)$/,
        exclude: /node_modules/,
        use: [
          // Thread loader for parallel processing
          {
            loader: 'thread-loader',
            options: {
              workers: require('os').cpus().length - 1,
            },
          },
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true,
              cacheCompression: false,
            },
          },
        ],
      },
    ],
  },
  
  // Ignore parsing of large libraries
  module: {
    noParse: /lodash|moment/,
  },
  
  // Reduce resolve complexity
  resolve: {
    modules: [path.resolve(__dirname, 'src'), 'node_modules'],
    extensions: ['.ts', '.tsx', '.js'], // Only what you need
  },
};
```

#### **2. Bundle Size Optimization**

```javascript
// webpack.config.js - Reduce bundle size
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  optimization: {
    usedExports: true, // Tree shaking
    sideEffects: false, // Enable tree shaking
    
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,
            drop_debugger: true,
            pure_funcs: ['console.log'], // Remove specific functions
          },
        },
      }),
    ],
    
    splitChunks: {
      chunks: 'all',
      minSize: 20000,
      maxSize: 244000, // Split large chunks
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name(module) {
            const packageName = module.context.match(
              /[\\/]node_modules[\\/](.*?)([\\/]|$)/
            )[1];
            return `vendor.${packageName.replace('@', '')}`;
          },
        },
      },
    },
  },
  
  plugins: [
    // Analyze bundle size
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html',
    }),
  ],
};
```

#### **3. Lazy Loading & Code Splitting**

```typescript
// App.tsx - Dynamic imports for code splitting
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

// Webpack creates separate chunks for each lazy import:
// main.js, Home.chunk.js, Dashboard.chunk.js, Profile.chunk.js

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

// Dynamic import with webpack magic comments
const AdminPanel = lazy(() => 
  import(
    /* webpackChunkName: "admin" */
    /* webpackPrefetch: true */
    './pages/AdminPanel'
  )
);
```

---

### **✅ Webpack Loaders Deep Dive**

```javascript
// Common Loaders Explained

module.exports = {
  module: {
    rules: [
      // 1. BABEL LOADER - Transpile modern JS to compatible JS
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', {
                targets: '> 0.25%, not dead', // Browser support
                useBuiltIns: 'usage', // Polyfill only what's needed
                corejs: 3,
              }],
            ],
          },
        },
      },
      
      // 2. TS LOADER - TypeScript to JavaScript
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
      
      // 3. CSS LOADER - Import CSS as modules
      {
        test: /\.css$/,
        use: [
          'style-loader', // Injects CSS into DOM
          'css-loader',   // Resolves @import and url()
        ],
      },
      
      // 4. SASS LOADER - SCSS to CSS
      {
        test: /\.scss$/,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader', // Compile SCSS to CSS
        ],
      },
      
      // 5. POSTCSS LOADER - Transform CSS with plugins
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  'autoprefixer', // Add vendor prefixes
                  'tailwindcss',  // Process Tailwind
                ],
              },
            },
          },
        ],
      },
      
      // 6. FILE LOADER / ASSET MODULES - Handle files
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource', // Emit file and export URL
      },
      
      // 7. URL LOADER - Inline small files as data URLs
      {
        test: /\.(png|jpg|gif)$/,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024, // Inline files < 8KB
          },
        },
      },
      
      // 8. SVGR LOADER - Import SVG as React components
      {
        test: /\.svg$/,
        use: ['@svgr/webpack'],
      },
    ],
  },
};

// Usage examples:
// import Logo from './logo.svg'; // React component
// import ImageURL from './image.png'; // URL string
// import styles from './styles.module.css'; // CSS modules object
```

---

### **✅ Webpack Plugins Deep Dive**

```javascript
// Essential Webpack Plugins

const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const webpack = require('webpack');
const WorkboxPlugin = require('workbox-webpack-plugin');

module.exports = {
  plugins: [
    // 1. HTML WEBPACK PLUGIN - Generate HTML files
    new HtmlWebpackPlugin({
      template: './src/index.html',
      filename: 'index.html',
      inject: 'body',
      scriptLoading: 'defer',
      minify: {
        removeComments: true,
        collapseWhitespace: true,
      },
    }),
    
    // 2. MINI CSS EXTRACT PLUGIN - Extract CSS to separate files
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
      chunkFilename: 'css/[id].[contenthash:8].css',
    }),
    
    // 3. CLEAN WEBPACK PLUGIN - Clean output directory
    new CleanWebpackPlugin({
      cleanOnceBeforeBuildPatterns: ['**/*', '!stats.json'],
    }),
    
    // 4. COPY WEBPACK PLUGIN - Copy static files
    new CopyWebpackPlugin({
      patterns: [
        {
          from: 'public',
          to: 'public',
          globOptions: {
            ignore: ['**/index.html'],
          },
        },
      ],
    }),
    
    // 5. DEFINE PLUGIN - Define global constants
    new webpack.DefinePlugin({
      'process.env.API_URL': JSON.stringify(process.env.API_URL),
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
      __DEV__: process.env.NODE_ENV !== 'production',
    }),
    
    // 6. PROVIDE PLUGIN - Auto-import modules
    new webpack.ProvidePlugin({
      React: 'react', // No need to import React in every file
      $: 'jquery',
    }),
    
    // 7. BUNDLE ANALYZER PLUGIN - Visualize bundle size
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html',
    }),
    
    // 8. COMPRESSION PLUGIN - Generate gzipped files
    new CompressionPlugin({
      filename: '[path][base].gz',
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 10240,
      minRatio: 0.8,
    }),
    
    // 9. WORKBOX PLUGIN - Service worker for PWA
    new WorkboxPlugin.GenerateSW({
      clientsClaim: true,
      skipWaiting: true,
      runtimeCaching: [
        {
          urlPattern: /^https:\/\/api\.example\.com/,
          handler: 'NetworkFirst',
          options: {
            cacheName: 'api-cache',
            expiration: {
              maxEntries: 50,
              maxAgeSeconds: 300, // 5 minutes
            },
          },
        },
      ],
    }),
    
    // 10. PROGRESS PLUGIN - Show build progress
    new webpack.ProgressPlugin({
      activeModules: false,
      entries: true,
      modules: true,
      modulesCount: 5000,
      profile: false,
      dependencies: true,
      dependenciesCount: 10000,
      percentBy: null,
    }),
  ],
};
```

---

### **✅ Webpack Dev Server Configuration**

```javascript
// webpack.config.js - Advanced DevServer
module.exports = {
  devServer: {
    static: {
      directory: path.join(__dirname, 'public'),
      publicPath: '/',
      watch: true,
    },
    
    // Server configuration
    host: '0.0.0.0', // Allow external access
    port: 3000,
    open: true,
    hot: true, // Hot Module Replacement
    liveReload: true,
    
    // Compression
    compress: true,
    
    // SPA routing
    historyApiFallback: {
      rewrites: [
        { from: /^\/admin/, to: '/admin.html' },
        { from: /./, to: '/index.html' },
      ],
    },
    
    // Proxy API requests
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        pathRewrite: { '^/api': '' },
        changeOrigin: true,
        secure: false,
      },
      '/auth': {
        target: 'http://localhost:8080',
        changeOrigin: true,
      },
    },
    
    // Headers
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, PATCH, OPTIONS',
      'Access-Control-Allow-Headers': 'X-Requested-With, content-type, Authorization',
    },
    
    // Client overlay
    client: {
      logging: 'info',
      overlay: {
        errors: true,
        warnings: false,
      },
      progress: true,
    },
    
    // HTTPS (for development)
    // https: {
    //   key: fs.readFileSync('path/to/server.key'),
    //   cert: fs.readFileSync('path/to/server.crt'),
    // },
  },
};
```

---

<a name="vite-esbuild"></a>
## 🔴 Vite & esbuild

### **What is Vite?**

Vite is a **next-generation frontend build tool** that leverages native ES modules for lightning-fast development and uses Rollup for optimized production builds.

### **Vite vs Webpack**

```
Development Server Start Time:
Webpack: 15-30 seconds (large project)
Vite: 1-2 seconds

Hot Module Replacement (HMR):
Webpack: 2-5 seconds
Vite: <100ms (instant)

Why so fast?
├── Vite uses native ESM (no bundling in dev)
├── esbuild pre-bundles dependencies (written in Go)
└── Only transforms files on-demand
```

---

### **✅ Vite Configuration**

#### **Basic Vite Config**

```javascript
// vite.config.js (Basic)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@utils': path.resolve(__dirname, './src/utils'),
    },
  },
  
  server: {
    port: 3000,
    open: true,
  },
  
  build: {
    outDir: 'dist',
  },
});
```

#### **Production-Optimized Vite Config**

```javascript
// vite.config.ts (Production)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';
import viteCompression from 'vite-plugin-compression';
import path from 'path';

export default defineConfig(({ mode }) => {
  const isDevelopment = mode === 'development';
  
  return {
    plugins: [
      react({
        // Fast Refresh options
        fastRefresh: true,
        
        // Babel options
        babel: {
          plugins: [
            // Production optimizations
            !isDevelopment && ['transform-remove-console'],
          ].filter(Boolean),
        },
      }),
      
      // Gzip compression
      viteCompression({
        algorithm: 'gzip',
        ext: '.gz',
        threshold: 10240, // Only compress files > 10KB
      }),
      
      // Brotli compression
      viteCompression({
        algorithm: 'brotliCompress',
        ext: '.br',
        threshold: 10240,
      }),
      
      // Bundle analyzer
      visualizer({
        open: false,
        filename: 'stats.html',
        gzipSize: true,
        brotliSize: true,
      }),
    ],
    
    resolve: {
      alias: {
        '@': path.resolve(__dirname, './src'),
        '@components': path.resolve(__dirname, './src/components'),
        '@utils': path.resolve(__dirname, './src/utils'),
        '@hooks': path.resolve(__dirname, './src/hooks'),
        '@assets': path.resolve(__dirname, './src/assets'),
        '@types': path.resolve(__dirname, './src/types'),
      },
    },
    
    css: {
      modules: {
        localsConvention: 'camelCase',
        generateScopedName: isDevelopment
          ? '[name]__[local]__[hash:base64:5]'
          : '[hash:base64:8]',
      },
      preprocessorOptions: {
        scss: {
          additionalData: `@import "@/styles/variables.scss";`,
        },
      },
    },
    
    build: {
      outDir: 'dist',
      assetsDir: 'assets',
      
      // Sourcemaps
      sourcemap: !isDevelopment,
      
      // Minification
      minify: 'esbuild',
      
      // Target
      target: 'es2015',
      
      // Chunk size warnings
      chunkSizeWarningLimit: 1000,
      
      // Rollup options
      rollupOptions: {
        output: {
          // Manual chunking strategy
          manualChunks: {
            // React core
            react: ['react', 'react-dom', 'react-router-dom'],
            
            // UI libraries
            ui: ['@mui/material', '@emotion/react', '@emotion/styled'],
            
            // Utilities
            utils: ['lodash-es', 'date-fns', 'axios'],
          },
          
          // Asset file names
          assetFileNames: (assetInfo) => {
            const info = assetInfo.name.split('.');
            let extType = info[info.length - 1];
            
            if (/png|jpe?g|svg|gif|tiff|bmp|ico/i.test(extType)) {
              extType = 'images';
            } else if (/woff|woff2|eot|ttf|otf/i.test(extType)) {
              extType = 'fonts';
            }
            
            return `assets/${extType}/[name].[hash][extname]`;
          },
          
          // Chunk file names
          chunkFileNames: 'js/[name].[hash].js',
          entryFileNames: 'js/[name].[hash].js',
        },
      },
      
      // Terser options (if using terser instead of esbuild)
      // minify: 'terser',
      // terserOptions: {
      //   compress: {
      //     drop_console: true,
      //     drop_debugger: true,
      //   },
      // },
    },
    
    server: {
      port: 3000,
      host: true,
      open: true,
      
      // HMR options
      hmr: {
        overlay: true,
      },
      
      // Proxy
      proxy: {
        '/api': {
          target: 'http://localhost:8080',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, ''),
        },
      },
    },
    
    preview: {
      port: 4173,
      host: true,
    },
    
    // Dependency optimization
    optimizeDeps: {
      include: ['react', 'react-dom', 'react-router-dom'],
      exclude: ['@vite/client', '@vite/env'],
    },
    
    // Environment variables prefix
    envPrefix: 'VITE_',
  };
});
```

---

### **✅ Vite Plugins Ecosystem**

```javascript
// vite.config.ts - Essential Plugins
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';
import { imagetools } from 'vite-imagetools';
import svgr from 'vite-plugin-svgr';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [
    // 1. React plugin with Fast Refresh
    react(),
    
    // 2. TypeScript paths resolution
    tsconfigPaths(),
    
    // 3. SVG as React components
    svgr({
      svgrOptions: {
        icon: true,
      },
    }),
    
    // 4. Image optimization
    imagetools({
      defaultDirectives: (url) => {
        if (url.searchParams.has('thumbnail')) {
          return new URLSearchParams({
            w: '300',
            h: '300',
            fit: 'cover',
            format: 'webp',
          });
        }
        return new URLSearchParams();
      },
    }),
    
    // 5. PWA support
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'robots.txt', 'apple-touch-icon.png'],
      manifest: {
        name: 'My App',
        short_name: 'App',
        description: 'My awesome app',
        theme_color: '#ffffff',
        icons: [
          {
            src: 'pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'pwa-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
        ],
      },
      workbox: {
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\.example\.com\/.*/i,
            handler: 'NetworkFirst',
            options: {
              cacheName: 'api-cache',
              expiration: {
                maxEntries: 10,
                maxAgeSeconds: 60 * 60 * 24, // 24 hours
              },
            },
          },
        ],
      },
    }),
  ],
});

// Usage examples:
// import Logo from './logo.svg?react'; // React component
// import thumbnailImage from './image.jpg?thumbnail'; // Optimized thumbnail
```

---

### **✅ esbuild Deep Dive**

```javascript
// esbuild.config.js - Standalone esbuild (super fast!)
const esbuild = require('esbuild');
const { sassPlugin } = require('esbuild-sass-plugin');

esbuild.build({
  entryPoints: ['src/index.tsx'],
  bundle: true,
  outfile: 'dist/bundle.js',
  
  // Minification
  minify: true,
  sourcemap: true,
  
  // Target environment
  target: 'es2020',
  
  // Format
  format: 'esm',
  
  // Splitting (code splitting)
  splitting: true,
  outdir: 'dist',
  
  // Plugins
  plugins: [sassPlugin()],
  
  // Tree shaking
  treeShaking: true,
  
  // External dependencies (not bundled)
  external: ['react', 'react-dom'],
  
  // Loader for different file types
  loader: {
    '.png': 'file',
    '.svg': 'dataurl',
    '.woff': 'file',
    '.woff2': 'file',
  },
  
  // Define globals
  define: {
    'process.env.NODE_ENV': '"production"',
    global: 'window',
  },
  
  // JSX configuration
  jsxFactory: 'React.createElement',
  jsxFragment: 'React.Fragment',
  
  // Watch mode
  watch: process.env.NODE_ENV === 'development' && {
    onRebuild(error, result) {
      if (error) console.error('watch build failed:', error);
      else console.log('watch build succeeded:', result);
    },
  },
  
}).catch(() => process.exit(1));

// Speed comparison:
// esbuild: 0.5s (50-100x faster than webpack)
// webpack: 25-50s
```

---

### **Webpack vs Vite vs esbuild - When to Use**

```typescript
// Decision Matrix

interface BuildToolComparison {
  useWebpack: string[];
  useVite: string[];
  useEsbuild: string[];
}

const when: BuildToolComparison = {
  useWebpack: [
    'Legacy projects with existing webpack config',
    'Need very specific loaders/plugins not available in Vite',
    'Micro-frontend architecture (Module Federation)',
    'Complex build requirements with custom loaders',
    'Team already familiar with webpack',
  ],
  
  useVite: [
    '✅ New projects (RECOMMENDED)',
    '✅ Fast development experience needed',
    'React/Vue/Svelte applications',
    'Want instant HMR',
    'Modern browser targets',
    'Simple to medium complexity builds',
  ],
  
  useEsbuild: [
    'Library development (ultra-fast builds)',
    'Simple bundling needs',
    'Maximum build speed critical',
    'Standalone CLI tools',
    'Node.js applications',
  ],
};

// Performance Comparison (large app)
const buildTimes = {
  webpack: {
    dev: '30s initial, 5s rebuild',
    prod: '2-5 minutes',
  },
  vite: {
    dev: '2s initial, <100ms rebuild',
    prod: '30s-1min',
  },
  esbuild: {
    dev: 'Instant',
    prod: '5-15s',
  },
};
```

---

<a name="tree-shaking"></a>
## 🔴 Tree Shaking

### **What is Tree Shaking?**

Tree shaking is the process of **removing dead code** (unused exports) from your final bundle.

```javascript
// utils.js - Library with many exports
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }
export function divide(a, b) { return a / b; }
export function power(a, b) { return Math.pow(a, b); }
export function sqrt(a) { return Math.sqrt(a); }

// app.js - Only use one function
import { add } from './utils.js';

console.log(add(2, 3));

// ❌ Without tree shaking:
// Final bundle includes ALL functions (add, subtract, multiply, divide, power, sqrt)

// ✅ With tree shaking:
// Final bundle includes ONLY add function
```

---

### **✅ How Tree Shaking Works**

Tree shaking relies on **ES6 module** static structure:

```javascript
// ✅ GOOD - ES6 modules (tree-shakeable)
export function usedFunction() { /* ... */ }
export function unusedFunction() { /* ... */ }

import { usedFunction } from './module';
// Only usedFunction is included in bundle

// ❌ BAD - CommonJS (NOT tree-shakeable)
module.exports = {
  usedFunction: function() { /* ... */ },
  unusedFunction: function() { /* ... */ },
};

const { usedFunction } = require('./module');
// BOTH functions included in bundle
```

---

### **✅ Enabling Tree Shaking**

#### **1. package.json Configuration**

```json
{
  "name": "my-library",
  "version": "1.0.0",
  
  // Mark package as side-effect free
  "sideEffects": false,
  
  // Or specify files with side effects
  "sideEffects": [
    "*.css",
    "*.scss",
    "./src/polyfills.js"
  ],
  
  // Provide ES module entry point
  "module": "dist/index.esm.js",
  "main": "dist/index.cjs.js",
  
  "exports": {
    ".": {
      "import": "./dist/index.esm.js",
      "require": "./dist/index.cjs.js"
    },
    "./utils": {
      "import": "./dist/utils.esm.js",
      "require": "./dist/utils.cjs.js"
    }
  }
}
```

#### **2. Webpack Configuration**

```javascript
// webpack.config.js
module.exports = {
  mode: 'production', // Enables tree shaking
  
  optimization: {
    usedExports: true, // Mark unused exports
    sideEffects: true, // Respect package.json sideEffects
    
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            unused: true, // Remove unused code
            dead_code: true, // Remove unreachable code
          },
        },
      }),
    ],
  },
  
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', {
                modules: false, // CRITICAL: Preserve ES modules
              }],
            ],
          },
        },
      },
    ],
  },
};
```

#### **3. Vite Configuration**

```javascript
// vite.config.js
export default {
  build: {
    minify: 'esbuild', // or 'terser'
    
    rollupOptions: {
      treeshake: {
        moduleSideEffects: false, // Aggressive tree shaking
        propertyReadSideEffects: false,
        unknownGlobalSideEffects: false,
      },
    },
  },
};
```

---

### **✅ Writing Tree-Shakeable Code**

#### **Pattern 1: Named Exports (Not Default)**

```javascript
// ❌ BAD - Default export of object (not tree-shakeable)
export default {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
};

// Usage
import utils from './utils';
utils.add(2, 3); // ALL functions bundled

// ✅ GOOD - Named exports (tree-shakeable)
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export const multiply = (a, b) => a * b;

// Usage
import { add } from './utils';
add(2, 3); // ONLY add function bundled
```

#### **Pattern 2: Avoid Side Effects**

```javascript
// ❌ BAD - Side effect in module
// utils.js
export function add(a, b) { return a + b; }

// Side effect - always executed
console.log('Utils loaded'); // Prevents tree shaking

// ✅ GOOD - Pure exports only
// utils.js
export function add(a, b) { return a + b; }

// Move side effects to explicit initialization
export function initUtils() {
  console.log('Utils initialized');
}
```

#### **Pattern 3: Conditional Imports**

```javascript
// ❌ BAD - Import everything
import * as utils from './utils';

if (needMath) {
  utils.add(2, 3);
}

// ✅ GOOD - Import only what's needed
if (needMath) {
  const { add } = await import('./utils');
  add(2, 3);
}
```

#### **Pattern 4: Library Design for Tree Shaking**

```javascript
// library/index.js - Main export file
// ❌ BAD - Export everything from all modules
export * from './array';
export * from './object';
export * from './string';
export * from './number';

// ✅ GOOD - Re-export selectively or keep separate
// Let users import from specific modules
// library/array.js
export function map(arr, fn) { /* ... */ }
export function filter(arr, fn) { /* ... */ }

// library/object.js
export function keys(obj) { /* ... */ }
export function values(obj) { /* ... */ }

// Usage - allows tree shaking
import { map } from 'library/array'; // Only array utils bundled
import { keys } from 'library/object'; // Only object utils bundled

// Or provide multiple entry points in package.json
{
  "exports": {
    "./array": "./dist/array.js",
    "./object": "./dist/object.js"
  }
}
```

---

### **✅ Common Tree Shaking Issues**

```javascript
// ISSUE 1: Babel transforms ES modules to CommonJS
// ❌ BAD babel config
{
  "presets": [
    ["@babel/preset-env", {
      "modules": "commonjs" // Breaks tree shaking!
    }]
  ]
}

// ✅ GOOD babel config
{
  "presets": [
    ["@babel/preset-env", {
      "modules": false // Preserve ES modules
    }]
  ]
}

// ISSUE 2: Using barrel exports (index.js)
// ❌ PROBLEMATIC - Barrel file
// components/index.js
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
// ... 50 more components

// Usage
import { Button } from './components'; // May bundle all components

// ✅ BETTER - Direct imports
import { Button } from './components/Button';

// ISSUE 3: Side effects in imported modules
// ❌ BAD - Module with side effects
// analytics.js
export function track(event) { /* ... */ }

// Side effect - always runs
window.analytics = { track };

// Mark in package.json
{
  "sideEffects": ["./src/analytics.js"]
}

// ISSUE 4: Dynamic property access
// ❌ BAD - Dynamic access prevents tree shaking
const utils = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
};

const operation = 'add';
utils[operation](2, 3); // Tree shaker can't analyze this

// ✅ GOOD - Static imports
import { add } from './utils';
add(2, 3);
```

---

### **✅ Measuring Tree Shaking Effectiveness**

```javascript
// webpack.config.js - Analyze bundle
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      generateStatsFile: true,
      statsOptions: { source: false },
    }),
  ],
  
  // Show which exports are used
  optimization: {
    usedExports: true,
    innerGraph: true, // Advanced tree shaking
  },
  
  stats: {
    // Display information about tree shaking
    usedExports: true,
    providedExports: true,
    optimizationBailout: true,
  },
};

// Build and check stats
// npm run build
// Open bundle-report.html to see what's included
```

#### **Manual Testing**

```bash
# Build production bundle
npm run build

# Check bundle size
ls -lh dist/

# Search for specific function in bundle
grep -r "unusedFunction" dist/

# If function appears, it wasn't tree-shaken
# If it doesn't appear, tree shaking worked!
```

---

### **✅ Library-Specific Tree Shaking**

```javascript
// Example: Tree-shaking lodash

// ❌ BAD - Imports entire lodash (~70KB)
import _ from 'lodash';
_.map([1, 2, 3], x => x * 2);

// ❌ BETTER - But still imports more than needed
import { map } from 'lodash';

// ✅ GOOD - Import specific function
import map from 'lodash/map';

// ✅ BEST - Use lodash-es (ES module version)
import { map } from 'lodash-es';

// Example: Tree-shaking Material-UI

// ❌ BAD - Imports everything
import { Button, TextField, Dialog } from '@mui/material';

// ✅ GOOD - Import from specific paths
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';
import Dialog from '@mui/material/Dialog';

// ✅ BETTER - Use babel plugin
// .babelrc
{
  "plugins": [
    ["import", {
      "libraryName": "@mui/material",
      "libraryDirectory": "",
      "camel2DashComponentName": false
    }]
  ]
}

// Then you can write:
import { Button, TextField } from '@mui/material';
// And it automatically transforms to individual imports
```

---

<a name="asset-optimization"></a>
## 🔴 Asset Optimization

### **Why Optimize Assets?**

```
Unoptimized Assets Impact:
├── Large bundle sizes → Slow initial load
├── Uncompressed images → Bandwidth waste
├── Unoptimized fonts → Render blocking
└── Multiple HTTP requests → Network overhead

Optimized Assets Benefits:
├── 50-80% smaller bundle size
├── Faster page load
├── Better Core Web Vitals
└── Lower bandwidth costs
```

---

### **✅ Image Optimization**

#### **1. Format Selection**

```javascript
// Image format decision tree
const selectImageFormat = (imageType) => {
  const formats = {
    // Photographs
    photos: {
      modern: 'webp', // 25-35% smaller than JPEG
      fallback: 'jpg',
      quality: 80,
    },
    
    // Graphics with transparency
    graphics: {
      modern: 'webp',
      fallback: 'png',
      quality: 85,
    },
    
    // Icons, logos
    icons: {
      vector: 'svg', // Scalable
      raster: 'png',
    },
    
    // Animations
    animations: {
      modern: 'webp', // Animated WebP
      fallback: 'gif',
    },
  };
  
  return formats[imageType];
};
```

#### **2. Responsive Images**

```html
<!-- Modern responsive images -->
<picture>
  <!-- WebP for modern browsers -->
  <source 
    type="image/webp"
    srcset="
      image-320.webp 320w,
      image-640.webp 640w,
      image-1024.webp 1024w
    "
    sizes="(max-width: 640px) 100vw, 50vw"
  />
  
  <!-- JPEG fallback -->
  <source 
    type="image/jpeg"
    srcset="
      image-320.jpg 320w,
      image-640.jpg 640w,
      image-1024.jpg 1024w
    "
    sizes="(max-width: 640px) 100vw, 50vw"
  />
  
  <!-- Fallback for old browsers -->
  <img 
    src="image-640.jpg" 
    alt="Description"
    loading="lazy"
    decoding="async"
  />
</picture>
```

#### **3. Webpack Image Optimization**

```javascript
// webpack.config.js - Image optimization
const ImageMinimizerPlugin = require('image-minimizer-webpack-plugin');

module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|gif|svg)$/i,
        type: 'asset',
      },
    ],
  },
  
  optimization: {
    minimizer: [
      new ImageMinimizerPlugin({
        minimizer: {
          implementation: ImageMinimizerPlugin.imageminMinify,
          options: {
            plugins: [
              // JPEG optimization
              ['mozjpeg', { quality: 80, progressive: true }],
              
              // PNG optimization
              ['pngquant', { quality: [0.6, 0.8], speed: 1 }],
              
              // GIF optimization
              ['gifsicle', { interlaced: true, optimizationLevel: 3 }],
              
              // SVG optimization
              [
                'svgo',
                {
                  plugins: [
                    {
                      name: 'preset-default',
                      params: {
                        overrides: {
                          removeViewBox: false,
                          cleanupIDs: false,
                        },
                      },
                    },
                  ],
                },
              ],
            ],
          },
        },
        
        // Generate WebP versions
        generator: [
          {
            preset: 'webp',
            implementation: ImageMinimizerPlugin.imageminGenerate,
            options: {
              plugins: ['imagemin-webp'],
            },
          },
        ],
      }),
    ],
  },
};
```

#### **4. Vite Image Optimization**

```javascript
// vite.config.js - Image optimization
import { defineConfig } from 'vite';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
  plugins: [
    imagetools({
      defaultDirectives: (url) => {
        // Optimize all images
        return new URLSearchParams({
          format: 'webp;jpg', // Generate both WebP and JPEG
          quality: '80',
          as: 'picture', // Generate <picture> element
        });
      },
    }),
  ],
});

// Usage in components
import imageUrl from './image.jpg?w=400&h=300&format=webp';
import responsiveImage from './photo.jpg?w=400;800;1200&format=webp;jpg&as=picture';

function MyComponent() {
  return (
    <div>
      <img src={imageUrl} alt="Optimized" />
      <Picture {...responsiveImage} alt="Responsive" />
    </div>
  );
}
```

#### **5. Lazy Loading Images**

```typescript
// components/LazyImage.tsx
import { useState, useEffect, useRef } from 'react';

interface LazyImageProps {
  src: string;
  alt: string;
  placeholder?: string;
  className?: string;
}

export function LazyImage({ 
  src, 
  alt, 
  placeholder = 'data:image/svg+xml;base64,...', // Low-res placeholder
  className 
}: LazyImageProps) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);
  
  useEffect(() => {
    if (!imgRef.current) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsInView(true);
          observer.disconnect();
        }
      },
      { rootMargin: '50px' } // Load 50px before visible
    );
    
    observer.observe(imgRef.current);
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <img
      ref={imgRef}
      src={isInView ? src : placeholder}
      alt={alt}
      className={className}
      onLoad={() => setIsLoaded(true)}
      style={{
        opacity: isLoaded ? 1 : 0,
        transition: 'opacity 0.3s',
      }}
      loading="lazy" // Native lazy loading fallback
    />
  );
}
```

---

### **✅ Font Optimization**

#### **1. Font Loading Strategies**

```css
/* Critical font loading */
@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap; /* Show fallback immediately */
}

/* Font loading strategies */
.font-strategies {
  /* auto: Browser default */
  font-display: auto;
  
  /* block: Hide text for ~3s, then show */
  font-display: block;
  
  /* swap: Show fallback immediately, swap when loaded */
  font-display: swap; /* RECOMMENDED */
  
  /* fallback: Hide text for ~100ms, swap if loaded in 3s */
  font-display: fallback;
  
  /* optional: Hide text for ~100ms, may not swap */
  font-display: optional;
}
```

#### **2. Font Subsetting**

```javascript
// webpack.config.js - Font subsetting
const FontminPlugin = require('fontmin-webpack');

module.exports = {
  plugins: [
    new FontminPlugin({
      // Only include characters you need
      text: 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 .,!?',
      
      // Or include specific Unicode ranges
      // glyphs: [0x0020, 0x007E], // Basic Latin
    }),
  ],
};

// Result: Font file size reduced by 70-90%
```

#### **3. Variable Fonts**

```css
/* Traditional: Multiple font files */
@font-face {
  font-family: 'Roboto';
  src: url('/fonts/roboto-regular.woff2');
  font-weight: 400;
}

@font-face {
  font-family: 'Roboto';
  src: url('/fonts/roboto-bold.woff2');
  font-weight: 700;
}

/* Total: ~80KB */

/* Variable font: Single file */
@font-face {
  font-family: 'Roboto Variable';
  src: url('/fonts/roboto-variable.woff2') format('woff2-variations');
  font-weight: 100 900; /* All weights in one file */
}

/* Total: ~45KB */

/* Usage */
.text {
  font-family: 'Roboto Variable';
  font-weight: 350; /* Any value between 100-900 */
}
```

#### **4. Preloading Critical Fonts**

```html
<!-- Preload critical fonts -->
<link 
  rel="preload" 
  href="/fonts/roboto-variable.woff2" 
  as="font" 
  type="font/woff2" 
  crossorigin
/>

<!-- Self-host Google Fonts for better performance -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">

<!-- Better: Self-host -->
<link rel="stylesheet" href="/fonts/fonts.css">
```

---

### **✅ CSS Optimization**

```javascript
// webpack.config.js - CSS optimization
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const PurgeCSSPlugin = require('purgecss-webpack-plugin');
const glob = require('glob');

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  'autoprefixer',
                  'cssnano', // Minify CSS
                ],
              },
            },
          },
        ],
      },
    ],
  },
  
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
    }),
    
    // Remove unused CSS
    new PurgeCSSPlugin({
      paths: glob.sync(`${path.join(__dirname, 'src')}/**/*`, { nodir: true }),
      safelist: {
        standard: ['active', 'show', 'collapse'],
        deep: [/^modal/, /^tooltip/],
        greedy: [/^data-/],
      },
    }),
  ],
  
  optimization: {
    minimizer: [
      new CssMinimizerPlugin({
        minimizerOptions: {
          preset: [
            'default',
            {
              discardComments: { removeAll: true },
              normalizeWhitespace: true,
              colormin: true,
              minifyFontValues: true,
              minifyGradients: true,
            },
          ],
        },
      }),
    ],
  },
};
```

---

### **✅ JavaScript Optimization**

```javascript
// webpack.config.js - JavaScript optimization
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // Remove console.log
            drop_debugger: true, // Remove debugger
            pure_funcs: ['console.info', 'console.debug'], // Remove specific functions
            passes: 2, // Multiple compression passes
          },
          mangle: {
            safari10: true, // Safari 10 bug fix
          },
          format: {
            comments: false, // Remove all comments
            ascii_only: true, // Escape Unicode characters
          },
        },
        extractComments: false, // Don't create separate license file
        parallel: true, // Use multi-process parallel running
      }),
    ],
    
    // Code splitting
    splitChunks: {
      chunks: 'all',
      minSize: 20000,
      maxSize: 244000,
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
        },
      },
    },
  },
};
```

---

<a name="env-config"></a>
## 🔴 Environment Configuration

### **Why Environment Variables?**

```javascript
// ❌ BAD - Hardcoded values
const API_URL = 'https://api.production.com';
const DEBUG = false;
const API_KEY = 'sk_live_12345'; // NEVER hardcode secrets!

// ✅ GOOD - Environment variables
const API_URL = process.env.REACT_APP_API_URL;
const DEBUG = process.env.NODE_ENV === 'development';
const API_KEY = process.env.API_KEY; // From secure env, never committed
```

---

### **✅ Environment Setup**

#### **1. Basic .env Files**

```bash
# .env.local (ignored by git - for secrets)
DATABASE_URL=postgresql://localhost:5432/mydb
API_KEY=sk_live_production_key_12345
JWT_SECRET=super-secret-key

# .env.development (committed - no secrets)
REACT_APP_API_URL=http://localhost:8080
REACT_APP_DEBUG=true
REACT_APP_ANALYTICS_ID=dev-analytics-id

# .env.production (committed - no secrets)
REACT_APP_API_URL=https://api.myapp.com
REACT_APP_DEBUG=false
REACT_APP_ANALYTICS_ID=prod-analytics-id

# .env.test
REACT_APP_API_URL=http://localhost:8080
REACT_APP_DEBUG=true
```

```bash
# .gitignore - Never commit secrets!
.env.local
.env.*.local
.env.production.local
```

#### **2. Webpack Environment Variables**

```javascript
// webpack.config.js
const webpack = require('webpack');
const dotenv = require('dotenv');

// Load environment variables
const env = dotenv.config().parsed;

module.exports = {
  plugins: [
    // Make env variables available in code
    new webpack.DefinePlugin({
      'process.env': JSON.stringify(env),
      
      // Or define specific variables
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
      'process.env.API_URL': JSON.stringify(process.env.API_URL),
      
      // Define constants
      __DEV__: process.env.NODE_ENV !== 'production',
      __PROD__: process.env.NODE_ENV === 'production',
      __VERSION__: JSON.stringify(require('./package.json').version),
    }),
    
    // Alternative: Dotenv plugin
    new Dotenv({
      path: `./.env.${process.env.NODE_ENV}`,
      safe: true, // Load .env.example to verify required variables exist
      systemvars: true, // Load system environment variables
      silent: false, // Warn if variables are missing
    }),
  ],
};

// Usage in code
if (__DEV__) {
  console.log('Development mode');
}

fetch(process.env.API_URL + '/users');
```

#### **3. Vite Environment Variables**

```javascript
// vite.config.js
import { defineConfig, loadEnv } from 'vite';

export default defineConfig(({ mode }) => {
  // Load environment variables
  const env = loadEnv(mode, process.cwd(), '');
  
  return {
    define: {
      __APP_VERSION__: JSON.stringify(env.npm_package_version),
      __API_URL__: JSON.stringify(env.VITE_API_URL),
    },
    
    // Vite automatically loads .env files
    // and exposes variables prefixed with VITE_
  };
});

// .env
VITE_API_URL=http://localhost:8080
VITE_APP_TITLE=My App
DB_PASSWORD=secret123 // NOT exposed (no VITE_ prefix)

// Usage in code
console.log(import.meta.env.VITE_API_URL);
console.log(import.meta.env.VITE_APP_TITLE);
console.log(import.meta.env.MODE); // 'development' or 'production'
console.log(import.meta.env.DEV); // boolean
console.log(import.meta.env.PROD); // boolean
```

---

### **✅ Type-Safe Environment Variables**

```typescript
// src/env.ts - Type-safe environment config
interface EnvironmentConfig {
  apiUrl: string;
  environment: 'development' | 'staging' | 'production';
  debug: boolean;
  analyticsId: string;
  features: {
    newUI: boolean;
    betaFeatures: boolean;
  };
}

function validateEnv(): EnvironmentConfig {
  const required = ['VITE_API_URL', 'VITE_ANALYTICS_ID'];
  
  for (const key of required) {
    if (!import.meta.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
  
  return {
    apiUrl: import.meta.env.VITE_API_URL,
    environment: import.meta.env.MODE as 'development' | 'staging' | 'production',
    debug: import.meta.env.DEV,
    analyticsId: import.meta.env.VITE_ANALYTICS_ID,
    features: {
      newUI: import.meta.env.VITE_FEATURE_NEW_UI === 'true',
      betaFeatures: import.meta.env.VITE_FEATURE_BETA === 'true',
    },
  };
}

export const env = validateEnv();

// Usage - fully type-safe!
import { env } from './env';

fetch(`${env.apiUrl}/users`); // ✅ Type-safe
console.log(env.features.newUI); // ✅ boolean type

// TypeScript will catch errors
env.apiUrl = 'new-url'; // ❌ Error: Cannot assign to 'apiUrl' because it is a read-only property
```

---

### **✅ Environment-Specific Builds**

```json
// package.json - Build scripts for different environments
{
  "scripts": {
    "dev": "vite --mode development",
    "build:dev": "vite build --mode development",
    "build:staging": "vite build --mode staging",
    "build:prod": "vite build --mode production",
    "preview": "vite preview"
  }
}
```

```bash
# .env.development
VITE_API_URL=http://localhost:8080
VITE_DEBUG=true
VITE_FEATURE_BETA=true

# .env.staging
VITE_API_URL=https://staging-api.myapp.com
VITE_DEBUG=true
VITE_FEATURE_BETA=true

# .env.production
VITE_API_URL=https://api.myapp.com
VITE_DEBUG=false
VITE_FEATURE_BETA=false
```

---

### **✅ Runtime Configuration (Advanced)**

```typescript
// config/runtime.ts - Load config at runtime (for Docker, etc.)
interface RuntimeConfig {
  apiUrl: string;
  features: Record<string, boolean>;
}

let config: RuntimeConfig | null = null;

export async function loadRuntimeConfig(): Promise<RuntimeConfig> {
  if (config) return config;
  
  try {
    // Fetch config from server at runtime
    const response = await fetch('/config.json');
    config = await response.json();
    return config;
  } catch (error) {
    console.error('Failed to load runtime config:', error);
    
    // Fallback to build-time config
    return {
      apiUrl: import.meta.env.VITE_API_URL,
      features: {},
    };
  }
}

export function getRuntimeConfig(): RuntimeConfig {
  if (!config) {
    throw new Error('Runtime config not loaded. Call loadRuntimeConfig() first.');
  }
  return config;
}

// App.tsx - Load config on startup
import { loadRuntimeConfig } from './config/runtime';

function App() {
  const [configLoaded, setConfigLoaded] = useState(false);
  
  useEffect(() => {
    loadRuntimeConfig().then(() => {
      setConfigLoaded(true);
    });
  }, []);
  
  if (!configLoaded) {
    return <div>Loading configuration...</div>;
  }
  
  return <MyApp />;
}

// Dockerfile - Mount config at runtime
# Dockerfile
COPY config.${ENVIRONMENT}.json /app/public/config.json
```

---

### **✅ Feature Flags with Environment Variables**

```typescript
// features/flags.ts
interface FeatureFlags {
  newDashboard: boolean;
  darkMode: boolean;
  experimentalEditor: boolean;
  aiAssistant: boolean;
}

export const features: FeatureFlags = {
  newDashboard: import.meta.env.VITE_FEATURE_NEW_DASHBOARD === 'true',
  darkMode: import.meta.env.VITE_FEATURE_DARK_MODE === 'true',
  experimentalEditor: import.meta.env.VITE_FEATURE_EXPERIMENTAL_EDITOR === 'true',
  aiAssistant: import.meta.env.VITE_FEATURE_AI_ASSISTANT === 'true',
};

// components/FeatureFlag.tsx
import { features } from '../features/flags';

interface FeatureFlagProps {
  feature: keyof typeof features;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

export function FeatureFlag({ feature, children, fallback = null }: FeatureFlagProps) {
  if (!features[feature]) {
    return <>{fallback}</>;
  }
  
  return <>{children}</>;
}

// Usage
function Dashboard() {
  return (
    <div>
      <FeatureFlag feature="newDashboard">
        <NewDashboard />
      </FeatureFlag>
      
      <FeatureFlag feature="newDashboard" fallback={<OldDashboard />}>
        <NewDashboard />
      </FeatureFlag>
      
      <FeatureFlag feature="aiAssistant">
        <AIAssistantPanel />
      </FeatureFlag>
    </div>
  );
}
```

---

<a name="interview-questions"></a>
## 🎯 Common Interview Questions

### **Q1: What is tree shaking and how does it work?**

**Answer:**
Tree shaking is the process of removing unused code (dead code elimination) from your final bundle.

**How it works:**
1. **Static Analysis**: Build tool analyzes ES6 imports/exports
2. **Dependency Graph**: Creates graph of what code is actually used
3. **Mark & Sweep**: Marks unused exports and removes them
4. **Minification**: Removes marked code in production build

**Requirements:**
- Must use ES6 modules (`import/export`), not CommonJS (`require`)
- Set `"sideEffects": false` in package.json
- Use production mode
- Babel should preserve ES modules (`modules: false`)

**Example:**
```javascript
// utils.js
export function used() { return 'used'; }
export function unused() { return 'unused'; }

// app.js
import { used } from './utils';
console.log(used());

// Final bundle contains ONLY used(), unused() is removed
```

**Interview tip**: Mention that tree shaking can reduce bundle size by 30-50% for typical applications.

---

### **Q2: Webpack vs Vite - when would you use each?**

**Answer:**

**Use Webpack when:**
- Legacy project with existing Webpack config
- Need Module Federation (micro-frontends)
- Complex build requirements with specific loaders
- Team expertise in Webpack

**Use Vite when:**
- ✅ New projects (recommended)
- Need fast development experience
- Modern browser targets
- React/Vue/Svelte apps
- Want instant HMR

**Key differences:**
```
Development:
- Webpack: Bundles everything upfront (~30s initial)
- Vite: Serves native ESM, bundles on-demand (~2s initial)

HMR Speed:
- Webpack: 2-5 seconds
- Vite: <100ms (nearly instant)

Production:
- Webpack: Uses Webpack bundler
- Vite: Uses Rollup (optimized bundles)
```

**My recommendation**: Use Vite for new projects unless you specifically need Webpack features.

---

### **Q3: How do you optimize bundle size?**

**Answer:**

**My approach:**

1. **Code Splitting**
```javascript
// Route-based splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
```

2. **Tree Shaking**
```javascript
// Use named imports
import { map } from 'lodash-es'; // NOT import _ from 'lodash'
```

3. **Analyze Bundle**
```bash
npm run build -- --analyze
# Identify large dependencies
```

4. **Replace Heavy Libraries**
```javascript
// Instead of moment.js (67KB)
import { formatDate } from 'date-fns'; // 13KB

// Instead of lodash (71KB)
import map from 'lodash-es/map'; // 3KB
```

5. **Compression**
```javascript
// Enable gzip/brotli compression
new CompressionPlugin({ algorithm: 'brotliCompress' });
```

6. **Asset Optimization**
- WebP images instead of JPEG/PNG
- SVG for icons
- Font subsetting

**Real-world result**: Reduced bundle from 1.2MB to 350KB (70% reduction)

---

### **Q4: Explain how module bundlers work**

**Answer:**

**Module bundler process:**

1. **Entry Point**: Start from entry file (index.js)
2. **Dependency Graph**: Recursively parse all imports
3. **Transform**: Process files with loaders/plugins
4. **Bundle**: Combine modules into output files
5. **Optimize**: Minify, tree-shake, split chunks

**Example:**
```javascript
// Input
// index.js
import { add } from './utils';
console.log(add(2, 3));

// utils.js
export const add = (a, b) => a + b;

// Output (simplified)
// bundle.js
(function() {
  const add = (a, b) => a + b;
  console.log(add(2, 3));
})();
```

**Modern vs Traditional:**
- **Traditional (Webpack)**: Bundle everything, serve bundle
- **Modern (Vite)**: Serve native ESM in dev, bundle for production

**Why bundling is needed:**
- Browsers don't understand TypeScript, JSX, etc.
- Reduce HTTP requests (100s of files → few bundles)
- Enable optimizations (minification, tree shaking)
- Support older browsers

---

### **Q5: How do you handle environment variables securely?**

**Answer:**

**Security best practices:**

1. **Never commit secrets**
```bash
# .gitignore
.env.local
.env.*.local
*.env.production
```

2. **Use different files per environment**
```bash
.env.development  # Committed, no secrets
.env.staging      # Committed, no secrets
.env.production   # NOT committed
.env.local        # NOT committed, for secrets
```

3. **Prefix public variables**
```bash
# Can be exposed to frontend
VITE_API_URL=https://api.com

# NEVER exposed (no VITE_ prefix)
DATABASE_PASSWORD=secret123
API_SECRET_KEY=sk_live_123
```

4. **Validate required variables**
```typescript
const required = ['API_URL', 'ANALYTICS_ID'];
for (const key of required) {
  if (!process.env[key]) {
    throw new Error(`Missing: ${key}`);
  }
}
```

5. **CI/CD secrets**
```yaml
# GitHub Actions
- name: Build
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: npm run build
```

**Interview tip**: Emphasize that API keys should NEVER be in frontend code. Use backend proxy instead.

---

<a name="best-practices"></a>
## 📚 Daily Best Practices

### **Build Performance Checklist**

```bash
# Daily optimization routine

# 1. Check bundle size after every major change
npm run build
ls -lh dist/

# 2. Run bundle analyzer weekly
npm run build -- --analyze

# 3. Monitor build time
time npm run build

# 4. Check for outdated dependencies
npm outdated

# 5. Audit for vulnerabilities
npm audit
```

### **Development Workflow**

```javascript
// Project structure for optimal builds
my-app/
├── src/
│   ├── pages/           # Route-based code splitting
│   │   ├── Home/
│   │   ├── Dashboard/
│   │   └── Profile/
│   ├── components/
│   │   ├── Button/
│   │   └── Input/
│   ├── utils/          # Tree-shakeable utilities
│   │   └── index.ts    # Named exports only
│   ├── assets/
│   │   ├── images/     # Optimized images
│   │   └── fonts/      # Subsetted fonts
│   └── config/
│       └── env.ts      # Type-safe env config
├── public/
│   └── config.json     # Runtime config
├── .env.development
├── .env.production
└── vite.config.ts
```

### **Progressive Learning Path**

**Week 1: Webpack Fundamentals**
- [ ] Set up basic Webpack config
- [ ] Configure loaders (Babel, CSS, images)
- [ ] Add plugins (HtmlWebpackPlugin)
- [ ] Set up dev server
- [ ] Implement code splitting

**Week 2: Vite & Modern Tools**
- [ ] Migrate project to Vite
- [ ] Compare build times
- [ ] Configure Vite plugins
- [ ] Set up esbuild
- [ ] Benchmark performance

**Week 3: Optimization**
- [ ] Implement tree shaking
- [ ] Optimize images (WebP, lazy loading)
- [ ] Optimize fonts (subsetting, variable fonts)
- [ ] Analyze and reduce bundle size
- [ ] Set up compression

**Week 4: Environment & Deployment**
- [ ] Configure environment variables
- [ ] Set up multiple environments (dev/staging/prod)
- [ ] Implement feature flags
- [ ] Create production build pipeline
- [ ] Document build process

---

## 🚀 Final Project

**Build a production-ready application with:**

✅ Optimized Webpack/Vite configuration
✅ Code splitting (route-based + vendor)
✅ Tree shaking enabled
✅ Image optimization (WebP, lazy loading)
✅ Font optimization (subsetting)
✅ Bundle size < 200KB (gzipped)
✅ Build time < 30 seconds
✅ Environment variable management
✅ Feature flags system

**Success Metrics:**
- Lighthouse Performance score > 90
- First Contentful Paint < 1.5s
- Time to Interactive < 3s
- Bundle size reduced by 50%+

---

**✅ You are now a build systems expert!** 🎉