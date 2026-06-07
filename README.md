# ZenQuantics 📈

A modern, feature-rich trading journal application built with React, TypeScript, and PocketBase. Inspired by professional trading platforms like Tradezella, Tradervue, and TraderSync.

## ✨ Features

###  Trading Management
- **Trade Logging**: Comprehensive trade entry with all essential fields
- **Real-time Synchronization**: Automatic data sync across devices via PocketBase
- **Trade Analytics**: P&L tracking, win rate, profit factor, and more
- **Trade Status**: Track open and closed positions
- **Risk Management**: R-multiple calculation and risk tracking
- **Multi-Account Support**: Manage and switch between multiple trading accounts

###  Journal & Documentation
- **Daily Journal**: Calendar-based journaling with week navigation, templates, and trade integration
- **Trading Playbooks**: Create and manage trading strategies and setups
- **Trade Notes**: Detailed notes for each trade including emotions and market conditions

###  Analytics & Insights
- **Performance Metrics**: Total P&L, win rate, profit factor, average win/loss
- **Visual Charts**: Performance visualization with Recharts and Lightweight Charts
- **Calendar Integration**: Track trading activity by date
- **Filtering & Sorting**: Advanced trade filtering and sorting capabilities
- **Pattern Performance**: Identify and track which setups perform best
- **Aggregate P&L Widgets**: Cross-account performance views

###  Authentication & Security
- **Email/Password Authentication**: Secure user registration and login
- **User Profiles**: Personalized user experience via PocketBase Auth

###  Data Management
- **CSV Import**: Import trades from broker exports
- **PDF Import**: Parse brokerage statements
- **Export**: Export trade data and reports
- **Data Backup**: Built-in backup and rollback capabilities

###  Market Data
- **Top Gainers**: Real-time top gainer data with caching
- **Futures Data**: Futures market integration
- **Yahoo Finance Proxy**: Volume and price data for traded symbols

##  Getting Started

### Prerequisites

