# 🎯 Admin Dashboard Development Guide - لوحة تحكم المدير

## 📋 Overview / نظرة عامة

This guide contains all requirements, migrations, controllers, and API endpoints needed to build the complete Admin Dashboard for DELPO platform.

---

## 🏗️ Database Architecture

### ✅ Existing Tables - COPY THESE FIRST

Before creating new tables, run these existing table migrations to set up the base system:

#### Users Table Migration
```bash
php artisan make:migration create_users_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->enum('role_type', ['client', 'store_owner', 'service_provider', 'admin'])->default('client');
            $table->boolean('is_active')->default(true);
            $table->rememberToken();
            $table->timestamps();
            
            $table->index('role_type');
            $table->index('is_active');
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};
```

#### Categories Table Migration
```bash
php artisan make:migration create_categories_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('slug')->unique();
            $table->text('description')->nullable();
            $table->string('image_url')->nullable();
            $table->foreignId('parent_id')->nullable()->constrained('categories')->onDelete('cascade');
            $table->boolean('is_active')->default(true);
            $table->integer('display_order')->default(0);
            $table->timestamps();
            
            $table->index('is_active');
            $table->index('parent_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('categories');
    }
};
```

#### Stores Table Migration
```bash
php artisan make:migration create_stores_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('stores', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->string('area_name');
            $table->string('phone_number');
            $table->string('email_address');
            $table->foreignId('category_id')->constrained()->onDelete('restrict');
            $table->string('logo_image_url')->nullable();
            $table->string('background_image_url')->nullable();
            $table->json('store_photos_urls')->nullable();
            $table->json('staff_photos_urls')->nullable();
            $table->decimal('latitude_coordinate', 10, 8)->nullable();
            $table->decimal('longitude_coordinate', 11, 8)->nullable();
            $table->text('street_address')->nullable();
            $table->text('description')->nullable();
            $table->enum('operational_status', ['open', 'closed', 'maintenance', 'pending'])->default('pending');
            $table->boolean('is_active')->default(false);
            $table->timestamps();
            
            $table->index(['is_active', 'operational_status']);
            $table->index(['category_id', 'is_active']);
            $table->index(['latitude_coordinate', 'longitude_coordinate']);
            $table->index('area_name');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('stores');
    }
};
```

