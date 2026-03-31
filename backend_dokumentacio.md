# Vizsgaremek – dokumentáció (Backend)
---
**Projekt neve:** Barbershop
---
**Készítette:** Kiss Zoltán Máté, Márki Zoltán Ákos, Máté Bálint Ákos 
---


**Fő funkciók:**
- Teljes auth flow (regisztráció, login, email verifikáció, password reset)
- Fodrász lista + részletek + következő szabad idősáv
- Időpont foglalás (elérhetőség-ellenőrzés)
- Fodrász saját foglalás-kezelés
- Hajstílusok (services) CRUD
- Galéria képek CRUD
- Admin panel (felhasználók, fodrászok, foglalások teljes kezelése)

---

Teljes mappa-struktúra
```bash
├── app/
│   ├── Console/
│   ├── Http/Controllers/Api/          ← összes API controller
│   ├── Jobs/
│   ├── Mail/
│   ├── Models/                        ← User, Barber, Booking, Hairstyle, GalleryImage
│   ├── Providers/
│   └── Services/                      ← BookingService, AvailabilityService stb.
├── bootstrap/
├── config/                            ← sanctum.php, auth.php stb.
├── database/
│   ├── factories/
│   ├── migrations/                    ← users, barbers, bookings, hairstyles, gallery_images 
│   └── seeders/
├── public/
├── resources/views/                   
├── routes/
│   ├── api.php                        ← teljes API
│   └── web.php
├── storage/
├── tests/
├── .env.example
├── artisan
├── composer.json
├── package.json
├── vite.config.js
```
### Adatbázis séma (fő mezői)
```
users: id, name, email, email_verified_at, password, role
barbers: id, user_id, name, bio, availability (JSON)
bookings: id, user_id, barber_id, hairstyle_id, start_time, end_time, status
hairstyles: id, name, description, price, duration
gallery_images: id, image_path, description
```
### Migrációk – Teljes lista és séma
---
```
0001_01_01_000000_create_users_table.php
Létrehozza a users táblát (standard Laravel auth tábla: id, name, email, email_verified_at, password, remember_token, timestamps).
```
```
0001_01_01_000001_create_cache_table.php
Cache tábla Laravelhez.
```
```
0001_01_01_000002_create_jobs_table.php
Queue jobs tábla (background feladatokhoz).
```
```
2025_02_24_000001_create_barbers_table.php
barbers tábla: barber adatok (name, bio, phone, availability stb.).
```
```
2025_02_24_000002_create_hairstyles_table.php
hairstyles tábla: szolgáltatások (name, description, price, duration_minutes, image_path?).
```
```
2025_02_24_000003_create_bookings_table.php
bookings tábla: foglalások (user_id, barber_id, hairstyle_id, start_time, end_time, status, notes).
```
```
2025_02_24_000004_create_gallery_images_table.php
gallery_images tábla: galéria képek (image_path, description, uploaded_by?).
```
```
2025_02_24_000005_add_role_to_users_table.php
Hozzáadja a role oszlopot a users táblához (enum: 'admin', 'barber', 'user').
```
```
2026_02_24_071718_create_personal_access_tokens_table.php
Sanctum token tábla (API autentikációhoz).
```
```
2026_02_25_085739_add_user_id_to_barbers_table.php
Hozzáadja a user_id foreign key-t a barbers táblához (egy user lehet barber).
```

