# Quickstart Guide: Equipment Manager Mini App

**Branch**: `001-equipment-manager` | **Date**: 2026-01-05 | **Plan**: [plan.md](./plan.md)

## Prerequisites

- **Node.js 20 LTS** or later
- **npm** 10.x or later
- **Telegram Bot Token** (from [@BotFather](https://t.me/BotFather))
- **ngrok** or similar tunneling tool (for local development)

## Project Setup

### 1. Clone and Install

```bash
# Clone the repository
git clone <repository-url>
cd equipment-manager

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### 2. Environment Configuration

Create `.env` files for both backend and frontend:

**backend/.env**
```env
# Server
PORT=3000
NODE_ENV=development

# Database
DATABASE_URL="file:./dev.db"

# Telegram Bot
BOT_TOKEN=your_bot_token_here

# Frontend URL (for CORS and Mini App)
WEBAPP_URL=https://your-ngrok-url.ngrok.io

# File Upload
MAX_FILE_SIZE=10485760
UPLOAD_DIR=./uploads
```

**frontend/.env**
```env
# API Base URL
VITE_API_URL=https://your-ngrok-url.ngrok.io/api

# Telegram WebApp (automatically provided by Telegram)
# VITE_TELEGRAM_WEBAPP - not needed, use window.Telegram.WebApp
```

### 3. Database Setup

```bash
cd backend

# Generate Prisma client
npx prisma generate

# Create database and run migrations
npx prisma migrate dev --name init

# (Optional) Seed with sample data
npx prisma db seed
```

### 4. Create Telegram Bot

1. Open Telegram and message [@BotFather](https://t.me/BotFather)
2. Send `/newbot` and follow the prompts
3. Copy the bot token to `backend/.env`
4. Send `/setmenubutton` to configure the Mini App button:
   - Select your bot
   - Choose "Configure menu button"
   - Enter button text: "Open App"
   - Enter URL: Your ngrok URL (e.g., `https://abc123.ngrok.io`)

### 5. Start Development Servers

**Terminal 1 - Start ngrok tunnel:**
```bash
ngrok http 3000
```
Copy the HTTPS URL and update both `.env` files.

**Terminal 2 - Start backend:**
```bash
cd backend
npm run dev
```

**Terminal 3 - Start frontend:**
```bash
cd frontend
npm run dev
```

### 6. Access the App

1. Open Telegram and find your bot
2. Send `/start` to the bot
3. Tap "Open App" button to launch the Mini App

## Project Structure

```text
equipment-manager/
├── backend/
│   ├── src/
│   │   ├── api/              # Express routes
│   │   │   ├── routes/       # Route definitions
│   │   │   └── middleware/   # Auth, validation, error handling
│   │   ├── services/         # Business logic
│   │   ├── bot/              # Telegram bot handlers
│   │   └── utils/            # Helpers (auth, files, QR)
│   ├── prisma/
│   │   ├── schema.prisma     # Database schema
│   │   ├── seed.ts           # Seed data
│   │   └── migrations/       # Database migrations
│   ├── uploads/              # File storage
│   └── tests/
│
├── frontend/
│   ├── src/
│   │   ├── components/       # Vue components
│   │   ├── pages/            # Route views
│   │   ├── stores/           # Pinia stores
│   │   ├── services/         # API client
│   │   └── composables/      # Vue composables
│   ├── public/
│   └── tests/
│
└── specs/                    # Feature specifications
    └── 001-equipment-manager/
```

## Development Workflow

### Running Tests

```bash
# Backend tests
cd backend
npm test

# Frontend tests
cd frontend
npm test

# Run specific test file
npm test -- path/to/test.spec.ts
```

### Database Operations

```bash
# View database in Prisma Studio
cd backend
npx prisma studio

# Create new migration after schema changes
npx prisma migrate dev --name describe_change

# Reset database (development only)
npx prisma migrate reset
```

### Code Quality

```bash
# Lint code
npm run lint

# Format code
npm run format

# Type check
npm run typecheck
```

## Common Development Tasks

### Adding a New API Endpoint

1. Define route in `backend/src/api/routes/`
2. Add Zod schema for validation
3. Implement service logic in `backend/src/services/`
4. Add tests in `backend/tests/`
5. Update OpenAPI spec in `specs/001-equipment-manager/contracts/`

### Adding a New Vue Component

1. Create component in `frontend/src/components/`
2. Use Vuetify components for UI
3. Add to page in `frontend/src/pages/`
4. Add tests in `frontend/tests/`

### Testing QR Scanning

For local development, QR scanning requires Telegram context. Options:

1. **Use ngrok**: Telegram Mini Apps require HTTPS
2. **Test on mobile**: Open bot in Telegram mobile app
3. **Mock scanner**: In development mode, provide manual ID entry

### Debugging Tips

**Backend:**
- Check `backend/logs/` for error logs
- Use Prisma Studio to inspect database
- Enable verbose logging: `DEBUG=* npm run dev`

**Frontend:**
- Open Telegram Mini App in debug mode (long press menu button)
- Use Vue DevTools browser extension
- Check Network tab for API calls

## Deployment

### Architecture: Backend Serves Frontend

This project uses a **unified deployment** model where the Node.js backend serves both:
- API endpoints at `/api/*`
- Vue frontend (built static files) at `/*`

**Why not GitHub Pages?** Unlike the [easy-qr-scan-bot](https://github.com/MBoretto/easy-qr-scan-bot) reference (which uses Telegram Cloud Storage and has no backend), this app requires a Node.js server for:
- SQLite database access (shared equipment data)
- Server-side Telegram initData validation
- File upload handling

### Local/On-Premise Deployment

```bash
# 1. Build frontend
cd frontend
npm run build  # Creates frontend/dist/

# 2. Backend will serve this build
cd ../backend

# 3. Configure production environment
cat > .env << EOF
NODE_ENV=production
PORT=3000
DATABASE_URL="file:./data/prod.db"
BOT_TOKEN=your_bot_token_here
WEBAPP_URL=https://your-domain.com
MAX_FILE_SIZE=10485760
UPLOAD_DIR=./uploads
EOF

# 4. Initialize production database
npx prisma migrate deploy
npx prisma db seed  # Optional: seed initial data

# 5. Start server (serves both API and frontend)
npm start
```

**HTTPS Requirement**: Use ngrok for development or configure SSL certificate for production:

```bash
# Development with ngrok
ngrok http 3000

# Or production with custom domain
# Configure reverse proxy (nginx/Apache) with SSL certificate
```

The backend serves both API and built frontend. The Express server configuration:

```javascript
// backend/src/server.js structure
app.use('/api', apiRouter);                     // API endpoints
app.use(express.static('../frontend/dist'));    // Static frontend
app.get('*', (req, res) => {                    // SPA fallback
  res.sendFile('frontend/dist/index.html');
});
```

### Heroku

```bash
# Login to Heroku
heroku login

# Create app
heroku create equipment-manager

# Set environment variables
heroku config:set BOT_TOKEN=your_token
heroku config:set WEBAPP_URL=https://equipment-manager.herokuapp.com

# Deploy
git push heroku main
```

### Environment Variables (Production)

| Variable | Description |
|----------|-------------|
| `NODE_ENV` | `production` |
| `PORT` | Server port (Heroku sets automatically) |
| `DATABASE_URL` | SQLite file path |
| `BOT_TOKEN` | Telegram bot token |
| `WEBAPP_URL` | Public URL of the Mini App |
| `MAX_FILE_SIZE` | Max upload size in bytes |

## Troubleshooting

### "User not found" error
- User must be added by an admin first
- Check if user exists in database via Prisma Studio

### QR scanner doesn't open
- Ensure using Telegram mobile app (not desktop)
- Check WebApp API version (requires 6.9+)
- Verify HTTPS URL is configured

### Files not uploading
- Check `UPLOAD_DIR` exists and is writable
- Verify file size is under `MAX_FILE_SIZE`
- Check MIME type is allowed

### initData validation fails
- Ensure `BOT_TOKEN` is correct
- Check initData hasn't expired (valid for 1 hour)
- Verify request includes `X-Telegram-Init-Data` header

## Resources

- [Telegram Mini Apps Documentation](https://core.telegram.org/bots/webapps)
- [Vue 3 Documentation](https://vuejs.org/)
- [Vuetify 3 Documentation](https://vuetifyjs.com/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Project Specification](./spec.md)
- [API Reference](./contracts/openapi.yaml)