#### Products Table Migration
```bash
php artisan make:migration create_products_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->foreignId('store_id')->constrained('stores')->onDelete('cascade');
            $table->foreignId('category_id')->constrained('categories')->onDelete('cascade');
            $table->string('name');
            $table->text('description');
            $table->decimal('price', 10, 2);
            $table->decimal('sale_price', 10, 2)->nullable();
            $table->string('sku')->unique();
            $table->integer('stock_quantity')->default(0);
            $table->boolean('track_quantity')->default(true);
            $table->boolean('is_active')->default(true);
            $table->decimal('weight', 8, 2)->nullable();
            $table->string('dimensions')->nullable();
            $table->enum('status', ['active', 'inactive', 'out_of_stock'])->default('active');
            $table->boolean('is_featured')->default(false);
            $table->json('attributes')->nullable();
            $table->timestamps();
            
            $table->index(['store_id', 'status']);
            $table->index(['category_id', 'status']);
            $table->index(['status', 'is_featured']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

#### Product Images Table Migration
```bash
php artisan make:migration create_product_images_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('product_images', function (Blueprint $table) {
            $table->id();
            $table->foreignId('product_id')->constrained('products')->onDelete('cascade');
            $table->string('image_url');
            $table->boolean('is_primary')->default(false);
            $table->integer('display_order')->default(0);
            $table->timestamps();
            
            $table->index('product_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('product_images');
    }
};
```

#### Orders Table Migration
```bash
php artisan make:migration create_orders_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained('users')->onDelete('cascade');
            $table->foreignId('store_id')->constrained('stores')->onDelete('cascade');
            $table->foreignId('delivery_person_id')->nullable()->constrained('users')->nullOnDelete();
            $table->string('order_number')->unique();
            $table->enum('status', ['pending', 'confirmed', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded'])->default('pending');
            $table->decimal('total_amount', 10, 2);
            $table->decimal('delivery_fee', 10, 2)->default(0);
            $table->json('shipping_address');
            $table->json('billing_address');
            $table->text('notes')->nullable();
            $table->string('tracking_number')->nullable();
            $table->timestamp('estimated_delivery_time')->nullable();
            $table->timestamp('delivered_at')->nullable();
            $table->timestamp('order_date')->default(now());
            $table->decimal('delivery_latitude', 10, 8)->nullable();
            $table->decimal('delivery_longitude', 11, 8)->nullable();
            $table->timestamps();
            
            $table->index(['user_id', 'status']);
            $table->index(['store_id', 'status']);
            $table->index(['delivery_person_id', 'status']);
            $table->index('order_date');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('orders');
    }
};
```

#### Order Items Table Migration
```bash
php artisan make:migration create_order_items_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('order_items', function (Blueprint $table) {
            $table->id();
            $table->foreignId('order_id')->constrained('orders')->onDelete('cascade');
            $table->foreignId('product_id')->constrained('products')->onDelete('restrict');
            $table->string('product_name');
            $table->decimal('price', 10, 2);
            $table->integer('quantity');
            $table->decimal('total', 10, 2);
            $table->timestamps();
            
            $table->index('order_id');
            $table->index('product_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('order_items');
    }
};
```

#### Notifications Table Migration
```bash
php artisan make:migration create_notifications_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('notifications', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained('users')->onDelete('cascade');
            $table->string('type');
            $table->string('title');
            $table->text('message');
            $table->json('data')->nullable();
            $table->boolean('is_read')->default(false);
            $table->timestamp('read_at')->nullable();
            $table->timestamps();
            
            $table->index(['user_id', 'is_read']);
            $table->index('created_at');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('notifications');
    }
};
```

#### Delivery Persons Table Migration
```bash
php artisan make:migration create_delivery_persons_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('delivery_persons', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained('users')->onDelete('cascade');
            $table->string('vehicle_type')->nullable();
            $table->string('vehicle_number')->nullable();
            $table->string('license_number')->nullable();
            $table->boolean('is_available')->default(true);
            $table->decimal('current_latitude', 10, 8)->nullable();
            $table->decimal('current_longitude', 11, 8)->nullable();
            $table->timestamps();
            
            $table->index('user_id');
            $table->index('is_available');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('delivery_persons');
    }
};
```

#### User Points Table Migration
```bash
php artisan make:migration create_user_points_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('user_points', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->unique()->constrained('users')->onDelete('cascade');
            $table->integer('total_points')->default(0);
            $table->integer('available_points')->default(0);
            $table->integer('redeemed_points')->default(0);
            $table->timestamps();
            
            $table->index('user_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('user_points');
    }
};
```

#### Point Transactions Table Migration
```bash
php artisan make:migration create_point_transactions_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('point_transactions', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained('users')->onDelete('cascade');
            $table->enum('type', ['earned', 'redeemed', 'expired', 'adjusted']);
            $table->integer('points');
            $table->string('reason');
            $table->foreignId('order_id')->nullable()->constrained('orders')->nullOnDelete();
            $table->text('notes')->nullable();
            $table->timestamps();
            
            $table->index(['user_id', 'type']);
            $table->index('created_at');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('point_transactions');
    }
};
```

**Run these migrations first:**
```bash
php artisan migrate
```

---

### 🆕 New Migrations Required

After setting up existing tables, create these 6 new tables for dashboard features:

#### 1. Settings & Configuration Table
```bash
php artisan make:migration create_settings_table
```

**File:** `2025_11_27_000001_create_settings_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('settings', function (Blueprint $table) {
            $table->id();
            $table->string('key')->unique(); // e.g., 'delivery_fee', 'tax_rate'
            $table->text('value')->nullable();
            $table->string('type')->default('string'); // string, integer, boolean, json
            $table->string('group')->default('general'); // general, payment, delivery, etc.
            $table->text('description')->nullable();
            $table->timestamps();
            
            $table->index('key');
            $table->index('group');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('settings');
    }
};
```

**Column Explanations:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `id` | BIGINT UNSIGNED | Primary key, auto-incrementing unique identifier | 1, 2, 3... |
| `key` | VARCHAR(255) UNIQUE | Unique identifier for the setting. Used to retrieve specific settings in code | `'tax_rate'`, `'delivery_fee'`, `'app_name'` |
| `value` | TEXT NULLABLE | The actual value of the setting. Can be string, number, JSON, etc. Stored as text | `'15'`, `'DELPO'`, `'{"min":10,"max":100}'` |
| `type` | VARCHAR(255) | Data type of the value to help with proper casting when retrieving | `'string'`, `'integer'`, `'boolean'`, `'json'`, `'float'` |
| `group` | VARCHAR(255) | Category/group for organizing settings in admin panel | `'general'`, `'payment'`, `'delivery'`, `'notification'` |
| `description` | TEXT NULLABLE | Human-readable explanation of what this setting controls | `'Percentage of tax applied to orders'` |
| `created_at` | TIMESTAMP | When this setting was first created | `2025-11-27 10:30:00` |
| `updated_at` | TIMESTAMP | When this setting was last modified | `2025-11-27 15:45:00` |

**Indexes:**
- `index('key')` - Speeds up lookups when fetching settings by key (most common operation)
- `index('group')` - Speeds up queries when displaying settings grouped by category

#### 2. Admin Activity Logs Table
```bash
php artisan make:migration create_admin_activity_logs_table
```

**File:** `2025_11_27_000002_create_admin_activity_logs_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('admin_activity_logs', function (Blueprint $table) {
            $table->id();
            $table->foreignId('admin_user_id')->constrained('users')->onDelete('cascade');
            $table->string('action'); // created, updated, deleted, approved, rejected
            $table->string('model_type'); // User, Store, Product, Order
            $table->unsignedBigInteger('model_id')->nullable();
            $table->text('description')->nullable();
            $table->json('old_values')->nullable();
            $table->json('new_values')->nullable();
            $table->string('ip_address')->nullable();
            $table->timestamps();
            
            $table->index(['admin_user_id', 'created_at']);
            $table->index(['model_type', 'model_id']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('admin_activity_logs');
    }
};
```

**Column Explanations:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `id` | BIGINT UNSIGNED | Primary key, auto-incrementing unique identifier | 1, 2, 3... |
| `admin_user_id` | BIGINT UNSIGNED (FK) | ID of the admin user who performed this action. References `users.id` | `5` (admin's user ID) |
| `action` | VARCHAR(255) | Type of action performed by the admin | `'created'`, `'updated'`, `'deleted'`, `'approved'`, `'rejected'`, `'activated'`, `'deactivated'` |
| `model_type` | VARCHAR(255) | Type of entity/model that was affected (for polymorphic tracking) | `'User'`, `'Store'`, `'Product'`, `'Order'`, `'Category'` |
| `model_id` | BIGINT UNSIGNED NULLABLE | ID of the specific record that was affected. Nullable if action affects multiple records | `123` (the ID of the user/store/product that was modified) |
| `description` | TEXT NULLABLE | Human-readable description of what happened | `'Admin approved store "ABC Electronics"'`, `'Deleted user account for john@example.com'` |
| `old_values` | JSON NULLABLE | The previous values before the change (for update actions). Stored as JSON object | `{"status":"pending","is_active":false}` |
| `new_values` | JSON NULLABLE | The new values after the change (for update actions). Stored as JSON object | `{"status":"approved","is_active":true}` |
| `ip_address` | VARCHAR(255) NULLABLE | IP address from which the admin performed this action (for security auditing) | `'192.168.1.100'`, `'2001:0db8:85a3::8a2e:0370:7334'` |
| `created_at` | TIMESTAMP | When this action was performed | `2025-11-27 10:30:00` |
| `updated_at` | TIMESTAMP | Not typically used in logs, but included by Laravel convention | `2025-11-27 10:30:00` |

**Indexes:**
- `index(['admin_user_id', 'created_at'])` - Efficiently find all actions by a specific admin, sorted by time
- `index(['model_type', 'model_id'])` - Efficiently find all actions performed on a specific record

**Foreign Key:**
- `admin_user_id` references `users.id` with `onDelete('cascade')` - If admin user is deleted, all their logs are also deleted

#### 3. System Reports Table
```bash
php artisan make:migration create_system_reports_table
```

**File:** `2025_11_27_000003_create_system_reports_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('system_reports', function (Blueprint $table) {
            $table->id();
            $table->foreignId('generated_by')->constrained('users')->onDelete('cascade');
            $table->string('report_type'); // sales, users, stores, products
            $table->string('period'); // daily, weekly, monthly, yearly
            $table->date('start_date');
            $table->date('end_date');
            $table->json('data'); // Store report data as JSON
            $table->string('status')->default('pending'); // pending, completed, failed
            $table->text('file_path')->nullable(); // Path to exported file
            $table->timestamps();
            
            $table->index(['report_type', 'period']);
            $table->index('created_at');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('system_reports');
    }
};
```

**Column Explanations:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `id` | BIGINT UNSIGNED | Primary key, auto-incrementing unique identifier | 1, 2, 3... |
| `generated_by` | BIGINT UNSIGNED (FK) | ID of the admin user who requested this report. References `users.id` | `5` (admin's user ID) |
| `report_type` | VARCHAR(255) | Category of report being generated | `'sales'`, `'users'`, `'stores'`, `'products'`, `'revenue'`, `'orders'` |
| `period` | VARCHAR(255) | Time period granularity for the report | `'daily'`, `'weekly'`, `'monthly'`, `'yearly'`, `'custom'` |
| `start_date` | DATE | Beginning date of the reporting period | `'2025-11-01'` |
| `end_date` | DATE | Ending date of the reporting period (inclusive) | `'2025-11-30'` |
| `data` | JSON | Complete report data stored as JSON (charts, tables, statistics) | `{"total_sales":15000,"orders":120,"avg_order":125}` |
| `status` | VARCHAR(255) | Current state of report generation process | `'pending'` (queued), `'processing'` (generating), `'completed'` (ready), `'failed'` (error occurred) |
| `file_path` | TEXT NULLABLE | Server path or URL to exported file (CSV/Excel/PDF) if generated | `'storage/reports/sales-2025-11.xlsx'`, `null` if not exported |
| `created_at` | TIMESTAMP | When the report generation was requested | `2025-11-27 10:30:00` |
| `updated_at` | TIMESTAMP | When the report was last updated (useful for tracking completion time) | `2025-11-27 10:35:00` |

**Indexes:**
- `index(['report_type', 'period'])` - Quickly find reports by type and period for caching/reuse
- `index('created_at')` - Efficiently list recent reports, useful for cleanup of old reports

**Foreign Key:**
- `generated_by` references `users.id` with `onDelete('cascade')` - If user is deleted, their reports are also deleted

**Usage Example:**
```php
// Store a sales report
SystemReport::create([
    'generated_by' => auth()->id(),
    'report_type' => 'sales',
    'period' => 'monthly',
    'start_date' => '2025-11-01',
    'end_date' => '2025-11-30',
    'data' => [
        'total_revenue' => 150000,
        'total_orders' => 1200,
        'daily_breakdown' => [...]
    ],
    'status' => 'completed'
]);
```

#### 4. Promotions & Banners Table
```bash
php artisan make:migration create_promotions_and_banners_tables
```

**File:** `2025_11_27_000004_create_promotions_and_banners_tables.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        // Promotions table for discounts and offers
        Schema::create('promotions', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->string('type'); // percentage, fixed_amount, buy_x_get_y
            $table->decimal('discount_value', 10, 2)->nullable();
            $table->decimal('min_purchase_amount', 10, 2)->nullable();
            $table->integer('max_uses')->nullable();
            $table->integer('uses_count')->default(0);
            $table->boolean('is_active')->default(true);
            $table->date('start_date');
            $table->date('end_date');
            $table->json('applicable_to')->nullable(); // products, categories, stores
            $table->timestamps();
            
            $table->index(['is_active', 'start_date', 'end_date']);
        });

        // Banners table for homepage/promotional banners
        Schema::create('banners', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('description')->nullable();
            $table->string('image_url');
            $table->string('link_type'); // product, category, store, external, none
            $table->string('link_value')->nullable();
            $table->integer('display_order')->default(0);
            $table->boolean('is_active')->default(true);
            $table->date('start_date');
            $table->date('end_date')->nullable();
            $table->timestamps();
            
            $table->index(['is_active', 'display_order']);
        });

        // Discount codes table
        Schema::create('discount_codes', function (Blueprint $table) {
            $table->id();
            $table->string('code')->unique();
            $table->foreignId('promotion_id')->nullable()->constrained()->nullOnDelete();
            $table->string('type'); // percentage, fixed_amount
            $table->decimal('discount_value', 10, 2);
            $table->decimal('min_purchase_amount', 10, 2)->nullable();
            $table->integer('max_uses')->nullable();
            $table->integer('uses_count')->default(0);
            $table->boolean('is_active')->default(true);
            $table->date('expires_at')->nullable();
            $table->timestamps();
            
            $table->index('code');
            $table->index(['is_active', 'expires_at']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('discount_codes');
        Schema::dropIfExists('banners');
        Schema::dropIfExists('promotions');
    }
};
```

**Column Explanations - `promotions` table:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `id` | BIGINT UNSIGNED | Primary key | 1, 2, 3... |
| `name` | VARCHAR(255) | Promotion name | `'Summer Sale'`, `'Buy 2 Get 1'` |
| `description` | TEXT NULLABLE | Detailed promotion description | `'Get 50% off on all electronics'` |
| `type` | VARCHAR(255) | Type of discount | `'percentage'`, `'fixed_amount'`, `'buy_x_get_y'` |
| `discount_value` | DECIMAL(10,2) | Discount amount or percentage | `20.00` (20% or 20 SAR) |
| `min_purchase_amount` | DECIMAL(10,2) | Minimum purchase to qualify | `100.00` (100 SAR minimum) |
| `max_uses` | INTEGER NULLABLE | Maximum number of times promotion can be used | `100`, `null` (unlimited) |
| `uses_count` | INTEGER | Current usage count | `45` |
| `is_active` | BOOLEAN | Whether promotion is currently active | `true`, `false` |
| `start_date` | DATE | When promotion becomes active | `'2025-12-01'` |
| `end_date` | DATE | When promotion expires | `'2025-12-31'` |
| `applicable_to` | JSON | What the promotion applies to | `{"products":[1,2,3],"categories":[5]}` |
| `created_at` | TIMESTAMP | When promotion was created | `2025-11-27 10:00:00` |
| `updated_at` | TIMESTAMP | Last update | `2025-11-27 10:00:00` |

**Column Explanations - `banners` table:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `id` | BIGINT UNSIGNED | Primary key | 1, 2, 3... |
| `title` | VARCHAR(255) | Banner title | `'New Products Available'` |
| `description` | TEXT NULLABLE | Banner description | `'Check out our latest collection'` |
| `image_url` | VARCHAR(255) | Path to banner image | `'storage/banners/summer-sale.jpg'` |
| `link_type` | VARCHAR(255) | What the banner links to | `'product'`, `'category'`, `'store'`, `'external'`, `'none'` |
| `link_value` | VARCHAR(255) | ID or URL of link target | `'123'` (product ID), `'https://example.com'` |
| `display_order` | INTEGER | Sort order for display | `1`, `2`, `3` (lower shows first) |
| `is_active` | BOOLEAN | Whether banner is shown | `true`, `false` |
| `start_date` | DATE | When banner becomes visible | `'2025-12-01'` |
| `end_date` | DATE NULLABLE | When banner expires (null = no expiry) | `'2025-12-31'`, `null` |

**Column Explanations - `discount_codes` table:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `id` | BIGINT UNSIGNED | Primary key | 1, 2, 3... |
| `code` | VARCHAR(255) UNIQUE | Coupon code | `'SUMMER20'`, `'WELCOME10'` |
| `promotion_id` | BIGINT UNSIGNED FK | Link to promotion (optional) | `5`, `null` |
| `type` | VARCHAR(255) | Discount type | `'percentage'`, `'fixed_amount'` |
| `discount_value` | DECIMAL(10,2) | Discount amount | `20.00` |
| `min_purchase_amount` | DECIMAL(10,2) | Minimum order value | `50.00` |
| `max_uses` | INTEGER NULLABLE | Usage limit | `100`, `null` (unlimited) |
| `uses_count` | INTEGER | Times used | `25` |
| `is_active` | BOOLEAN | Active status | `true`, `false` |
| `expires_at` | DATE NULLABLE | Expiration date | `'2025-12-31'`, `null` |

#### 5. Campaigns Table
```bash
php artisan make:migration create_campaigns_table
```

**File:** `2025_11_27_000005_create_campaigns_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('campaigns', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->string('type'); // email, sms, push_notification, in_app
            $table->string('status')->default('draft'); // draft, scheduled, active, completed, paused
            $table->json('target_audience'); // user filters: role, location, purchase_history
            $table->text('message_content');
            $table->string('image_url')->nullable();
            $table->string('action_type')->nullable(); // link, discount_code, product
            $table->string('action_value')->nullable();
            $table->timestamp('scheduled_at')->nullable();
            $table->timestamp('sent_at')->nullable();
            $table->integer('target_count')->default(0);
            $table->integer('sent_count')->default(0);
            $table->integer('opened_count')->default(0);
            $table->integer('clicked_count')->default(0);
            $table->foreignId('created_by')->constrained('users')->onDelete('cascade');
            $table->timestamps();
            
            $table->index(['status', 'scheduled_at']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('campaigns');
    }
};
```

**Column Explanations - `campaigns` table:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `id` | BIGINT UNSIGNED | Primary key | 1, 2, 3... |
| `name` | VARCHAR(255) | Campaign name | `'Black Friday Sale'`, `'New User Welcome'` |
| `description` | TEXT NULLABLE | Campaign description | `'Promote our biggest sale of the year'` |
| `type` | VARCHAR(255) | Communication channel | `'email'`, `'sms'`, `'push_notification'`, `'in_app'` |
| `status` | VARCHAR(255) | Campaign state | `'draft'`, `'scheduled'`, `'active'`, `'completed'`, `'paused'` |
| `target_audience` | JSON | Who receives this campaign | `{"role":"client","location":"Riyadh"}` |
| `message_content` | TEXT | The message to send | `'Get 50% off today only!'` |
| `image_url` | VARCHAR(255) | Campaign image | `'storage/campaigns/banner.jpg'` |
| `action_type` | VARCHAR(255) | What happens when clicked | `'link'`, `'discount_code'`, `'product'` |
| `action_value` | VARCHAR(255) | Link/code/product ID | `'SAVE50'`, `'https://...'`, `'123'` |
| `scheduled_at` | TIMESTAMP | When to send | `'2025-12-25 09:00:00'` |
| `sent_at` | TIMESTAMP | When actually sent | `'2025-12-25 09:05:23'` |
| `target_count` | INTEGER | How many users targeted | `5000` |
| `sent_count` | INTEGER | Successfully sent | `4850` |
| `opened_count` | INTEGER | How many opened | `2400` |
| `clicked_count` | INTEGER | How many clicked | `750` |
| `created_by` | BIGINT UNSIGNED FK | Admin who created it | `5` |

#### 6. Delivery Services Table
```bash
php artisan make:migration create_delivery_services_table
```

**File:** `2025_11_27_000006_create_delivery_services_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('delivery_services', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->string('type'); // standard, express, scheduled, pickup
            $table->decimal('base_fee', 10, 2);
            $table->decimal('per_km_fee', 10, 2)->nullable();
            $table->integer('estimated_time_minutes');
            $table->boolean('is_active')->default(true);
            $table->json('availability')->nullable(); // days, hours
            $table->timestamps();
            
            $table->index(['type', 'is_active']);
        });

        // Add delivery_service_id to orders table
        Schema::table('orders', function (Blueprint $table) {
            $table->foreignId('delivery_service_id')->nullable()->after('delivery_person_id')
                  ->constrained('delivery_services')->nullOnDelete();
        });
    }

    public function down(): void
    {
        Schema::table('orders', function (Blueprint $table) {
            $table->dropForeign(['delivery_service_id']);
            $table->dropColumn('delivery_service_id');
        });
        
        Schema::dropIfExists('delivery_services');
    }
};
```

**Column Explanations - `delivery_services` table:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `id` | BIGINT UNSIGNED | Primary key | 1, 2, 3... |
| `name` | VARCHAR(255) | Service name | `'Standard Delivery'`, `'Express 1-Hour'` |
| `description` | TEXT NULLABLE | Service details | `'Delivery within 2-3 days'` |
| `type` | VARCHAR(255) | Service type | `'standard'`, `'express'`, `'scheduled'`, `'pickup'` |
| `base_fee` | DECIMAL(10,2) | Base delivery cost | `15.00` (15 SAR) |
| `per_km_fee` | DECIMAL(10,2) | Additional cost per km | `2.00` (2 SAR/km), `null` if flat rate |
| `estimated_time_minutes` | INTEGER | Estimated delivery time | `120` (2 hours), `1440` (24 hours) |
| `is_active` | BOOLEAN | Service available | `true`, `false` |
| `availability` | JSON | When service is available | `{"days":[1,2,3,4,5],"hours":"9:00-18:00"}` |

**Note:** 6 new migrations needed for all dashboard features.

---

## 📁 Models Required

### Create New Models:

#### 1. Setting Model
```bash
php artisan make:model Setting
```

**File:** `app/Models/Setting.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Setting extends Model
{
    protected $fillable = [
        'key',
        'value',
        'type',
        'group',
        'description'
    ];

    protected $casts = [
        'value' => 'json',
    ];

    /**
     * Get setting value by key
     */
    public static function get($key, $default = null)
    {
        $setting = self::where('key', $key)->first();
        return $setting ? $setting->value : $default;
    }

    /**
     * Set setting value by key
     */
    public static function set($key, $value, $type = 'string', $group = 'general')
    {
        return self::updateOrCreate(
            ['key' => $key],
            ['value' => $value, 'type' => $type, 'group' => $group]
        );
    }
}
```

#### 2. AdminActivityLog Model
```bash
php artisan make:model AdminActivityLog
```

**File:** `app/Models/AdminActivityLog.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class AdminActivityLog extends Model
{
    protected $fillable = [
        'admin_user_id',
        'action',
        'model_type',
        'model_id',
        'description',
        'old_values',
        'new_values',
        'ip_address'
    ];

    protected $casts = [
        'old_values' => 'array',
        'new_values' => 'array',
    ];

    /**
     * Admin who performed the action
     */
    public function admin()
    {
        return $this->belongsTo(User::class, 'admin_user_id');
    }

    /**
     * Log an admin action
     */
    public static function logActivity($adminId, $action, $modelType, $modelId, $description, $oldValues = null, $newValues = null)
    {
        return self::create([
            'admin_user_id' => $adminId,
            'action' => $action,
            'model_type' => $modelType,
            'model_id' => $modelId,
            'description' => $description,
            'old_values' => $oldValues,
            'new_values' => $newValues,
            'ip_address' => request()->ip()
        ]);
    }
}
```

#### 3. SystemReport Model
```bash
php artisan make:model SystemReport
```

**File:** `app/Models/SystemReport.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class SystemReport extends Model
{
    protected $fillable = [
        'generated_by',
        'report_type',
        'period',
        'start_date',
        'end_date',
        'data',
        'status',
        'file_path'
    ];

    protected $casts = [
        'data' => 'array',
        'start_date' => 'date',
        'end_date' => 'date',
    ];

    /**
     * User who generated the report
     */
    public function generatedBy()
    {
        return $this->belongsTo(User::class, 'generated_by');
    }
}
```

#### 4. Promotion Model
```bash
php artisan make:model Promotion
```

**File:** `app/Models/Promotion.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Carbon\Carbon;