### Fontos kapcsolatok:
---
```
User 1:1 Barber (barber user-hez kötve)
User 1:N Booking
Barber 1:N Booking
Hairstyle 1:N Booking
GalleryImage önálló vagy user-hez kötve
```
### routes/web.php 
```bash
PHP<?php

use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});
```
---
### routes/api.php (teljes tartalom + endpoint magyarázat)
---
```bash
<?php

use App\Http\Controllers\Api\AuthController;
use App\Http\Controllers\Api\BarberController;
use App\Http\Controllers\Api\BarberBookingController;
use App\Http\Controllers\Api\BookingController;
use App\Http\Controllers\Api\GalleryController;
use App\Http\Controllers\Api\HairstyleController;
use App\Http\Controllers\Api\UserController;
use Illuminate\Support\Facades\Route;

Route::get('/ping', function () {
    return response()->json(['message' => 'pong']);
});

Route::prefix('auth')->group(function () {
    Route::post('/register', [AuthController::class, 'register']);
    Route::post('/login', [AuthController::class, 'login']);
    Route::post('/forgot-password', [AuthController::class, 'forgotPassword']);
    Route::post('/reset-password', [AuthController::class, 'resetPassword']);

    Route::middleware('auth:sanctum')->group(function () {
        Route::post('/logout', [AuthController::class, 'logout']);
        Route::get('/me', [AuthController::class, 'me']);
        Route::post('/email/resend', [AuthController::class, 'resendVerification']);
    });
});

Route::get('/email/verify/{id}/{hash}', [AuthController::class, 'verifyEmail'])->name('verification.verify');

// Nyilvános barber adatok
Route::get('/barbers', [BarberController::class, 'index']);
Route::get('/barbers/{barber}', [BarberController::class, 'show']);
Route::get('/barbers/{barber}/next-slot', [BarberController::class, 'nextSlot']);
Route::get('/barbers/{barber}/schedule', [BarberController::class, 'schedule']);

// Admin-only
Route::middleware(['auth:sanctum', 'role:admin'])->group(function () {
    Route::post('/barbers', [BarberController::class, 'store']);
    Route::put('/barbers/{barber}', [BarberController::class, 'update']);
    Route::delete('/barbers/{barber}', [BarberController::class, 'destroy']);
});

// Barber dashboard
Route::middleware(['auth:sanctum', 'role:barber,admin'])->group(function () {
    Route::get('/barber/me', [BarberBookingController::class, 'me']);
    Route::get('/barber/bookings', [BarberBookingController::class, 'index']);
    Route::put('/barber/bookings/{booking}', [BarberBookingController::class, 'update']);
    Route::delete('/barber/bookings/{booking}', [BarberBookingController::class, 'destroy']);
});

// Foglalás
Route::get('/availability', [BookingController::class, 'availability']);
Route::post('/bookings', [BookingController::class, 'store']);

Route::middleware('auth:sanctum')->group(function () {
    Route::get('/bookings', [BookingController::class, 'index']);
    Route::get('/bookings/{booking}', [BookingController::class, 'show']);
    Route::delete('/bookings/{booking}', [BookingController::class, 'destroy']);

    Route::middleware('role:admin')->group(function () {
        Route::put('/bookings/{booking}', [BookingController::class, 'update']);
    });
});

// Hajstílusok
Route::get('/hairstyles', [HairstyleController::class, 'index']);
Route::get('/hairstyles/{hairstyle}', [HairstyleController::class, 'show']);

Route::middleware(['auth:sanctum', 'role:admin'])->group(function () {
    Route::post('/hairstyles', [HairstyleController::class, 'store']);
    Route::put('/hairstyles/{hairstyle}', [HairstyleController::class, 'update']);
    Route::delete('/hairstyles/{hairstyle}', [HairstyleController::class, 'destroy']);
});

// Galéria
Route::get('/gallery', [GalleryController::class, 'index']);

Route::middleware(['auth:sanctum', 'role:admin'])->group(function () {
    Route::post('/gallery', [GalleryController::class, 'store']);
    Route::delete('/gallery/{galleryImage}', [GalleryController::class, 'destroy']);

    // Admin user management
    Route::get('/users', [UserController::class, 'index']);
    Route::get('/users/{user}', [UserController::class, 'show']);
    Route::put('/users/{user}', [UserController::class, 'update']);
    Route::delete('/users/{user}', [UserController::class, 'destroy']);
});
```
---
## Modellek
---
### User.php (app/Models/User.php)
---
```bash
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable implements MustVerifyEmail
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasApiTokens, HasFactory, Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var list<string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
        'role',
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var list<string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }

    public function bookings()
    {
        return $this->hasMany(Booking::class);
    }
}

### Barber.php (app/Models/Barber.php)
---
```bash
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Barber extends Model
{
    use HasFactory;

    protected $fillable = [
        'user_id',
        'name',
        'specialization',
        'bio',
        'photo_url',
    ];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function bookings()
    {
        return $this->hasMany(Booking::class);
    }
}
```
Célja: Fodrász profil adatainak tárolása.

### Booking.php (app/Models/Booking.php)
---
```bash
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Booking extends Model
{
    use HasFactory;

    protected $fillable = [
        'barber_id',
        'user_id',
        'customer_name',
        'customer_email',
        'customer_phone',
        'start_at',
        'duration_min',
        'note',
        'status',
    ];

    protected function casts(): array
    {
        return [
            'start_at' => 'datetime',
        ];
    }

    public function barber()
    {
        return $this->belongsTo(Barber::class);
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

Célja: Időpontfoglalás.

### GalleryImage.php (app/Models/GalleryImage.php)
---
```bash
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class GalleryImage extends Model
{
    use HasFactory;

    protected $table = 'gallery_images';

    protected $fillable = [
        'title',
        'image_url',
        'source',
    ];
}
```

### Tesztelés
```
php artisan test 
```