- Node.js (v18 or higher)
- npm
- [PocketBase](https://pocketbase.io/) (v0.22+)

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd zenquantics
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start PocketBase**
   Download and run PocketBase locally:
   ```bash
   ./pocketbase serve
   ```
   PocketBase will be available at `http://127.0.0.1:8090`. Open the admin UI at `http://127.0.0.1:8090/_/` to set up your collections.

4. **Configure environment variables**
   ```bash
   cp .env.example .env
   ```

   Set your PocketBase URL if different from the default:
   ```env
   VITE_POCKETBASE_URL=http://127.0.0.1:8090
   ```

5. **Start the development server**
   ```bash
   npm run dev
   ```

## 🏗️ Tech Stack

### Frontend
- **React 18** - Modern React with hooks
- **TypeScript** - Type-safe development
- **Vite** - Fast build tool and dev server
- **Tailwind CSS** - Utility-first CSS framework

### UI Components
- **shadcn/ui** - Accessible React component library
- **Radix UI** - Low-level UI primitives
- **Mantine** - Additional UI components and hooks
- **Lucide React** - Icon library
- **Recharts** - Composable charting library
- **Lightweight Charts** - TradingView-style financial charts
- **react-grid-layout** - Draggable/resizable dashboard widgets

### Backend & Data
- **PocketBase** - Self-hosted backend with real-time subscriptions, auth, and file storage
- **Serverless API Routes** (`functions/api`) - Cloudflare Pages Functions for server-side market data proxying

### State Management
- **React Context** - Global state management
- **TanStack Query (React Query)** - Server state and caching
- **React Hook Form** - Form state management

### Validation
- **Zod** - TypeScript-first schema validation

## 📁 Project Structure

```
src/
├── components/          # React components
│   ├── ui/             # shadcn/ui base components
│   ├── Dashboard.tsx   # Main dashboard
│   ├── AddTrade.tsx    # Trade entry form
│   ├── DailyJournal.tsx # Daily journal view
│   ├── EnhancedPlaybooks.tsx # Strategy playbooks
│   └── ...
├── clients/
│   └── pocketbaseClient.ts  # PocketBase client instance
├── contexts/           # React contexts
│   ├── AuthContext.tsx # Authentication state
│   └── TradeContext.tsx # Trade data state
├── lib/               # Core utilities and services
│   ├── services/      # PocketBase service layer
│   ├── analytics/     # Analytics calculations
│   ├── trade/         # Trade logic
│   └── ...
├── services/          # External service integrations
├── hooks/             # Custom React hooks
├── pages/             # Page-level components
└── types/             # TypeScript type definitions
functions/
└── api/
    └── top-gainers.js # Cloudflare Pages Function — ChartsWatcher proxy
api/                   # Legacy Vercel-format stubs (unused in production)
```

## 📊 Data Structure

### Trade Model
```typescript
interface Trade {
  id: string;
  symbol: string;
  date: string;
  timeIn: string;
  timeOut?: string;
  side: 'long' | 'short';
  entryPrice: number;
  exitPrice?: number;
  quantity: number;
  pnl?: number;
  commission: number;
  stopLoss?: number;
  takeProfit?: number;
  strategy?: string;
  marketConditions?: string;
  timeframe?: string;
  confidence?: number;
  emotions?: string;
  notes?: string;
  screenshots?: string[];
  status: 'open' | 'closed';
  riskAmount?: number;
  rMultiple?: number;
}
```

##  Deployment

### Build for Production
```bash
npm run build
```

The output is a static site in `dist/` that can be served by any static host (Nginx, Caddy, Vercel, etc.).

### PocketBase in Production
Run PocketBase on your server alongside the static frontend:
```bash
./pocketbase serve --http="0.0.0.0:8090"
```

### API Routes (Market Data)
The `/api` directory contains legacy Vercel-format stubs. The active serverless functions live in `functions/api/` and are written for **Cloudflare Pages Functions** (Workers runtime).

### Deploying to Cloudflare Pages with Wrangler

1. **Install Wrangler**
   ```bash
   npm install -g wrangler
   ```

2. **Authenticate**
   ```bash
   wrangler login
   ```

3. **Build the project**
   ```bash
   npm run build
   ```

4. **Deploy**
   ```bash
   wrangler pages deploy dist --project-name zenquantics
   ```

5. **Set environment variables**
   Set these in the Cloudflare dashboard under **Pages → zenquantics → Settings → Environment Variables**, or via CLI:
   ```bash
   wrangler pages secret put CHARTWATCHERS_USER_ID --project-name zenquantics
   wrangler pages secret put CHARTWATCHERS_API_KEY --project-name zenquantics
   wrangler pages secret put CHARTWATCHERS_TOP_GAINERS_CONFIG_ID --project-name zenquantics
   ```
   Add them to both **Production** and **Preview** environments as needed.

   > These variables must **not** have the `VITE_` prefix — they are server-side only and are never sent to the browser.

6. **Subsequent deploys**
   ```bash
   npm run build && wrangler pages deploy dist --project-name zenquantics
   ```

#### How the Functions Work

Cloudflare Pages automatically routes requests matching `functions/api/<name>.js` to the corresponding Worker. The `GET /api/top-gainers` route is handled by `functions/api/top-gainers.js`, which proxies the ChartsWatcher API server-side so credentials are never exposed to the client. The `public/_redirects` SPA catch-all (`/* /index.html 200`) only applies after Pages Functions are checked, so API routes take priority.

##  Development

### Available Scripts
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build
- `npm run lint` - Run ESLint
- `npm run type-check` - TypeScript type checking
- `npm run test` - Run tests
- `npm run benchmark` - Run performance benchmarks

### Code Style
- **ESLint** - Code linting
- **TypeScript** - Strict type checking
- **Prettier** - Code formatting (recommended)

##  Features Roadmap

### ✅ Completed
- User authentication
- Trade CRUD operations
- Real-time data synchronization via PocketBase
- Dashboard with key metrics and draggable widgets
- Multi-account support
- Daily journal with calendar navigation and templates
- Trading playbooks
- CSV and PDF trade import
- Export and backup
- Market data integration (top gainers, futures, Yahoo Finance)
- Responsive design

### 🚧 In Progress
- Advanced analytics and reporting
- Performance comparison tools

### 📋 Planned
- Mobile app (React Native)
- AI-powered trade insights
- Broker API integrations

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Inspired by [Tradezella](https://tradezella.com/), [Tradervue](https://www.tradervue.com/), and [TraderSync](https://tradersync.com/)
- Built with [shadcn/ui](https://ui.shadcn.com/) components
- Backend powered by [PocketBase](https://pocketbase.io/)
- Icons by [Lucide](https://lucide.dev/)
- Charts by [Recharts](https://recharts.org/) and [Lightweight Charts](https://tradingview.github.io/lightweight-charts/)

---

Made with ❤️ for traders, by traders.