class Promotion extends Model
{
    protected $fillable = [
        'name',
        'description',
        'type',
        'discount_value',
        'min_purchase_amount',
        'max_uses',
        'uses_count',
        'is_active',
        'start_date',
        'end_date',
        'applicable_to'
    ];

    protected $casts = [
        'discount_value' => 'decimal:2',
        'min_purchase_amount' => 'decimal:2',
        'is_active' => 'boolean',
        'start_date' => 'date',
        'end_date' => 'date',
        'applicable_to' => 'array',
    ];

    /**
     * Check if promotion is currently valid
     */
    public function isValid()
    {
        $now = Carbon::now();
        return $this->is_active 
            && $this->start_date <= $now 
            && $this->end_date >= $now
            && ($this->max_uses === null || $this->uses_count < $this->max_uses);
    }

    /**
     * Calculate discount amount
     */
    public function calculateDiscount($amount)
    {
        if ($this->type === 'percentage') {
            return $amount * ($this->discount_value / 100);
        }
        return $this->discount_value;
    }
}
```

#### 5. Banner Model
```bash
php artisan make:model Banner
```

**File:** `app/Models/Banner.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Carbon\Carbon;

class Banner extends Model
{
    protected $fillable = [
        'title',
        'description',
        'image_url',
        'link_type',
        'link_value',
        'display_order',
        'is_active',
        'start_date',
        'end_date'
    ];

