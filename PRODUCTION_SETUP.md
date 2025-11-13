# 🚀 Zynga Poker Panel - Production Setup Guide

## ✅ Quick Start (Production Ready)

### Step 1: Database Setup (MongoDB Atlas)

```bash
cd backend
npm run seed
```

**Ye script:**
- ✅ Saare demo/test/mock accounts delete karega
- ✅ Real registered players ko preserve karega
- ✅ Naya admin account create karega: `demo_admin@zynga.com` / `Demo@1234`

### Step 2: Verify Database

```bash
npm run verify-db
```

**Ye check karega:**
- ✅ Admin account exists hai ya nahi
- ✅ Password correct hai ya nahi
- ✅ Demo accounts clean hain ya nahi

### Step 3: Backend Start (Local Testing)

```bash
# Development mode
npm run dev

# Production mode
npm start
```

**Backend chalta hai:** `http://localhost:5000`

### Step 4: Frontend Start

```bash
cd frontend
npm run dev
```

**Frontend chalta hai:** `http://localhost:3000`

### Step 5: Test Admin Login

**Credentials:**
- Email: `demo_admin@zynga.com`
- Password: `Demo@1234`

## 🔧 Production Deployment (Railway/Vercel)

### Backend Deployment:

1. **Railway/Vercel pe deploy karo:**
   - Backend code push karo
   - Environment variables set karo:
     - `MONGO_URI` - MongoDB Atlas connection string
     - `JWT_SECRET` - Random secret key (minimum 32 characters)
     - `JWT_EXPIRE` - Token expiry (default: "1d")
     - `NODE_ENV` - "production"

2. **Database seed karo:**
   ```bash
   npm run seed
   ```

3. **Backend URL note karo:**
   - Example: `https://your-backend.railway.app`

### Frontend Deployment:

1. **Environment variable set karo:**
   - Create `.env` file ya Vercel environment variables:
   ```
   VITE_API_URL=https://your-backend.railway.app/api
   ```

2. **Frontend deploy karo:**
   ```bash
   npm run build
   # Deploy to Vercel/Netlify
   ```

## 🐛 Common Issues & Solutions

### Issue 1: "No token" or 401 Errors

**Problem:** Backend properly deployed nahi hai ya routes missing hain.

**Solution:**
1. Check backend logs
2. Verify backend URL correct hai
3. Check environment variables (JWT_SECRET, MONGO_URI)
4. Verify routes exist: `/api/login`, `/api/register`

### Issue 2: Admin Login Nahi Ho Raha

**Problem:** Admin account database mein nahi hai.

**Solution:**
```bash
cd backend
npm run seed
npm run verify-db
```

### Issue 3: Player Registration 401 Error

**Problem:** Backend public routes ko block kar raha hai.

**Solution:**
- Check `backend/routes/authRoutes.js` - routes public hain
- Check `frontend/src/services/api.js` - public routes don't send auth headers

### Issue 4: Token Not Stored

**Problem:** Frontend token store nahi kar raha.

**Solution:**
- Check browser console for errors
- Verify `AuthContext.jsx` properly saves token
- Check localStorage in browser DevTools

## 📋 Production Checklist

- [ ] MongoDB Atlas connected
- [ ] Seed script run kiya (`npm run seed`)
- [ ] Database verified (`npm run verify-db`)
- [ ] Backend environment variables set (JWT_SECRET, MONGO_URI)
- [ ] Backend deployed and running
- [ ] Frontend environment variable set (VITE_API_URL)
- [ ] Frontend deployed
- [ ] Admin login tested
- [ ] Player registration tested
- [ ] Player login tested
- [ ] Token authentication working

## 🔐 Admin Credentials

**Email:** `demo_admin@zynga.com`  
**Password:** `Demo@1234`

**⚠️ IMPORTANT:** Production mein password change karna recommended hai!

## 📞 Support

Agar koi issue ho:
1. Check backend logs
2. Check frontend console
3. Run `npm run verify-db` to check database
4. Verify environment variables

---

**Made with ❤️ for Production**

