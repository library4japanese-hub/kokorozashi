# Kokorozashi Deployment Guide

## Architecture Overview

Kokorozashi is a full-stack TypeScript application:
- **Frontend**: React 19 + Vite + Tailwind CSS
- **Backend**: Express.js server with TypeScript
- **Database**: Supabase (PostgreSQL)
- **Package Manager**: npm

The application is located in the `kokorozashi-source-2026-04-18/` directory.

## Quick Start: Deploy to Vercel

### Step 1: Prerequisites
- Vercel account (free tier available)
- GitHub account with repository access
- Node.js 18+ installed locally

### Step 2: Connect GitHub to Vercel
1. Go to [vercel.com](https://vercel.com)
2. Click "New Project"
3. Select "Import Git Repository"
4. Authorize GitHub and select `library4japanese-hub/Kokorozashi`
5. Click "Import"

### Step 3: Configure Project Settings

In Vercel dashboard for your project:

1. **Root Directory**: Set to `kokorozashi-source-2026-04-18`
2. **Build Command**: `npm run build`
3. **Output Directory**: `dist`
4. **Install Command**: `npm ci`

### Step 4: Add Environment Variables

In Vercel dashboard → Settings → Environment Variables, add:

```
GEMINI_API_KEY          = sk_****
SUPABASE_URL            = https://your-project.supabase.co
SUPABASE_ANON_KEY       = eyJhbGc...
SUPABASE_SERVICE_ROLE_KEY = eyJhbGc...
STRIPE_SECRET_KEY       = sk_test_**** (optional)
ALLOWED_ORIGINS         = https://kokorozashi.vercel.app
NODE_ENV                = production
```

**Note**: Set each variable for "Production", "Preview", and "Development" environments.

### Step 5: Deploy
Click the "Deploy" button. Vercel will:
1. Clone your repository
2. Install dependencies
3. Build the project
4. Deploy to CDN

Your app will be live at `https://kokorozashi.vercel.app` (or your custom domain).

---

## Automated Deployments with GitHub Actions

A GitHub Actions workflow has been included (`.github/workflows/vercel-deploy.yml`) for automated testing and deployment.

### Prerequisites for CI/CD

1. **Vercel Token**: 
   - Go to Vercel Settings → Tokens
   - Create a new token, copy it

2. **Vercel Project IDs**:
   - Go to your Vercel project Settings
   - Copy `Project ID` and `Org ID`

3. **Add GitHub Secrets**:
   - Go to GitHub repo → Settings → Secrets and variables → Actions
   - Click "New repository secret" and add:

| Secret Name | Value |
|---|---|
| VERCEL_TOKEN | Your Vercel token |
| VERCEL_ORG_ID | Your Vercel Organization ID |
| VERCEL_PROJECT_ID | Your Vercel Project ID |
| GEMINI_API_KEY | Your Gemini API key |
| SUPABASE_URL | Your Supabase project URL |
| SUPABASE_ANON_KEY | Your Supabase anon key |
| SUPABASE_SERVICE_ROLE_KEY | Your Supabase service role key |
| STRIPE_SECRET_KEY | (Optional) Your Stripe secret key |

### How It Works

- **Pull Requests**: Creates preview deployments automatically
- **Push to main**: Deploys to production automatically
- **Runs tests**: Lint and build checks before deployment
- **Only deploys on changes**: Skips if no changes to source code

### View Deployment Status

In GitHub:
1. Go to your repo → "Actions" tab
2. Click on the latest workflow run
3. View logs and deployment status

---

## Manual Deployment with Vercel CLI

### Local Testing Before Deploy

```bash
cd kokorozashi-source-2026-04-18

# Install dependencies
npm ci

# Run type checking
npm run lint

# Build locally
npm run build

# Preview the build
npm run preview
```

### Deploy Using CLI

```bash
# Install Vercel CLI globally
npm install -g vercel

# Navigate to project directory
cd kokorozashi-source-2026-04-18

# Login to Vercel
vercel login

# Deploy (preview)
vercel

# Deploy to production
vercel --prod
```

---

## Database Setup: Supabase

### Initial Setup

1. Create Supabase account at [supabase.com](https://supabase.com)
2. Create a new project
3. Copy your project URL and API keys

### Run Database Schema

1. In Supabase dashboard, go to SQL Editor
2. Create a new query
3. Copy and paste the contents of `kokorozashi-source-2026-04-18/supabase_setup.sql`
4. Click "Run"

This creates the required tables:
- `kokorozashi_news` - News articles for JLPT practice
- `profiles` - User profiles with admin roles
- Other required tables

### Verify Setup

```bash
# In your application
curl https://your-app.vercel.app/api/news
# Should return: [{ title, content, difficulty, ... }]
```

---

## Testing the Deployment

After deployment, test these endpoints:

### Frontend
- [x] App loads at root `/`
- [x] No 404 errors
- [x] Responsive design works

### API Endpoints

```bash
# Health check
curl https://your-app.vercel.app/api/health

# Fetch news
curl https://your-app.vercel.app/api/news

# Sync news from NHK
curl -X POST https://your-app.vercel.app/api/news/sync
```

### Features to Test
- [ ] Frontend loads without errors
- [ ] News displays correctly
- [ ] Gemini AI features work (if configured)
- [ ] Stripe checkout works (if enabled)
- [ ] Admin features accessible to authenticated users

---

## Environment Variables Reference

| Variable | Required | Type | Description |
|---|---|---|---|
| GEMINI_API_KEY | Yes | API Key | Google Gemini API key for AI features |
| SUPABASE_URL | Yes | URL | Supabase project URL |
| SUPABASE_ANON_KEY | Yes | API Key | Supabase anonymous key |
| SUPABASE_SERVICE_ROLE_KEY | Yes | API Key | Supabase service role key (admin access) |
| STRIPE_SECRET_KEY | No | API Key | Stripe secret key for donations |
| NODE_ENV | Yes | String | Set to "production" on Vercel |
| PORT | No | Number | Port number (Vercel sets automatically) |
| ALLOWED_ORIGINS | Yes | String | Comma-separated CORS allowed origins |

---

## Troubleshooting

### Build Fails: "Cannot find module"
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
npm run build
```

### "Supabase not configured" Error
- Verify SUPABASE_URL and SUPABASE_SERVICE_ROLE_KEY are set
- Check Supabase tables exist (run supabase_setup.sql)
- Verify environment variables in Vercel dashboard

### API Routes Return 404
- Ensure Express server is running on production
- Check that build output includes `dist/` folder
- Verify `vercel.json` has correct root directory

### CORS Errors
- Update ALLOWED_ORIGINS to include your Vercel domain
- Format: `https://kokorozashi.vercel.app,https://preview-branch.vercel.app`

### Slow Build Times
- Reduce `node_modules` size by removing unused packages
- Use `npm ci` instead of `npm install`
- Check Vercel build logs for bottlenecks

---

## Monitoring Deployments

### Vercel Dashboard
- View all deployments and rollback if needed
- Monitor performance and analytics
- Check real-time logs

### GitHub Actions
- View workflow runs and logs
- Set up status checks for PRs
- Get deployment notifications

### Application Logs
```bash
# View Vercel logs
vercel logs

# Stream live logs
vercel logs --follow
```

---

## Custom Domain Setup

1. In Vercel dashboard → Settings → Domains
2. Add your custom domain (e.g., kokorozashi.com)
3. Update DNS records as instructed
4. SSL certificate auto-generated

---

## Rollback to Previous Deployment

In Vercel dashboard:
1. Go to Deployments
2. Find the deployment to restore
3. Click "Redeploy"

Or via CLI:
```bash
vercel rollback
```

---

## Next Steps

1. ✅ Deploy to Vercel
2. ✅ Configure Supabase database
3. ✅ Add environment variables
4. ✅ Set up GitHub Actions CI/CD
5. Consider: Custom domain, monitoring, scaling

For detailed Vercel documentation: [vercel.com/docs](https://vercel.com/docs)