    protected $casts = [
        'is_active' => 'boolean',
        'start_date' => 'date',
        'end_date' => 'date',
        'display_order' => 'integer',
    ];

    /**
     * Check if banner should be displayed
     */
    public function isVisible()
    {
        $now = Carbon::now();
        return $this->is_active 
            && $this->start_date <= $now 
            && ($this->end_date === null || $this->end_date >= $now);
    }

    /**
     * Get active banners ordered by display order
     */
    public static function getActive()
    {
        return self::where('is_active', true)
            ->where('start_date', '<=', Carbon::now())
            ->where(function($query) {
                $query->whereNull('end_date')
                      ->orWhere('end_date', '>=', Carbon::now());
            })
            ->orderBy('display_order')
            ->get();
    }
}
```

#### 6. DiscountCode Model
```bash
php artisan make:model DiscountCode
```

**File:** `app/Models/DiscountCode.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Carbon\Carbon;

class DiscountCode extends Model
{
    protected $fillable = [
        'code',
        'promotion_id',
        'type',
        'discount_value',
        'min_purchase_amount',
        'max_uses',
        'uses_count',
        'is_active',
        'expires_at'
    ];

    protected $casts = [
        'discount_value' => 'decimal:2',
        'min_purchase_amount' => 'decimal:2',
        'is_active' => 'boolean',
        'expires_at' => 'date',
    ];

    /**
     * Promotion this code belongs to
     */
    public function promotion()
    {
        return $this->belongsTo(Promotion::class);
    }

    /**
     * Check if code is valid
     */
    public function isValid($orderAmount = 0)
    {
        $now = Carbon::now();
        return $this->is_active
            && ($this->expires_at === null || $this->expires_at >= $now)
            && ($this->max_uses === null || $this->uses_count < $this->max_uses)
            && ($this->min_purchase_amount === null || $orderAmount >= $this->min_purchase_amount);
    }

    /**
     * Apply discount code
     */
    public function apply($amount)
    {
        if ($this->type === 'percentage') {
            return $amount * ($this->discount_value / 100);
        }
        return min($this->discount_value, $amount);
    }

    /**
     * Increment usage count
     */
    public function incrementUsage()
    {
        $this->increment('uses_count');
    }
}
```

#### 7. Campaign Model
```bash
php artisan make:model Campaign
```

**File:** `app/Models/Campaign.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Campaign extends Model
{
    protected $fillable = [
        'name',
        'description',
        'type',
        'status',
        'target_audience',
        'message_content',
        'image_url',
        'action_type',
        'action_value',
        'scheduled_at',
        'sent_at',
        'target_count',
        'sent_count',
        'opened_count',
        'clicked_count',
        'created_by'
    ];

    protected $casts = [
        'target_audience' => 'array',
        'scheduled_at' => 'datetime',
        'sent_at' => 'datetime',
    ];

    /**
     * User who created the campaign
     */
    public function creator()
    {
        return $this->belongsTo(User::class, 'created_by');
    }

    /**
     * Get campaign performance metrics
     */
    public function getMetrics()
    {
        return [
            'open_rate' => $this->sent_count > 0 ? ($this->opened_count / $this->sent_count) * 100 : 0,
            'click_rate' => $this->opened_count > 0 ? ($this->clicked_count / $this->opened_count) * 100 : 0,
            'delivery_rate' => $this->target_count > 0 ? ($this->sent_count / $this->target_count) * 100 : 0,
        ];
    }
}
```

#### 8. DeliveryService Model
```bash
php artisan make:model DeliveryService
```

**File:** `app/Models/DeliveryService.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class DeliveryService extends Model
{
    protected $fillable = [
        'name',
        'description',
        'type',
        'base_fee',
        'per_km_fee',
        'estimated_time_minutes',
        'is_active',
        'availability'
    ];

    protected $casts = [
        'base_fee' => 'decimal:2',
        'per_km_fee' => 'decimal:2',
        'is_active' => 'boolean',
        'availability' => 'array',
    ];

    /**
     * Orders using this service
     */
    public function orders()
    {
        return $this->hasMany(Order::class);
    }

    /**
     * Calculate delivery fee
     */
    public function calculateFee($distance_km = 0)
    {
        $fee = $this->base_fee;
        if ($this->per_km_fee && $distance_km > 0) {
            $fee += ($distance_km * $this->per_km_fee);
        }
        return $fee;
    }

    /**
     * Get active delivery services
     */
    public static function getActive()
    {
        return self::where('is_active', true)->get();
    }
}
```

---

## 🎮 Controllers to Create/Extend

### ✅ 1. AdminController (Already Exists - Needs Enhancement)

**Location:** `app/Http/Controllers/AdminController.php`

The AdminController already exists with basic functionality. You need to ADD these methods:

```php
/**
 * Get dashboard statistics with advanced metrics
 */
public function getDashboardStats()
{
    // Revenue trends
    // User growth graphs
    // Order completion rates
    // Top performing stores
    // Low stock alerts
}

/**
 * Get all system settings
 */
public function getSettings()
{
    return Setting::all()->groupBy('group');
}

/**
 * Update system settings
 */
public function updateSettings(Request $request)
{
    // Update multiple settings at once
}

/**
 * Export data (users, orders, products, stores)
 */
public function exportData(Request $request)
{
    // Export to CSV/Excel
}

/**
 * Get system logs and activities
 */
public function getActivityLogs(Request $request)
{
    return AdminActivityLog::with('admin')
        ->latest()
        ->paginate(50);
}

/**
 * Get system alerts (low stock, pending orders, etc.)
 */
public function getSystemAlerts()
{
    return response()->json([
        'success' => true,
        'data' => [
            'low_stock_products' => Product::where('stock_quantity', '<', 10)->count(),
            'out_of_stock_products' => Product::where('stock_quantity', 0)->count(),
            'pending_orders' => Order::where('status', 'pending')->count(),
            'pending_stores' => Stores::where('operational_status', 'pending')->count(),
        ]
    ]);
}
```

### 🆕 2. DashboardController (NEW)
```bash
php artisan make:controller DashboardController
```

**File:** `app/Http/Controllers/DashboardController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use App\Models\Order;
use App\Models\Product;
use App\Models\Stores;
use App\Models\Category;
use Illuminate\Http\Request;
use Carbon\Carbon;
use Illuminate\Support\Facades\DB;

class DashboardController extends Controller
{
    /**
     * Get comprehensive dashboard statistics
     */
    public function getStats(Request $request)
    {
        $period = $request->get('period', '7days'); // 7days, 30days, 90days, year

        return response()->json([
            'success' => true,
            'data' => [
                'overview' => $this->getOverviewStats(),
                'charts' => [
                    'revenue' => $this->getRevenueChart($period),
                    'orders' => $this->getOrdersChart($period),
                    'users' => $this->getUsersChart($period),
                    'products' => $this->getProductsChart($period),
                ],
                'top_performers' => [
                    'stores' => $this->getTopStores(10),
                    'products' => $this->getTopProducts(10),
                    'categories' => $this->getTopCategories(10),
                ],
                'recent_activities' => $this->getRecentActivities(),
                'alerts' => $this->getSystemAlerts(),
            ]
        ]);
    }

    /**
     * Overview statistics (cards at top)
     */
    private function getOverviewStats()
    {
        return [
            'total_revenue' => Order::where('status', 'delivered')->sum('total_amount'),
            'total_orders' => Order::count(),
            'total_users' => User::count(),
            'total_stores' => Stores::count(),
            'total_products' => Product::count(),
            'pending_orders' => Order::where('status', 'pending')->count(),
            'active_stores' => Stores::where('is_active', true)->count(),
            'new_users_today' => User::whereDate('created_at', today())->count(),
        ];
    }

    /**
     * Revenue chart data
     */
    private function getRevenueChart($period)
    {
        $dates = $this->getDateRange($period);
        
        $data = Order::where('status', 'delivered')
            ->whereBetween('created_at', [$dates['start'], $dates['end']])
            ->selectRaw('DATE(created_at) as date, SUM(total_amount) as total')
            ->groupBy('date')
            ->orderBy('date')
            ->get();

        return [
            'labels' => $data->pluck('date')->toArray(),
            'values' => $data->pluck('total')->toArray(),
        ];
    }

    /**
     * Orders chart data
     */
    private function getOrdersChart($period)
    {
        $dates = $this->getDateRange($period);
        
        $data = Order::whereBetween('created_at', [$dates['start'], $dates['end']])
            ->selectRaw('DATE(created_at) as date, status, COUNT(*) as count')
            ->groupBy('date', 'status')
            ->orderBy('date')
            ->get();

        return [
            'labels' => $data->pluck('date')->unique()->values()->toArray(),
            'datasets' => $this->formatOrderStatusDatasets($data),
        ];
    }

    /**
     * Users chart data (growth over time)
     */
    private function getUsersChart($period)
    {
        $dates = $this->getDateRange($period);
        
        $data = User::whereBetween('created_at', [$dates['start'], $dates['end']])
            ->selectRaw('DATE(created_at) as date, role_type, COUNT(*) as count')
            ->groupBy('date', 'role_type')
            ->orderBy('date')
            ->get();

        return [
            'labels' => $data->pluck('date')->unique()->values()->toArray(),
            'datasets' => $this->formatUserRoleDatasets($data),
        ];
    }

