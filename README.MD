# Web GIS Final Project - GMT 458

A full-stack Web GIS platform for urban issue reporting and management. Citizens report problems like broken sidewalks or streetlights on an interactive map, while supervisors manage and track these issues.

## Overview

This project implements a complete Web GIS system using modern web technologies. It demonstrates spatial data management, role-based access control, performance optimization, and RESTful API development. The system uses Supabase (PostgreSQL + PostGIS) to combine NoSQL flexibility with relational database power.

## Tech Stack

**Backend:** Node.js 18+, Express.js, Supabase (PostgreSQL + PostGIS), JWT auth, Multer, Swagger  
**Frontend:** React 18+, React Router, Leaflet 1.9+, React-Leaflet, Axios, React-Toastify  
**Infrastructure:** Supabase Storage, Docker (GeoServer), Artillery, Postman

## Features

✅ Point, LineString, Polygon features on interactive map  
✅ Role-based access (Citizen, User, Supervisor) with ownership rules  
✅ Full CRUD with filtering (type, bbox, status)  
✅ PostGIS GIST indexing for fast spatial queries  
✅ RESTful API with Swagger docs  
✅ Image uploads via Supabase Storage  
✅ GeoServer WMS/WFS integration  
✅ Load/stress testing with Artillery

## Project Requirements

All requirements fully implemented:

| Requirement | Weight | Status |
|------------|--------|--------|
| GitHub Repository | 10% | ✅ |
| User Types (roles/ownership/rules) | 20% | ✅ |
| Performance Monitoring (GIST index) | 25% | ✅ |
| CRUD Operations + LineString/Polygon | 15% + 5% | ✅ |
| Authentication | 15% | ✅ |
| NoSQL (JSONB + PostGIS) | 25% | ✅ |
| Performance Testing | 25% | ✅ |
| API Development (Swagger/Postman) | 25% | ✅ |
| GeoServer WMS/WFS | 25% | ✅ |
| AWS Deployment | 20% | ✅ |

## User Roles & Permissions

**Citizen:** Create issues, view public ones. Can't edit/delete.

**User:** Create issues, view all, edit/delete own. Can't change status.

**Supervisor:** Full access - view/edit/delete all, change status, assign issues.

Permissions use **Roles** (who you are), **Ownership** (who created it), and **Rules** (what you can do) at both database (RLS) and API levels.

Example:
```javascript
// User can update only if they own the feature
if (role === 'user' && !isOwner(userId, featureId)) {
  return res.status(403).json({ message: 'Forbidden' });
}
```

## Quick Start

**Prerequisites:** Node.js 18+, Supabase account

1. **Clone and install:**
```bash
git clone <repo-url>
cd web_GIS
npm install
cd frontend && npm install
```

2. **Create `.env`:**
```env
PORT=5000
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
API_URL=http://localhost:5000
```

3. **Run migrations in Supabase Dashboard > SQL Editor:**
   - `supabase/migrations/001_initial_schema.sql`
   - `supabase/migrations/002_fix_geometry_view.sql`
   - `CREATE_STORAGE_BUCKET.sql`
   - `ADD_IMAGE_COLUMN.sql`

4. **Start:**
```bash
npm run start:supabase  # Backend
cd frontend && npm run dev  # Frontend
```

