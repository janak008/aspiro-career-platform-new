# Aspiro Project Setup Guide

Complete setup guide for the Aspiro AI-Powered Career Recommendation Platform.

## 🚀 Quick Start

### Option 1: Automated Setup (Recommended)
```bash
# Windows
.\start.bat

# Linux/Mac
chmod +x start.sh
./start.sh
```

### Option 2: Manual Setup
```bash
# 1. Install all dependencies
npm run install-all

# 2. Start all services
npm run dev

# 3. Test integration (in new terminal)
npm test
```

## 📋 Prerequisites

- **Node.js** (v16+)
- **Python** (v3.8+)
- **MongoDB** (local or cloud)
- **Git**

## 🌐 Service Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │    Backend      │    │   ML Service   │
│   React.js      │◄──►│   Node.js       │◄──►│   Python        │
│   Port: 5173    │    │   Port: 5000    │    │   Port: 5001    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 ▼
                    ┌─────────────────┐
                    │    MongoDB      │
                    │   Port: 27017   │
                    └─────────────────┘
```

## 🔧 Environment Configuration

### Backend (.env)
```env
NODE_ENV=development
PORT=5000
MONGODB_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
ML_SERVICE_URL=http://localhost:5001
```

### Frontend (.env)
```env
VITE_API_URL=http://localhost:5000/api
VITE_ML_API_URL=http://localhost:5001/api
```

### ML Service (.env)
```env
FLASK_ENV=development
PORT=5001
BACKEND_URL=http://localhost:5000
```

## 🛠️ Individual Service Setup

### Backend Setup
```bash
cd backend
npm install
npm run dev
```

### Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

### ML Service Setup
```bash
cd ml-service
pip install -r requirements.txt
python train_models.py
python app.py
```

## 🧪 Testing

### Integration Tests
```bash
# Test all services together
npm test

# Test individual services
cd ml-service && python test_service.py
```

### Manual Testing
1. **Frontend**: http://localhost:5173
2. **Backend API**: http://localhost:5000/api/health
3. **ML Service**: http://localhost:5001/api/health

## 🐳 Docker Deployment

```bash
# Build and start all services
docker-compose up --build

# Run in background
docker-compose up -d

# Stop services
docker-compose down
```

## 📊 Features Overview

### ✅ Career Recommendation System
- AI-powered career suggestions
- Random Forest + KNN algorithms
- Real-time predictions
- User profile analysis

### ✅ Resume Analyzer
- PDF/DOCX parsing
- NLP entity extraction
- Job matching with similarity scoring
- Comprehensive feedback

### ✅ User Management
- Authentication & authorization
- Profile management
- History tracking
- Role-based access

### ✅ Job Management
- Job posting & search
- Application tracking
- Recruiter dashboard
- Advanced filtering

## 🔍 API Endpoints

### Authentication
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `GET /api/auth/me` - Get current user

### Career Services
- `POST /api/careers/suggestions` - Get career recommendations
- `POST /api/careers/report` - Generate career report
- `GET /api/careers/history` - Get recommendation history

### Resume Services
- `POST /api/resume/analyze` - Analyze resume
- `GET /api/resume/history` - Get analysis history

### ML Services (Direct)
- `POST /api/career/recommend` - Direct ML career recommendation
- `POST /api/resume/analyze` - Direct ML resume analysis

## 🚨 Troubleshooting

### Common Issues

#### Port Conflicts
```bash
# Check if ports are in use
netstat -an | findstr :5000
netstat -an | findstr :5001
netstat -an | findstr :5173

# Kill processes if needed
taskkill /PID <process_id> /F
```

#### Python Dependencies
```bash
# If pip install fails, try:
pip install --upgrade pip
pip install -r requirements.txt --no-cache-dir
```

#### MongoDB Connection
```bash
# Check MongoDB status
mongosh --eval "db.adminCommand('ismaster')"

# Or use MongoDB Compass for GUI
```

#### ML Models Not Training
```bash
cd ml-service
python train_models.py
# Check for error messages and install missing dependencies
```

## 📈 Performance Optimization

### Development
- Use `npm run dev` for hot reloading
- Enable MongoDB indexing
- Use Redis for caching (optional)

### Production
- Build frontend: `npm run build`
- Use PM2 for process management
- Enable gzip compression
- Use CDN for static assets

## 🔐 Security Considerations

- JWT tokens for authentication
- Input validation and sanitization
- Rate limiting enabled
- CORS properly configured
- File upload restrictions
- Environment variables for secrets

## 📝 Development Workflow

1. **Start Development**
   ```bash
   npm run dev
   ```

2. **Make Changes**
   - Frontend: Hot reload enabled
   - Backend: Nodemon auto-restart
   - ML Service: Manual restart needed

3. **Test Changes**
   ```bash
   npm test
   ```

4. **Build for Production**
   ```bash
   npm run build
   ```

## 🤝 Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature/amazing-feature`
3. Make changes and test: `npm test`
4. Commit changes: `git commit -m 'Add amazing feature'`
5. Push to branch: `git push origin feature/amazing-feature`
6. Submit pull request

## 📄 License

MIT License - see LICENSE file for details.

## 🆘 Support

If you encounter issues:

1. Check this setup guide
2. Run integration tests: `npm test`
3. Check service logs
4. Verify environment configurations
5. Ensure all prerequisites are installed

For additional help, check the individual service README files:
- `backend/README.md`
- `frontend/README.md`
- `ml-service/README.md`