    /**
     * Products statistics
     */
    private function getProductsChart($period)
    {
        return [
            'by_category' => Product::join('categories', 'products.category_id', '=', 'categories.id')
                ->selectRaw('categories.name, COUNT(*) as count')
                ->groupBy('categories.name')
                ->get(),
            'by_status' => [
                'active' => Product::where('is_active', true)->count(),
                'inactive' => Product::where('is_active', false)->count(),
                'low_stock' => Product::where('stock_quantity', '<', 10)->where('stock_quantity', '>', 0)->count(),
                'out_of_stock' => Product::where('stock_quantity', 0)->count(),
            ]
        ];
    }

    /**
     * Top performing stores
     */
    private function getTopStores($limit = 10)
    {
        return Stores::withCount('orders')
            ->with('user:id,name,email')
            ->orderBy('orders_count', 'desc')
            ->limit($limit)
            ->get();
    }

    /**
     * Top selling products
     */
    private function getTopProducts($limit = 10)
    {
        return Product::select('products.*')
            ->join('order_items', 'products.id', '=', 'order_items.product_id')
            ->selectRaw('products.*, SUM(order_items.quantity) as total_sold')
            ->groupBy('products.id')
            ->orderBy('total_sold', 'desc')
            ->limit($limit)
            ->get();
    }

    /**
     * Top categories by sales
     */
    private function getTopCategories($limit = 10)
    {
        return Category::select('categories.*')
            ->join('products', 'categories.id', '=', 'products.category_id')
            ->join('order_items', 'products.id', '=', 'order_items.product_id')
            ->selectRaw('categories.*, COUNT(order_items.id) as total_sales')
            ->groupBy('categories.id')
            ->orderBy('total_sales', 'desc')
            ->limit($limit)
            ->get();
    }

    /**
     * Recent system activities
     */
    private function getRecentActivities()
    {
        return [
            'recent_orders' => Order::with('user:id,name')->latest()->limit(5)->get(),
            'recent_users' => User::latest()->limit(5)->get(['id', 'name', 'email', 'role_type', 'created_at']),
            'recent_stores' => Stores::with('user:id,name')->latest()->limit(5)->get(),
            'recent_products' => Product::latest()->limit(5)->get(['id', 'name', 'price', 'stock_quantity', 'created_at']),
        ];
    }

    /**
     * System alerts and warnings
     */
    private function getSystemAlerts()
    {
        return [
            'low_stock_products' => Product::where('stock_quantity', '<', 10)
                ->where('stock_quantity', '>', 0)
                ->count(),
            'out_of_stock_products' => Product::where('stock_quantity', 0)->count(),
            'pending_orders' => Order::where('status', 'pending')->count(),
            'pending_stores' => Stores::where('operational_status', 'pending')->count(),
        ];
    }

    /**
     * Helper: Get date range based on period
     */
    private function getDateRange($period)
    {
        $end = Carbon::now();
        
        switch ($period) {
            case '7days':
                $start = $end->copy()->subDays(7);
                break;
            case '30days':
                $start = $end->copy()->subDays(30);
                break;
            case '90days':
                $start = $end->copy()->subDays(90);
                break;
            case 'year':
                $start = $end->copy()->subYear();
                break;
            default:
                $start = $end->copy()->subDays(7);
        }

        return ['start' => $start, 'end' => $end];
    }

    /**
     * Helper: Format order status datasets for charts
     */
    private function formatOrderStatusDatasets($data)
    {
        $statuses = ['pending', 'confirmed', 'processing', 'shipped', 'delivered', 'cancelled'];
        $datasets = [];

        foreach ($statuses as $status) {
            $datasets[] = [
                'label' => ucfirst($status),
                'data' => $data->where('status', $status)->pluck('count')->toArray(),
            ];
        }

        return $datasets;
    }

    /**
     * Helper: Format user role datasets for charts
     */
    private function formatUserRoleDatasets($data)
    {
        $roles = ['admin', 'store_owner', 'client', 'delivery_person'];
        $datasets = [];

        foreach ($roles as $role) {
            $datasets[] = [
                'label' => ucfirst(str_replace('_', ' ', $role)),
                'data' => $data->where('role_type', $role)->pluck('count')->toArray(),
            ];
        }

        return $datasets;
    }
}
```

### 🆕 3. SettingsController (NEW)
```bash
php artisan make:controller SettingsController
```

**File:** `app/Http/Controllers/SettingsController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Setting;
use Illuminate\Http\Request;

class SettingsController extends Controller
{
    public function __construct()
    {
        $this->middleware(['auth:sanctum', 'admin']);
    }

    /**
     * Get all settings grouped by category
     */
    public function index()
    {
        $settings = Setting::all()->groupBy('group');
        
        return response()->json([
            'success' => true,
            'data' => $settings
        ]);
    }

    /**
     * Get specific setting
     */
    public function show($key)
    {
        $setting = Setting::where('key', $key)->first();
        
        if (!$setting) {
            return response()->json([
                'success' => false,
                'message' => 'Setting not found'
            ], 404);
        }

        return response()->json([
            'success' => true,
            'data' => $setting
        ]);
    }

    /**
     * Update or create setting
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'key' => 'required|string|max:255',
            'value' => 'required',
            'type' => 'required|in:string,integer,boolean,json',
            'group' => 'required|string',
            'description' => 'nullable|string'
        ]);

        $setting = Setting::set(
            $validated['key'],
            $validated['value'],
            $validated['type'],
            $validated['group']
        );

        if (isset($validated['description'])) {
            $setting->description = $validated['description'];
            $setting->save();
        }

        return response()->json([
            'success' => true,
            'message' => 'Setting updated successfully',
            'data' => $setting
        ]);
    }

    /**
     * Bulk update settings
     */
    public function bulkUpdate(Request $request)
    {
        $validated = $request->validate([
            'settings' => 'required|array',
            'settings.*.key' => 'required|string',
            'settings.*.value' => 'required'
        ]);

        foreach ($validated['settings'] as $settingData) {
            Setting::set($settingData['key'], $settingData['value']);
        }

        return response()->json([
            'success' => true,
            'message' => 'Settings updated successfully'
        ]);
    }

    /**
     * Delete setting
     */
    public function destroy($key)
    {
        $setting = Setting::where('key', $key)->first();
        
        if (!$setting) {
            return response()->json([
                'success' => false,
                'message' => 'Setting not found'
            ], 404);
        }

        $setting->delete();

        return response()->json([
            'success' => true,
            'message' => 'Setting deleted successfully'
        ]);
    }
}
```

### 🆕 4. PromotionsController (NEW)
```bash
php artisan make:controller PromotionsController
```

**File:** `app/Http/Controllers/PromotionsController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Promotion;
use App\Models\Banner;
use App\Models\DiscountCode;
use Illuminate\Http\Request;

class PromotionsController extends Controller
{
    public function __construct()
    {
        $this->middleware(['auth:sanctum', 'admin']);
    }

    /**
     * Get all promotions
     */
    public function indexPromotions(Request $request)
    {
        $query = Promotion::query();

        if ($request->has('is_active')) {
            $query->where('is_active', $request->is_active);
        }

        $promotions = $query->latest()->paginate($request->get('per_page', 15));

        return response()->json([
            'success' => true,
            'data' => $promotions
        ]);
    }