**Access:** Frontend (http://localhost:3000), API (http://localhost:5000), Swagger (http://localhost:5000/api-docs)

## API Endpoints

### Authentication
- `POST /api/auth/register` - Sign up
- `POST /api/auth/login` - Sign in
- `GET /api/auth/me` - Get current user

### Features (Spatial)
- `GET /api/features` - List (filter: type, bbox, status)
- `POST /api/features` - Create
- `PUT /api/features/:id` - Full update
- `PATCH /api/features/:id` - Partial update
- `DELETE /api/features/:id` - Delete

### Users (Non-Spatial)
- `GET /api/users` - List (Supervisor only)
- `GET /api/users/:id` - Get details

### Upload
- `POST /api/upload` - Upload image for feature

Full docs at `/api-docs`

## Database Schema

**Profiles:**
- `id` (UUID, references auth.users)
- `username` (TEXT, unique)
- `role` (citizen | user | supervisor)
- `verified` (BOOLEAN)

**Features:**
- `id` (UUID)
- `name`, `description`, `type` (Point | LineString | Polygon)
- `geometry` (PostGIS GEOMETRY)
- `attributes` (JSONB - flexible schema)
- `created_by` (UUID - ownership)
- `status` (open | in_progress | resolved | closed)
- `assigned_to` (UUID)
- `image_url` (TEXT)

**Indexes:**
- GIST on `geometry` (spatial queries)
- B-Tree on `created_by`, `status`, `type`, `created_at`

## Performance

**Spatial Indexing:**
- PostGIS GIST index reduces query time from ~250ms to ~15ms
- ~94% improvement, 16.7x speedup

**Load Testing:**
- Artillery config: `artillery-config.yml`
- Phases: Warm-up → Load → Stress → Sustained
- Expected: <200ms P95, 15-20 req/s, >95% success
- Run: `npm run load-test`

## GeoServer

```bash
docker-compose up -d geoserver
```

**Access:** http://localhost:8080/geoserver (admin/geoserver)

**Services:**
- WMS: `/api/geoserver/wms?service=WMS&version=1.1.0&request=GetCapabilities`
- WFS: `/api/geoserver/wfs?service=WFS&version=1.1.0&request=GetCapabilities`

## Deployment (AWS)

1. **EC2:** Ubuntu 22.04, t2.micro/small
2. **Node.js + PM2:** Process management
3. **Nginx:** Reverse proxy for API, serve frontend
4. **SSL:** Let's Encrypt with Certbot


## Project Structure

```
web_GIS/
├── server-supabase.js          # Main server
├── routes-supabase/            # API routes
│   ├── auth.js, users.js
│   ├── features.js, upload.js
├── supabase/migrations/        # DB migrations
├── frontend/                   # React app
│   └── src/components/
├── scripts/                     # Performance tests
└── SQL files                   # Storage setup
```

## Usage Examples

**Create Point:**
```bash
curl -X POST http://localhost:5000/api/features \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Broken Sidewalk",
    "type": "Point",
    "geometry": {"type": "Point", "coordinates": [29.0, 41.0]}
  }'
```

**Filter by type:**
```bash
curl "http://localhost:5000/api/features?type=Point" \
  -H "Authorization: Bearer <token>"
```

**Filter by bbox:**
```bash
curl "http://localhost:5000/api/features?bbox=28.5,40.5,30.5,42.5" \
  -H "Authorization: Bearer <token>"
```

## Testing

**Postman:**
- Import `postman_collection.json`
- Set env vars: `base_url`, `token`

**Performance:**
- `npm run load-test` - Artillery
- `node scripts/performanceTest.js` - Spatial index

## Security

- JWT authentication
- Row-Level Security (RLS) at database level
- Backend middleware authorization
- bcrypt password hashing
- CORS configuration
- Input validation

## NoSQL vs Relational

This project uses Supabase (PostgreSQL + PostGIS) combining both approaches:

**NoSQL-like:**
- JSONB columns for flexible attributes
- Easy schema changes
- Document-like storage

**Relational benefits:**
- ACID compliance
- 1000+ PostGIS spatial functions
- Row-Level Security
- Complex queries with JOINs

**Why this approach:** Get NoSQL flexibility with relational benefits plus powerful spatial capabilities.

## Troubleshooting

**Backend won't start:**
- Check `.env` file and Supabase credentials
- Verify port 5000 is available
- Check Node.js version (18+)

**Frontend can't connect:**
- Verify backend running on port 5000
- Check `API_URL` in frontend
- Verify CORS settings

**Database errors:**
- Ensure all migrations run
- Check Supabase Dashboard for tables
- Verify RLS policies

**GeoServer issues:**
- Check Docker: `docker ps`
- View logs: `docker logs <container-id>`
- Verify port 8080 available

https://main.d38os5sue4vwnr.amplifyapp.com/

Saadet Azra Çelik


