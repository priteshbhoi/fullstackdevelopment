```markdown
# Standard Operating Procedure (SOP): Full-Stack Laravel Lifecycle & Project Management Guide

This document defines the complete end-to-end framework for developing, managing, and deploying Laravel applications. Designed for Full-Stack Engineers acting as Project Managers, it establishes a **Schema-First, Zero-Data-Loss** workflow across local environments, GitHub version control, and Hostinger production servers.

---

## 1. Architectural Foundation (The 3-Tier Structure)

To prevent code corruption, security leaks, or database overwrites, every application operates across three strictly isolated environments:

| Environment | Role & Scope | Database Connection | Code State |
| :--- | :--- | :--- | :--- |
| **1. Local (PC)** | Active feature development, UI prototyping, and testing. | Local MySQL / phpMyAdmin (`127.0.0.1`) | Feature branches (`feature/*`) |
| **2. Repository (GitHub)** | Single source of truth, code reviews, and version tracking. | *None (Only `.env.example` stored)* | Consolidated `main` branch |
| **3. Production (Hostinger)** | Live client website serving real users. | Production MySQL / phpMyAdmin | Pulled strictly from GitHub `main` |

---

## 2. Phase 1 — Project Initialization & Local Environment Setup

### Step 1: Create Project Structure
Inside your target project folder on your local machine:

```bash
# Install Laravel directly into current empty directory
composer create-project laravel/laravel .

```

### Step 2: Configure Environment (`.env`)

Open `.env` in your editor and update local MySQL settings (connecting to XAMPP / local phpMyAdmin):

```env
APP_NAME="BiztacsApp"
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=[http://127.0.0.1:8000](http://127.0.0.1:8000)

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=biztacs_db
DB_USERNAME=root
DB_PASSWORD=

```

Generate the secret encryption key:

```bash
php artisan key:generate

```

### Step 3: Validate Security (`.gitignore`)

Ensure `.gitignore` is present in your root directory to prevent pushing credentials or heavy dependencies to version control:

```gitignore
/vendor
/node_modules
/public/storage
.env
.env.backup
.env.production

```

---

## 3. Phase 2 — Schema-First Development Workflow

All features begin with database modeling. Never alter database tables manually inside phpMyAdmin; use Laravel Migrations to keep database changes tracked in code.

```
┌────────────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐
│  1. Create Migration   │ ───► │  2. Run Migration      │ ───► │  3. Build Application  │
│  Define Schema in Code │      │  `php artisan migrate` │      │  Models, Controllers & │
└────────────────────────┘      └────────────────────────┘      └────────────────────────┘

```

### Step 1: Generate Model and Migration Together

When building a new module (e.g., `Review`):

```bash
php artisan make:model Review -m

```

### Step 2: Define Schema in Migration File

Open `database/migrations/xxxx_xx_xx_create_reviews_table.php` and define table structures inside the `up()` method:

```php
public function up(): void
{
    Schema::create('reviews', function (Blueprint $table) {
        $table->id();                                    // Primary Key
        $table->foreignId('user_id')->constrained();     // Foreign Key
        $table->string('title');                         // Text input
        $table->text('comment');                         // Textarea input
        $table->integer('rating')->default(5);           // Integer rating
        $table->boolean('is_approved')->default(false);  // Status flag
        $table->timestamps();                            // created_at & updated_at
    });
}

```

### Step 3: Execute Migration Locally

Run artisan migrate to build the table inside local phpMyAdmin:

```bash
php artisan migrate

```

---

## 4. Phase 3 — Git Version Control & Branch Management

Follow a feature-branching strategy to protect the stability of the `main` branch.

### Step 1: Initialize Git Repository

```bash
git init
git add .
git commit -m "Build initial application foundation and core schemas"
git branch -M main
git remote add origin [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
git push -u origin main

```

### Step 2: Feature Branch Lifecycle

Whenever starting a new feature request:

```bash
# 1. Create and switch to new branch
git checkout -b feature/client-reviews

# 2. Build model, controller, views, and migrations
php artisan make:controller ReviewController

# 3. Stage and commit incremental updates
git add .
git commit -m "Implement Review model, controller, and database schema"

# 4. Push feature branch to GitHub
git push origin feature/client-reviews

```

### Step 3: Pull Request & Merge

1. Open GitHub $\rightarrow$ Navigate to **Pull Requests**.
2. Create a PR from `feature/client-reviews` into `main`.
3. Review code changes and click **Merge Pull Request**.
4. Sync your local `main` branch:
```bash
git checkout main
git pull origin main

```



---

## 5. Phase 4 — Safe Hostinger Deployment (Zero Data Loss)

To deploy new features without deleting live customer data or causing downtime, follow these step-by-step procedures.

```
           LOCAL PC                         GITHUB                          HOSTINGER LIVE
  ┌────────────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐
  │ Merge feature to main  │ ───► │ Push code to main      │ ───► │ Pull main & run        │
  │ Test schema & views    │      │ Trigger auto/SSH pull  │      │ `migrate --force`      │
  └────────────────────────┘      └────────────────────────┘      └────────────────────────┘

```

### Deployment Execution (Via SSH or Hostinger Terminal)

Connect to Hostinger via SSH or open the hPanel Terminal, navigate to `public_html`, and execute:

```bash
# 1. Navigate to application directory
cd public_html

# 2. Pull latest merged code from GitHub
git pull origin main

# 3. Safely apply NEW database migrations
php artisan migrate --force

# 4. Clear and optimize production cache
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache

```

### Critical Deployment Commandments

1. **NEVER run `php artisan migrate:fresh` on Production:** This command drops all database tables and permanently wipes live user data.
2. **NEVER Overwrite Live `.env`:** Hostinger's `.env` contains live database credentials and production encryption keys.
3. **NEVER Overwrite `storage/` Directory:** Preserves live uploaded media, files, and server logs.
4. **Always Backup Live Database Before Releases:** Export a `.sql` snapshot in Hostinger phpMyAdmin prior to running major production updates.

---

## 6. Phase 5 — Full-Stack PM Governance & Release Checklist

Use this operational checklist to manage every feature request from client request to final release sign-off.

```
[ ] 1. Requirement & Scope Definition
    ├── Document functional requirement & acceptance criteria.
    └── Determine required schema/table additions.

[ ] 2. Local Schema & Feature Development
    ├── Create feature branch (git checkout -b feature/<name>).
    ├── Generate migrations and models (php artisan make:model <Name> -m).
    ├── Run local migrations (php artisan migrate).
    └── Build Controller, Routes, and Blade Views.

[ ] 3. Quality Assurance & Testing
    ├── Test form inputs, CSRF security (@csrf), and validations.
    └── Verify data persistence in local phpMyAdmin.

[ ] 4. Code Consolidation
    ├── Commit changes and push feature branch to GitHub.
    ├── Open and merge Pull Request into main branch.
    └── Verify repository build status.

[ ] 5. Production Release
    ├── Export safety backup of live DB in Hostinger phpMyAdmin.
    ├── Pull updated codebase via Hostinger SSH (git pull origin main).
    ├── Execute migration updates (php artisan migrate --force).
    └── Clear production cache (php artisan optimize:clear).

[ ] 6. Post-Deployment Audit
    ├── Verify live database table creation in Hostinger phpMyAdmin.
    └── Test feature functionality live on client domain.

```

```

```