    /**
     * Create promotion
     */
    public function storePromotion(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'description' => 'nullable|string',
            'type' => 'required|in:percentage,fixed_amount,buy_x_get_y',
            'discount_value' => 'required|numeric|min:0',
            'min_purchase_amount' => 'nullable|numeric|min:0',
            'max_uses' => 'nullable|integer|min:1',
            'start_date' => 'required|date',
            'end_date' => 'required|date|after:start_date',
            'applicable_to' => 'nullable|array'
        ]);

        $promotion = Promotion::create($validated);

        return response()->json([
            'success' => true,
            'message' => 'Promotion created successfully',
            'data' => $promotion
        ], 201);
    }

    /**
     * Update promotion
     */
    public function updatePromotion(Request $request, $id)
    {
        $promotion = Promotion::findOrFail($id);

        $validated = $request->validate([
            'name' => 'sometimes|string|max:255',
            'description' => 'nullable|string',
            'type' => 'sometimes|in:percentage,fixed_amount,buy_x_get_y',
            'discount_value' => 'sometimes|numeric|min:0',
            'min_purchase_amount' => 'nullable|numeric|min:0',
            'max_uses' => 'nullable|integer|min:1',
            'is_active' => 'sometimes|boolean',
            'start_date' => 'sometimes|date',
            'end_date' => 'sometimes|date|after:start_date',
            'applicable_to' => 'nullable|array'
        ]);

        $promotion->update($validated);

        return response()->json([
            'success' => true,
            'message' => 'Promotion updated successfully',
            'data' => $promotion
        ]);
    }

    /**
     * Delete promotion
     */
    public function destroyPromotion($id)
    {
        $promotion = Promotion::findOrFail($id);
        $promotion->delete();

        return response()->json([
            'success' => true,
            'message' => 'Promotion deleted successfully'
        ]);
    }

    /**
     * Get all banners
     */
    public function indexBanners(Request $request)
    {
        $query = Banner::query();

        if ($request->has('is_active')) {
            $query->where('is_active', $request->is_active);
        }

        $banners = $query->orderBy('display_order')->paginate($request->get('per_page', 15));

        return response()->json([
            'success' => true,
            'data' => $banners
        ]);
    }

    /**
     * Create banner
     */
    public function storeBanner(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'image_url' => 'required|string',
            'link_type' => 'required|in:product,category,store,external,none',
            'link_value' => 'nullable|string',
            'display_order' => 'nullable|integer',
            'start_date' => 'required|date',
            'end_date' => 'nullable|date|after:start_date'
        ]);

        $banner = Banner::create($validated);

        return response()->json([
            'success' => true,
            'message' => 'Banner created successfully',
            'data' => $banner
        ], 201);
    }

    /**
     * Update banner
     */
    public function updateBanner(Request $request, $id)
    {
        $banner = Banner::findOrFail($id);

        $validated = $request->validate([
            'title' => 'sometimes|string|max:255',
            'description' => 'nullable|string',
            'image_url' => 'sometimes|string',
            'link_type' => 'sometimes|in:product,category,store,external,none',
            'link_value' => 'nullable|string',
            'display_order' => 'nullable|integer',
            'is_active' => 'sometimes|boolean',
            'start_date' => 'sometimes|date',
            'end_date' => 'nullable|date|after:start_date'
        ]);

        $banner->update($validated);

        return response()->json([
            'success' => true,
            'message' => 'Banner updated successfully',
            'data' => $banner
        ]);
    }

    /**
     * Delete banner
     */
    public function destroyBanner($id)
    {
        $banner = Banner::findOrFail($id);
        $banner->delete();

        return response()->json([
            'success' => true,
            'message' => 'Banner deleted successfully'
        ]);
    }

    /**
     * Get all discount codes
     */
    public function indexDiscountCodes(Request $request)
    {
        $codes = DiscountCode::with('promotion')
            ->latest()
            ->paginate($request->get('per_page', 15));

        return response()->json([
            'success' => true,
            'data' => $codes
        ]);
    }

    /**
     * Create discount code
     */
    public function storeDiscountCode(Request $request)
    {
        $validated = $request->validate([
            'code' => 'required|string|unique:discount_codes,code',
            'promotion_id' => 'nullable|exists:promotions,id',
            'type' => 'required|in:percentage,fixed_amount',
            'discount_value' => 'required|numeric|min:0',
            'min_purchase_amount' => 'nullable|numeric|min:0',
            'max_uses' => 'nullable|integer|min:1',
            'expires_at' => 'nullable|date'
        ]);

        $code = DiscountCode::create($validated);

        return response()->json([
            'success' => true,
            'message' => 'Discount code created successfully',
            'data' => $code->load('promotion')
        ], 201);
    }

    /**
     * Validate discount code (public endpoint)
     */
    public function validateCode(Request $request)
    {
        $validated = $request->validate([
            'code' => 'required|string',
            'order_amount' => 'required|numeric|min:0'
        ]);

        $code = DiscountCode::where('code', $validated['code'])->first();

        if (!$code) {
            return response()->json([
                'success' => false,
                'message' => 'Invalid discount code'
            ], 404);
        }

        if (!$code->isValid($validated['order_amount'])) {
            return response()->json([
                'success' => false,
                'message' => 'Discount code is not valid or has expired'
            ], 400);
        }

        return response()->json([
            'success' => true,
            'data' => [
                'code' => $code,
                'discount_amount' => $code->apply($validated['order_amount'])
            ]
        ]);
    }
}
```

### 🆕 5. CampaignsController (NEW)
```bash
php artisan make:controller CampaignsController
```

**File:** `app/Http/Controllers/CampaignsController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Campaign;
use App\Models\User;
use Illuminate\Http\Request;

class CampaignsController extends Controller
{
    public function __construct()
    {
        $this->middleware(['auth:sanctum', 'admin']);
    }

    /**
     * Get all campaigns
     */
    public function index(Request $request)
    {
        $query = Campaign::with('creator:id,name,email');

        if ($request->has('status')) {
            $query->where('status', $request->status);
        }

        if ($request->has('type')) {
            $query->where('type', $request->type);
        }

        $campaigns = $query->latest()->paginate($request->get('per_page', 15));

        return response()->json([
            'success' => true,
            'data' => $campaigns
        ]);
    }

    /**
     * Get single campaign with metrics
     */
    public function show($id)
    {
        $campaign = Campaign::with('creator')->findOrFail($id);
        
        return response()->json([
            'success' => true,
            'data' => [
                'campaign' => $campaign,
                'metrics' => $campaign->getMetrics()
            ]
        ]);
    }

    /**
     * Create campaign
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'description' => 'nullable|string',
            'type' => 'required|in:email,sms,push_notification,in_app',
            'target_audience' => 'required|array',
            'message_content' => 'required|string',
            'image_url' => 'nullable|string',
            'action_type' => 'nullable|in:link,discount_code,product',
            'action_value' => 'nullable|string',
            'scheduled_at' => 'nullable|date'
        ]);

        $validated['created_by'] = auth()->id();
        $validated['status'] = 'draft';

        // Calculate target count based on audience filters
        $validated['target_count'] = $this->calculateTargetCount($validated['target_audience']);

        $campaign = Campaign::create($validated);

        return response()->json([
            'success' => true,
            'message' => 'Campaign created successfully',
            'data' => $campaign
        ], 201);
    }

    /**
     * Update campaign
     */
    public function update(Request $request, $id)
    {
        $campaign = Campaign::findOrFail($id);

        $validated = $request->validate([
            'name' => 'sometimes|string|max:255',
            'description' => 'nullable|string',
            'type' => 'sometimes|in:email,sms,push_notification,in_app',
            'status' => 'sometimes|in:draft,scheduled,active,completed,paused',
            'target_audience' => 'sometimes|array',
            'message_content' => 'sometimes|string',
            'image_url' => 'nullable|string',
            'action_type' => 'nullable|in:link,discount_code,product',
            'action_value' => 'nullable|string',
            'scheduled_at' => 'nullable|date'
        ]);

        if (isset($validated['target_audience'])) {
            $validated['target_count'] = $this->calculateTargetCount($validated['target_audience']);
        }

        $campaign->update($validated);

        return response()->json([
            'success' => true,
            'message' => 'Campaign updated successfully',
            'data' => $campaign->fresh('creator')
        ]);
    }

    /**
     * Delete campaign
     */
    public function destroy($id)
    {
        $campaign = Campaign::findOrFail($id);
        
        if (in_array($campaign->status, ['active', 'completed'])) {
            return response()->json([
                'success' => false,
                'message' => 'Cannot delete active or completed campaigns'
            ], 400);
        }

        $campaign->delete();

        return response()->json([
            'success' => true,
            'message' => 'Campaign deleted successfully'
        ]);
    }

    /**
     * Send campaign immediately
     */
    public function send($id)
    {
        $campaign = Campaign::findOrFail($id);

        if ($campaign->status !== 'draft' && $campaign->status !== 'scheduled') {
            return response()->json([
                'success' => false,
                'message' => 'Campaign cannot be sent in current status'
            ], 400);
        }

        // TODO: Implement actual sending logic based on campaign type
        // For now, just update status
        $campaign->update([
            'status' => 'active',
            'sent_at' => now()
        ]);

        return response()->json([
            'success' => true,
            'message' => 'Campaign sending initiated',
            'data' => $campaign
        ]);
    }

    /**
     * Calculate target audience count
     */
    private function calculateTargetCount($targetAudience)
    {
        $query = User::query();

        if (isset($targetAudience['role'])) {
            $query->where('role_type', $targetAudience['role']);
        }

        if (isset($targetAudience['location'])) {
            $query->where('location', 'LIKE', "%{$targetAudience['location']}%");
        }

        if (isset($targetAudience['is_active'])) {
            $query->where('is_active', $targetAudience['is_active']);
        }

        return $query->count();
    }
}
```

### 🆕 6. DeliveryServicesController (NEW)
```bash
php artisan make:controller DeliveryServicesController
```

**File:** `app/Http/Controllers/DeliveryServicesController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\DeliveryService;
use Illuminate\Http\Request;

class DeliveryServicesController extends Controller
{
    public function __construct()
    {
        $this->middleware(['auth:sanctum', 'admin'])->except(['index', 'calculateFee']);
    }

    /**
     * Get all delivery services
     */
    public function index(Request $request)
    {
        $query = DeliveryService::query();

        if ($request->has('is_active')) {
            $query->where('is_active', $request->is_active);
        }

        if ($request->has('type')) {
            $query->where('type', $request->type);
        }

        $services = $query->orderBy('estimated_time_minutes')->get();

        return response()->json([
            'success' => true,
            'data' => $services
        ]);
    }

    /**
     * Get single service
     */
    public function show($id)
    {
        $service = DeliveryService::findOrFail($id);

        return response()->json([
            'success' => true,
            'data' => $service
        ]);
    }

    /**
     * Create delivery service
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'description' => 'nullable|string',
            'type' => 'required|in:standard,express,scheduled,pickup',
            'base_fee' => 'required|numeric|min:0',
            'per_km_fee' => 'nullable|numeric|min:0',
            'estimated_time_minutes' => 'required|integer|min:1',
            'availability' => 'nullable|array'
        ]);

        $service = DeliveryService::create($validated);

        return response()->json([
            'success' => true,
            'message' => 'Delivery service created successfully',
            'data' => $service
        ], 201);
    }

    /**
     * Update delivery service
     */
    public function update(Request $request, $id)
    {
        $service = DeliveryService::findOrFail($id);

        $validated = $request->validate([
            'name' => 'sometimes|string|max:255',
            'description' => 'nullable|string',
            'type' => 'sometimes|in:standard,express,scheduled,pickup',
            'base_fee' => 'sometimes|numeric|min:0',
            'per_km_fee' => 'nullable|numeric|min:0',
            'estimated_time_minutes' => 'sometimes|integer|min:1',
            'is_active' => 'sometimes|boolean',
            'availability' => 'nullable|array'
        ]);

        $service->update($validated);

        return response()->json([
            'success' => true,
            'message' => 'Delivery service updated successfully',
            'data' => $service
        ]);
    }

    /**
     * Delete delivery service
     */
    public function destroy($id)
    {
        $service = DeliveryService::findOrFail($id);
        $service->delete();

        return response()->json([
            'success' => true,
            'message' => 'Delivery service deleted successfully'
        ]);
    }

    /**
     * Calculate delivery fee
     */
    public function calculateFee(Request $request)
    {
        $validated = $request->validate([
            'service_id' => 'required|exists:delivery_services,id',
            'distance_km' => 'nullable|numeric|min:0'
        ]);

        $service = DeliveryService::findOrFail($validated['service_id']);
        $distance = $validated['distance_km'] ?? 0;

        $fee = $service->calculateFee($distance);

        return response()->json([
            'success' => true,
            'data' => [
                'service' => $service,
                'distance_km' => $distance,
                'calculated_fee' => $fee
            ]
        ]);
    }
}
```

### 🆕 7. ReportsController (NEW)
```bash
php artisan make:controller ReportsController
```

**File:** `app/Http/Controllers/ReportsController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Order;
use App\Models\User;
use App\Models\Product;
use App\Models\Stores;
use App\Models\SystemReport;
use Illuminate\Http\Request;
use Carbon\Carbon;
use Illuminate\Support\Facades\DB;

class ReportsController extends Controller
{
    public function __construct()
    {
        $this->middleware(['auth:sanctum', 'admin']);
    }

    /**
     * Generate sales report
     */
    public function salesReport(Request $request)
    {
        $validated = $request->validate([
            'start_date' => 'required|date',
            'end_date' => 'required|date|after:start_date',
            'group_by' => 'nullable|in:day,week,month'
        ]);

        $groupBy = $validated['group_by'] ?? 'day';

        $data = Order::where('status', 'delivered')
            ->whereBetween('created_at', [$validated['start_date'], $validated['end_date']])
            ->select(
                DB::raw($this->getDateGrouping($groupBy) . ' as period'),
                DB::raw('COUNT(*) as total_orders'),
                DB::raw('SUM(total_amount) as total_revenue'),
                DB::raw('AVG(total_amount) as average_order_value')
            )
            ->groupBy('period')
            ->orderBy('period')
            ->get();

        return response()->json([
            'success' => true,
            'data' => [
                'report_type' => 'sales',
                'period' => $groupBy,
                'start_date' => $validated['start_date'],
                'end_date' => $validated['end_date'],
                'summary' => [
                    'total_orders' => $data->sum('total_orders'),
                    'total_revenue' => $data->sum('total_revenue'),
                    'average_order_value' => $data->avg('average_order_value'),
                ],
                'data' => $data
            ]
        ]);
    }

    /**
     * Generate user analytics report
     */
    public function userAnalytics(Request $request)
    {
        $validated = $request->validate([
            'start_date' => 'required|date',
            'end_date' => 'required|date|after:start_date'
        ]);

        $data = [
            'total_users' => User::whereBetween('created_at', [$validated['start_date'], $validated['end_date']])->count(),
            'by_role' => User::whereBetween('created_at', [$validated['start_date'], $validated['end_date']])
                ->select('role_type', DB::raw('COUNT(*) as count'))
                ->groupBy('role_type')
                ->get(),
            'active_users' => User::where('is_active', true)
                ->whereBetween('created_at', [$validated['start_date'], $validated['end_date']])
                ->count(),
            'users_with_orders' => User::whereHas('orders')
                ->whereBetween('created_at', [$validated['start_date'], $validated['end_date']])
                ->count(),
        ];

        return response()->json([
            'success' => true,
            'data' => $data
        ]);
    }

    /**
     * Generate product performance report
     */
    public function productPerformance(Request $request)
    {
        $validated = $request->validate([
            'start_date' => 'required|date',
            'end_date' => 'required|date|after:start_date',
            'limit' => 'nullable|integer|min:1|max:100'
        ]);

        $limit = $validated['limit'] ?? 20;

        $topProducts = Product::select('products.*')
            ->join('order_items', 'products.id', '=', 'order_items.product_id')
            ->join('orders', 'order_items.order_id', '=', 'orders.id')
            ->whereBetween('orders.created_at', [$validated['start_date'], $validated['end_date']])
            ->where('orders.status', 'delivered')
            ->selectRaw('SUM(order_items.quantity) as total_sold')
            ->selectRaw('SUM(order_items.total) as total_revenue')
            ->groupBy('products.id')
            ->orderBy('total_sold', 'desc')
            ->limit($limit)
            ->get();

        return response()->json([
            'success' => true,
            'data' => [
                'top_products' => $topProducts,
                'low_stock_alert' => Product::where('stock_quantity', '<', 10)->count(),
                'out_of_stock' => Product::where('stock_quantity', 0)->count(),
            ]
        ]);
    }

    /**
     * Generate store performance report
     */
    public function storePerformance(Request $request)
    {
        $validated = $request->validate([
            'start_date' => 'required|date',
            'end_date' => 'required|date|after:start_date'
        ]);

        $stores = Stores::with('user')
            ->withCount(['orders' => function($query) use ($validated) {
                $query->whereBetween('created_at', [$validated['start_date'], $validated['end_date']]);
            }])
            ->orderBy('orders_count', 'desc')
            ->get();

        return response()->json([
            'success' => true,
            'data' => $stores
        ]);
    }

    /**
     * Export report to file
     */
    public function exportReport(Request $request)
    {
        $validated = $request->validate([
            'report_type' => 'required|in:sales,users,products,stores',
            'start_date' => 'required|date',
            'end_date' => 'required|date|after:start_date',
            'format' => 'required|in:csv,excel,pdf'
        ]);

        // Generate report based on type
        // Export to file
        // Return download link

        return response()->json([
            'success' => true,
            'message' => 'Report export initiated',
            'download_url' => '/api/admin/reports/download/...'
        ]);
    }

    /**
     * Helper: Get SQL date grouping based on period
     */
    private function getDateGrouping($groupBy)
    {
        switch ($groupBy) {
            case 'week':
                return "DATE_FORMAT(created_at, '%Y-%u')";
            case 'month':
                return "DATE_FORMAT(created_at, '%Y-%m')";
            case 'day':
            default:
                return "DATE(created_at)";
        }
    }
}
```

---

## 🛣️ API Routes to Add

**File:** `routes/api.php`

Add these routes:

```php
// Admin Dashboard Routes
Route::middleware(['auth:sanctum', 'admin'])->prefix('admin')->group(function () {
    
    // Dashboard Overview
    Route::get('/dashboard', [DashboardController::class, 'getStats']);
    Route::get('/dashboard/stats', [AdminController::class, 'dashboard']);
    
    // User Management
    Route::get('/users', [AdminController::class, 'getUsers']);
    Route::patch('/users/{id}/status', [AdminController::class, 'updateUserStatus']);
    Route::get('/users/{id}', [AdminController::class, 'getUserDetails']);
    Route::delete('/users/{id}', [AdminController::class, 'deleteUser']);
    
    // Store Management
    Route::get('/stores', [AdminController::class, 'getStores']);
    Route::post('/stores/{id}/approve', [AdminController::class, 'approveStore']);
    Route::patch('/stores/{id}', [AdminController::class, 'updateStore']);
    Route::delete('/stores/{id}', [AdminController::class, 'deleteStore']);
    
    // Product Management  
    Route::get('/products', [AdminController::class, 'getAllProducts']);
    Route::patch('/products/{id}/status', [AdminController::class, 'updateProductStatus']);
    Route::delete('/products/{id}', [AdminController::class, 'deleteProduct']);
    
    // Order Management
    Route::get('/orders', [AdminController::class, 'getOrders']);
    Route::patch('/orders/{id}/status', [AdminController::class, 'updateOrderStatus']);
    Route::get('/orders/{id}', [AdminController::class, 'getOrderDetails']);
    
    // Category Management
    Route::get('/categories', [AdminController::class, 'getCategories']);
    Route::post('/categories', [AdminController::class, 'createCategory']);
    Route::patch('/categories/{id}', [AdminController::class, 'updateCategory']);
    Route::delete('/categories/{id}', [AdminController::class, 'deleteCategory']);
    
    // Settings
    Route::get('/settings', [SettingsController::class, 'index']);
    Route::get('/settings/{key}', [SettingsController::class, 'show']);
    Route::post('/settings', [SettingsController::class, 'store']);
    Route::post('/settings/bulk', [SettingsController::class, 'bulkUpdate']);
    Route::delete('/settings/{key}', [SettingsController::class, 'destroy']);
    
    // Reports
    Route::get('/reports/sales', [ReportsController::class, 'salesReport']);
    Route::get('/reports/users', [ReportsController::class, 'userAnalytics']);
    Route::get('/reports/products', [ReportsController::class, 'productPerformance']);
    Route::get('/reports/stores', [ReportsController::class, 'storePerformance']);
    Route::post('/reports/export', [ReportsController::class, 'exportReport']);
    
    // Promotions & Banners
    Route::get('/promotions', [PromotionsController::class, 'indexPromotions']);
    Route::post('/promotions', [PromotionsController::class, 'storePromotion']);
    Route::patch('/promotions/{id}', [PromotionsController::class, 'updatePromotion']);
    Route::delete('/promotions/{id}', [PromotionsController::class, 'destroyPromotion']);
    
    Route::get('/banners', [PromotionsController::class, 'indexBanners']);
    Route::post('/banners', [PromotionsController::class, 'storeBanner']);
    Route::patch('/banners/{id}', [PromotionsController::class, 'updateBanner']);
    Route::delete('/banners/{id}', [PromotionsController::class, 'destroyBanner']);
    
    Route::get('/discount-codes', [PromotionsController::class, 'indexDiscountCodes']);
    Route::post('/discount-codes', [PromotionsController::class, 'storeDiscountCode']);
    
    // Campaigns
    Route::get('/campaigns', [CampaignsController::class, 'index']);
    Route::get('/campaigns/{id}', [CampaignsController::class, 'show']);
    Route::post('/campaigns', [CampaignsController::class, 'store']);
    Route::patch('/campaigns/{id}', [CampaignsController::class, 'update']);
    Route::delete('/campaigns/{id}', [CampaignsController::class, 'destroy']);
    Route::post('/campaigns/{id}/send', [CampaignsController::class, 'send']);
    
    // Delivery Services
    Route::get('/delivery-services', [DeliveryServicesController::class, 'index']);
    Route::get('/delivery-services/{id}', [DeliveryServicesController::class, 'show']);
    Route::post('/delivery-services', [DeliveryServicesController::class, 'store']);
    Route::patch('/delivery-services/{id}', [DeliveryServicesController::class, 'update']);
    Route::delete('/delivery-services/{id}', [DeliveryServicesController::class, 'destroy']);
    
    // Activity Logs
    Route::get('/activity-logs', [AdminController::class, 'getActivityLogs']);
});

// Public routes
Route::middleware(['auth:sanctum'])->group(function () {
    // Validate discount code (for customers during checkout)
    Route::post('/discount-codes/validate', [PromotionsController::class, 'validateCode']);
    
    // Calculate delivery fee (for customers)
    Route::post('/delivery-services/calculate-fee', [DeliveryServicesController::class, 'calculateFee']);
});
```

---

## ✅ TODO Checklist

### Phase 1: Database Setup (Day 1)
- [ ] Run existing migrations to understand database structure
- [ ] Create 6 new migrations:
  - [ ] `create_settings_table`
  - [ ] `create_admin_activity_logs_table`
  - [ ] `create_system_reports_table`
  - [ ] `create_promotions_and_banners_tables`
  - [ ] `create_campaigns_table`
  - [ ] `create_delivery_services_table`
- [ ] Run migrations: `php artisan migrate`
- [ ] Verify all tables created successfully

### Phase 2: Models Creation (Day 1-2)
- [ ] Create `Setting` model with helper methods
- [ ] Create `AdminActivityLog` model
- [ ] Create `SystemReport` model
- [ ] Create `Promotion` model with validation logic
- [ ] Create `Banner` model with visibility checks
- [ ] Create `DiscountCode` model with apply method
- [ ] Create `Campaign` model with metrics
- [ ] Create `DeliveryService` model with fee calculation
- [ ] Define all relationships in models
- [ ] Test models with Tinker: `php artisan tinker`

### Phase 3: Controllers Implementation (Day 2-5)
- [ ] Create `DashboardController` with all methods
- [ ] Create `SettingsController` with CRUD operations
- [ ] Create `ReportsController` with report generation
- [ ] Create `PromotionsController` for promotions, banners, discount codes
- [ ] Create `CampaignsController` for marketing campaigns
- [ ] Create `DeliveryServicesController` for delivery options management
- [ ] Enhance existing `AdminController` with new methods:
  - [ ] `getDashboardStats()`
  - [ ] `getSettings()`
  - [ ] `updateSettings()`
  - [ ] `exportData()`
  - [ ] `getActivityLogs()`
  - [ ] Add missing methods for categories management
  - [ ] Add missing methods for product management

### Phase 4: API Routes (Day 4)
- [ ] Add all admin dashboard routes to `routes/api.php`
- [ ] Add middleware protection (`auth:sanctum`, `admin`)
- [ ] Test routes with Postman/Insomnia:
  - [ ] Dashboard stats endpoint
  - [ ] User management endpoints
  - [ ] Store management endpoints
  - [ ] Order management endpoints
  - [ ] Settings endpoints
  - [ ] Reports endpoints
  - [ ] Promotions & Banners endpoints
  - [ ] Campaigns endpoints
  - [ ] Delivery services endpoints

### Phase 5: Admin Middleware (Day 5)
- [ ] Create admin middleware if doesn't exist: `php artisan make:middleware AdminMiddleware`
- [ ] Register middleware in `app/Http/Kernel.php`
- [ ] Add `isAdmin()` method to User model if not exists:
```php
public function isAdmin()
{
    return $this->role_type === 'admin';
}
```

### Phase 6: Seed Sample Data (Day 5)
- [ ] Create seeders for testing:
  - [ ] `php artisan make:seeder SettingsSeeder`
  - [ ] `php artisan make:seeder AdminUserSeeder`
  - [ ] Seed default settings (tax rate, delivery fee, etc.)
  - [ ] Create test admin user
- [ ] Run seeders: `php artisan db:seed`

### Phase 7: Testing (Day 7-9)
- [ ] Test all dashboard endpoints with Postman
- [ ] Test user management (CRUD)
- [ ] Test store approval workflow
- [ ] Test order status updates
- [ ] Test settings management
- [ ] Test reports generation
- [ ] Test promotions, banners, and discount codes
- [ ] Test campaign creation and sending
- [ ] Test delivery services and fee calculation
- [ ] Document API responses

### Phase 8: Documentation (Day 7)
- [ ] Create API documentation file
- [ ] Document all endpoints with:
  - Request method (GET, POST, PATCH, DELETE)
  - URL
  - Required headers (Authorization)
  - Request body examples
  - Response examples
  - Error responses
- [ ] Create Postman collection for all endpoints

### Phase 9: Security & Validation (Day 8)
- [ ] Add request validation to all controllers
- [ ] Implement rate limiting for API endpoints
- [ ] Add activity logging for sensitive actions
- [ ] Test authorization (only admins can access)
- [ ] Add CORS configuration if needed

### Phase 10: Frontend Integration Preparation (Day 9)
- [ ] Create detailed API documentation for Flutter developer
- [ ] Provide sample requests/responses
- [ ] List all required API endpoints
- [ ] Define data models/DTOs for Flutter
- [ ] Test with sample frontend requests

---

## 📊 Admin Dashboard Sections (Based on Image)

### 1. **Parameters / الإعدادات (Settings)**
- System configuration
- Update language
- Privacy settings
- User management interface

### 2. **Support / الدعم (Support)**
- Help center
- Contact forms
- Technical support
- Manage delivery updates

### 3. **Content / المحتوى (Content)**
- Manage advertising & promotions
- Control main navigation
- Delivery tracking
- Advanced media settings

### 4. **Delivery / التوصيل (Delivery)**
- Delivery management
- Driver tracking
- Delivery schedules
- Route optimization

### 6. **Requests / الطلبات (Orders)**
- Order list with filters
- Order details view
- Status updates
- Order tracking

### 7. **Products / المنتجات (Products)**
- Product catalog
- Add/Edit/Delete products
- Product categories
- Stock management

### 8. **Stores / المتاجر (Stores)**
- Store list
- Store approval/rejection
- Store performance metrics
- Store owner management

### 9. **Users / المستخدمين (Users)**
- User list (all roles)
- User activation/deactivation
- User role management
- User activity tracking

### 10. **Home / الصفحة الرئيسية (Dashboard Home)**
- Statistics overview
- Charts & graphs
- Recent activities
- Quick actions

---

## 🧪 Testing Endpoints

Use these curl commands or Postman:

```bash
# Login as admin first
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@delpo.com","password":"password"}'

# Get dashboard stats
curl -X GET http://localhost:8000/api/admin/dashboard \
  -H "Authorization: Bearer YOUR_TOKEN"

# Get all users
curl -X GET http://localhost:8000/api/admin/users \
  -H "Authorization: Bearer YOUR_TOKEN"

# Approve a store
curl -X POST http://localhost:8000/api/admin/stores/1/approve \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"action":"approve"}'

# Update order status
curl -X PATCH http://localhost:8000/api/admin/orders/1/status \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status":"confirmed"}'
```

---

## 📝 Notes 

1. **Authentication**: All admin routes require `auth:sanctum` middleware
2. **Authorization**: Create admin middleware to check if user role is 'admin'
3. **Validation**: Always validate request data before processing
4. **Error Handling**: Use try-catch blocks and return proper error responses
5. **Logging**: Log important admin actions using `AdminActivityLog`
6. **Performance**: Use eager loading (`with()`) to avoid N+1 queries
7. **Pagination**: Use `paginate()` for large datasets
8. **Caching**: Consider caching dashboard stats for better performance
9. **API Responses**: Always return consistent JSON structure:
   ```json
   {
     "success": true/false,
     "message": "...",
     "data": {...}
   }
   ```
10. **Testing**: Test each endpoint thoroughly before moving to next

---

## 🎨 Frontend Requirements (for Flutter Developer)

The dashboard should include:
- 📊 Statistics cards (users, orders, revenue, stores)
- 📈 Charts (line, bar, pie) for analytics
- 📋 Data tables with filters, search, pagination
- 🔔 Real-time notifications
- 📱 Responsive design (works on tablets & desktop)
- 🌙 Dark mode support
- 🌍 RTL support (Arabic)
- 🔐 Secure login/logout
- 📥 Export functionality (CSV, Excel)

---

## 🚀 Estimated Timeline

- **Day 1-2**: Database & Models (6 migrations, 8 models)
- **Day 3-6**: Controllers & API Routes (7 controllers)
- **Day 7-9**: Testing & Bug Fixes
- **Day 10-11**: Documentation & Security
- **Day 12**: Final review & handoff to Flutter developer

**Total**: 12 working days for complete backend admin dashboard with all features

---

## 📞 Support

If you have questions during development:
1. Check existing controllers for examples
2. Review Laravel documentation
3. Test with Postman/Insomnia
4. Document any issues or blockers

**Good luck! 🎉**
