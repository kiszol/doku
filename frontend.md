## Vizsgaremek – dokumentáció (Frontend)
---
## **Projekt neve:** Barbershop
## **Készítette:** Kiss Zoltán Máté, Márki Zoltán Ákos, Máté Bálint Ákos

**Fő funkciók:**
- Teljes auth flow (regisztráció, bejelentkezés, email verifikáció, jelszó visszaállítás)
- Főoldal – borbélyok listázása kártyás nézetben, következő szabad idősáv megjelenítésével
- SOS Foglalás – gyors időpont a legkorábbi elérhető borbélynál
- Időpont foglalás – részletes foglalási űrlap borbély adataival és hajstílusok listájával
- Galéria – képek rácsos nézetben, lightbox előnézettel
- Barber Dashboard – fodrász saját foglalásainak kezelése (listázás, lemondás)
- Admin Panel – tabfüles vezérlőpult (borbélyok, foglalások, hajstílusok, galéria, felhasználók kezelése)
- Rólunk oldal – szalon bemutató, szolgáltatások, kreditek
- Reszponzív design – Bootstrap 5 alapú, mobil-baráti elrendezés

---
## Teljes mappa-struktúra
```
barbershop-frontend/
├── src/
│   ├── app/
│   │   ├── core/
│   │   │   ├── guards/
│   │   │   │   └── auth.guard.ts           
│   │   │   ├── interceptors/
│   │   │   │   └── auth.interceptor.ts     
│   │   │   ├── models/
│   │   │   │   ├── barber.model.ts         
│   │   │   │   ├── booking.model.ts        
│   │   │   │   ├── gallery.model.ts        
│   │   │   │   ├── hairstyle.model.ts      
│   │   │   │   ├── user.model.ts           
│   │   │   │   └── index.ts                
│   │   │   └── services/
│   │   │       ├── api.service.ts          
│   │   │       ├── auth.service.ts         
│   │   │       ├── barber-bookings.service.ts ← fodrász saját foglalásai
│   │   │       ├── barbers.service.ts      
│   │   │       ├── bookings.service.ts     
│   │   │       ├── gallery.service.ts      ← galéria képek kezelése
│   │   │       ├── hairstyles.service.ts   
│   │   │       ├── users.service.ts        ← admin felhasználókezelés
│   │   │       └── index.ts                
│   │   ├── pages/
│   │   │   ├── about/                      ← Rólunk oldal
│   │   │   ├── admin/                      ← Admin Panel
│   │   │   ├── auth/
│   │   │   │   ├── forgot-password/        ← Elfelejtett jelszó
│   │   │   │   ├── login/                  ← Bejelentkezés
│   │   │   │   ├── register/               ← Regisztráció
│   │   │   │   ├── reset-password/         ← Jelszó visszaállítás
│   │   │   │   └── verify-email/           ← Email megerősítés
│   │   │   ├── barber-dashboard/           
│   │   │   ├── booking/                    
│   │   │   ├── booking-success/            
│   │   │   ├── gallery/                    
│   │   │   └── home/                       ← Főoldal
│   │   ├── shared/
│   │   │   └── components/
│   │   │       ├── barber-card/            
│   │   │       ├── footer/                 
│   │   │       ├── navbar/                 
│   │   │       └── toast/                  
│   │   ├── app.component.html
│   │   ├── app.component.scss
│   │   ├── app.component.ts                
│   │   ├── app.config.ts                   
│   │   └── app.routes.ts                   ← Összes útvonal definíciója
│   ├── environments/
│   │   ├── environment.ts                  
│   │   └── environment.prod.ts             
│   ├── index.html                          ← HTML belépőpont
│   ├── main.ts                             
│   └── styles.scss                         ← Stílusok
├── angular.json                            
├── package.json                            
├── tsconfig.json                           
├── tsconfig.app.json                       
└── tsconfig.spec.json                      
```
---
## App konfiguráció (app.config.ts)
```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withInMemoryScrolling } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { routes } from './app.routes';
import { authInterceptor } from './core/interceptors/auth.interceptor';
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withInMemoryScrolling({ anchorScrolling: 'enabled', scrollPositionRestoration: 'enabled' })
    ),
    provideHttpClient(withInterceptors([authInterceptor]))
  ]
};
```

### Útvonalak táblázata
| URL | Komponens | Guard | Leírás |
|---|---|---|---|
| `/` | `HomeComponent` | – | Főoldal, borbélyok listázása |
| `/login` | `LoginComponent` | `guestGuard` | Bejelentkezés (csak nem bejelentkezett usernek) |
| `/register` | `RegisterComponent` | `guestGuard` | Regisztráció (csak nem bejelentkezett usernek) |
| `/forgot-password` | `ForgotPasswordComponent` | – | Elfelejtett jelszó kérés |
| `/reset-password` | `ResetPasswordComponent` | – | Jelszó visszaállítás tokennel |
| `/verify-email/:id/:hash` | `VerifyEmailComponent` | – | Email megerősítés linkből |
| `/booking/:barberId` | `BookingComponent` | – | Időpontfoglalás adott borbélynál |
| `/booking-success` | `BookingSuccessComponent` | – | Sikeres foglalás visszaigazolása |
| `/gallery` | `GalleryComponent` | – | Galéria oldal |
| `/about` | `AboutComponent` | – | Rólunk oldal |
| `/admin` | `AdminComponent` | `adminGuard` | Admin panel (csak adminnak) |
| `/barber-dashboard` | `BarberDashboardComponent` | `barberGuard` | Fodrász dashboard (fodrásznak/adminnak) |
| `**` | – | – | Ismeretlen URL → főoldalra irányít |
---
## Gyök komponens (AppComponent)
### app.component.ts
```typescript
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, RouterOutlet, NavbarComponent, FooterComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent implements OnInit {
  title = 'barbershop-frontend';
  showLayout = true;
  private authRoutes = ['/login', '/register', '/forgot-password', '/reset-password', '/verify-email'];
  constructor(
    private authService: AuthService,
    private router: Router
  ) {
    this.router.events.pipe(
      filter(event => event instanceof NavigationEnd)
    ).subscribe((event: any) => {
      this.showLayout = !this.authRoutes.some(route => event.urlAfterRedirects.startsWith(route));
    });
  }
  ngOnInit(): void {
    this.authService.me().subscribe({
      error: () => {}
    });
  }
}
```
### app.component.html
```html
<app-navbar *ngIf="showLayout"></app-navbar>
<main [class]="showLayout ? 'd-flex flex-column min-vh-100' : ''">
  <router-outlet></router-outlet>
</main>
<app-footer *ngIf="showLayout"></app-footer>
```
### Modellek (src/app/core/models/)

#### user.model.ts
```typescript
export interface User {
  id: number;
  name: string;
  email: string;
  email_verified_at: string | null;
  role: 'user' | 'admin' | 'barber';
  created_at?: string;
  updated_at?: string;
}
export interface LoginRequest {
  email: string;
  password: string;
}
export interface RegisterRequest {
  name: string;
  email: string;
  password: string;
  password_confirmation: string;
}
export interface ForgotPasswordRequest {
  email: string;
}
export interface ResetPasswordRequest {
  token: string;
  email: string;
  password: string;
  password_confirmation: string;
}
```
#### barber.model.ts
```typescript
export interface Barber {
  id: number;
  user_id: number | null;
  name: string;
  specialization: string | null;
  bio: string | null;
  photo_url: string | null;
  created_at?: string;
  updated_at?: string;
}
```
- `user_id` nullable: ha a borbélyhoz nincs hozzárendelt felhasználói fiók.
- `specialization`, `bio`, `photo_url` mind nullable, mivel opcionális profiladatok.
#### booking.model.ts
```typescript
export interface Booking {
  id: number;
  barber_id: number;
  user_id: number | null;
  customer_name: string;
  customer_email: string;
  customer_phone: string;
  start_at: string;
  duration_min: number;
  note: string | null;
  status: 'pending' | 'confirmed' | 'cancelled' | 'completed';
  barber?: Barber;
  created_at?: string;
  updated_at?: string;
}
export interface BookingRequest {
  barber_id: number;
  customer_name: string;
  customer_email: string;
  customer_phone: string;
  start_at: string;
  duration_min: number;
  note?: string;
}
```
- `status` union típus: a foglalás lehetséges állapotai.
- `barber?: Barber` – opcionálisan tartalmazza a kapcsolódó borbély adatait (when eager loaded).
- `user_id` nullable: vendég foglaláskor nincs regisztrált felhasználóhoz kötve.
- `BookingRequest` – az új foglalás létrehozásához szükséges adatok küldési típusa.
#### hairstyle.model.ts
```typescript
export interface Hairstyle {
  id: number;
  name: string;
  description: string | null;
  price_from: number | null;
  created_at?: string;
  updated_at?: string;
}
```
- `price_from` nullable: az ártól kategória megjelenítéséhez.
#### gallery.model.ts
```typescript
export interface GalleryImage {
  id: number;
  title: string | null;
  image_url: string;
  source: string | null;
  created_at?: string;
  updated_at?: string;
}
```
- `title` és `source` nullable, opcionális metaadatok.
#### index.ts (barrel export)
```typescript
export * from './barber.model';
export * from './booking.model';
export * from './hairstyle.model';
export * from './user.model';
export * from './gallery.model';
```
---
### Szolgáltatások (src/app/core/services/)
#### api.service.ts – Alap HTTP szolgáltatás
```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  protected baseUrl = environment.apiUrl;
  constructor(protected http: HttpClient) {}
  protected get<T>(path: string, params: any = {}): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}${path}`, { params });
  }
  protected post<T>(path: string, body: any = {}): Observable<T> {
    return this.http.post<T>(`${this.baseUrl}${path}`, body);
  }
  protected put<T>(path: string, body: any = {}): Observable<T> {
    return this.http.put<T>(`${this.baseUrl}${path}`, body);
  }
  protected delete<T>(path: string): Observable<T> {
    return this.http.delete<T>(`${this.baseUrl}${path}`);
  }
}
```
---
#### auth.service.ts – Autentikációs szolgáltatás
```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  public currentUser$ = this.currentUserSubject.asObservable();
  get currentUser(): User | null { return this.currentUserSubject.value; }
  get isLoggedIn(): boolean { return !!this.currentUserSubject.value; }
  get isAdmin(): boolean { return this.currentUserSubject.value?.role === 'admin'; }
  get isBarber(): boolean { return this.currentUserSubject.value?.role === 'barber'; }
  get token(): string | null { return localStorage.getItem('auth_token'); }
}
```
**Metódusok:**
| Metódus | HTTP | URL | Leírás |
|---|---|---|---|
| `register(data)` | POST | `/auth/register` | Regisztráció, visszaad egy üzenetet |
| `login(data)` | POST | `/auth/login` | Bejelentkezés, elmenti a tokent és a user adatait |
| `logout()` | POST | `/auth/logout` | Kijelentkezés, törli a tokent és a user state-et |
| `me()` | GET | `/auth/me` | Bejelentkezett user lekérése token alapján |
| `forgotPassword(data)` | POST | `/auth/forgot-password` | Jelszó visszaállító email küldése |
| `resetPassword(data)` | POST | `/auth/reset-password` | Jelszó visszaállítása tokennel |
| `resendVerification()` | POST | `/auth/email/resend` | Email megerősítő link újraküldése |
| `setUser(user)` | – | – | Manuálisan beállítja a current user state-et |
**Belső token kezelés:**
- `saveToken(token)` – `localStorage.setItem('auth_token', token)` hívja.
- `clearToken()` – `localStorage.removeItem('auth_token')` hívja.
- `login()` pipe-ban `tap` operátorral automatikusan menti a tokent és frissíti a `BehaviorSubject`-et.
- `me()` hiba esetén törli a tokent (érvénytelen/lejárt token esetén automatikus kijelentkezés).
---
#### barbers.service.ts – Borbélyok szolgáltatás
Örökli az `ApiService`-t.
| Metódus | HTTP | URL | Leírás |
|---|---|---|---|
| `getAll()` | GET | `/barbers` | Összes borbély listázása |
| `getById(id)` | GET | `/barbers/{id}` | Egy borbély adatainak lekérése |
| `getNextSlot(barberId)` | GET | `/barbers/{id}/next-slot` | Legkorábbi szabad időpont lekérése |
| `getSchedule(barberId, dateFrom, dateTo)` | GET | `/barbers/{id}/schedule` | Borbély naptárának lekérése dátumtartományban |
| `create(data)` | POST | `/barbers` | Új borbély létrehozása (admin) |
| `update(id, data)` | PUT | `/barbers/{id}` | Borbély adatainak módosítása (admin) |
| `remove(id)` | DELETE | `/barbers/{id}` | Borbély törlése (admin) |
---
#### bookings.service.ts – Foglalások szolgáltatás
Örökli az `ApiService`-t.
| Metódus | HTTP | URL | Leírás |
|---|---|---|---|
| `getAll()` | GET | `/bookings` | Összes foglalás listázása (bejelentkezett user saját foglalásai) |
| `getById(id)` | GET | `/bookings/{id}` | Egy foglalás adatai |
| `create(data)` | POST | `/bookings` | Új foglalás létrehozása |
| `update(id, data)` | PUT | `/bookings/{id}` | Foglalás frissítése (admin) |
| `remove(id)` | DELETE | `/bookings/{id}` | Foglalás törlése/lemondása |
| `getAvailability(barberId, dateFrom, dateTo)` | GET | `/availability` | Szabad időpontok lekérése |
---
#### barber-bookings.service.ts – Fodrász saját foglalásai
Örökli az `ApiService`-t. Kizárólag `barber` vagy `admin` role-lal rendelkező felhasználók számára.
| Metódus | HTTP | URL | Leírás |
|---|---|---|---|
| `getMyBarberProfile()` | GET | `/barber/me` | Bejelentkezett fodrász profil adatai |
| `getMyBookings()` | GET | `/barber/bookings` | Fodrász saját foglalásainak listája |
| `updateBooking(id, data)` | PUT | `/barber/bookings/{id}` | Foglalás státuszának módosítása |
| `cancelBooking(id)` | DELETE | `/barber/bookings/{id}` | Foglalás lemondása |
---
#### hairstyles.service.ts – Hajstílusok szolgáltatás
Örökli az `ApiService`-t.
| Metódus | HTTP | URL | Leírás |
|---|---|---|---|
| `getAll()` | GET | `/hairstyles` | Összes hajstílus listázása |
| `getById(id)` | GET | `/hairstyles/{id}` | Egy hajstílus adatai |
| `create(data)` | POST | `/hairstyles` | Új hajstílus létrehozása (admin) |
| `update(id, data)` | PUT | `/hairstyles/{id}` | Hajstílus módosítása (admin) |
| `remove(id)` | DELETE | `/hairstyles/{id}` | Hajstílus törlése (admin) |
---
#### gallery.service.ts – Galéria szolgáltatás
Örökli az `ApiService`-t.
| Metódus | HTTP | URL | Leírás |
|---|---|---|---|
| `getAll()` | GET | `/gallery` | Összes galéria kép listázása |
| `create(data)` | POST | `/gallery` | Új kép hozzáadása (admin) |
| `remove(id)` | DELETE | `/gallery/{id}` | Kép törlése (admin) |
---
#### users.service.ts – Felhasználókezelő szolgáltatás
Örökli az `ApiService`-t. Kizárólag `admin` role-lal rendelkező felhasználók számára.
| Metódus | HTTP | URL | Leírás |
|---|---|---|---|
| `getAll()` | GET | `/users` | Összes felhasználó listázása |
| `getById(id)` | GET | `/users/{id}` | Egy felhasználó adatai |
| `update(id, data)` | PUT | `/users/{id}` | Felhasználó adatainak módosítása (pl. role váltás) |
| `remove(id)` | DELETE | `/users/{id}` | Felhasználó törlése |
---
## Shared komponensek (src/app/shared/components/)
### NavbarComponent
**Fájlok:** `navbar.component.ts`, `navbar.component.html`, `navbar.component.scss`
**TypeScript:**
```typescript
export class NavbarComponent implements OnInit {
  isMenuOpen = false;
  constructor(public authService: AuthService, private router: Router) {}
  toggleMenu(): void { this.isMenuOpen = !this.isMenuOpen; }
  logout(): void {
    this.authService.logout().subscribe({
      next: () => this.router.navigate(['/']),
      error: () => this.router.navigate(['/'])
    });
  }
}
```
**Template főbb elemei:**
- Sötét, fix pozícionálású navigációs sáv (`navbar-dark bg-dark fixed-top`).
- Hamburger menü gomb mobilon, `(click)="toggleMenu()"` eseménykezelővel.
- `*ngIf="authService.isLoggedIn"` – bejelentkezett állapottól függő menüelemek.
  - Admin → „Admin" menüpont (`/admin`).
  - Borbély → „Foglalásaim" menüpont (`/barber-dashboard`).
  - Bejelentkezett user neve: `(authService.currentUser$ | async)?.name`.
  - „Kijelentkezés" gomb.
- Nem bejelentkezett állapotban: „Bejelentkezés" és „Regisztráció" gombok.
- `routerLinkActive="active"` jelöli az aktuálisan aktív menüpontot.
- A kijelentkezés hiba esetén is a főoldalra irányít (lokális state mindenképpen törlődik).
---
### FooterComponent
**Fájlok:** `footer.component.ts`, `footer.component.html`, `footer.component.scss`
```typescript
export class FooterComponent {
  currentYear = new Date().getFullYear();
}
```
**Template főbb elemei:**
- Háromoszlopos elrendezés: márka/leírás, navigáció linkek, kapcsolati adatok.
- `currentYear` dinamikusan adja az aktuális évet a copyright szöveghez.
- Kapcsolati adatok: cím, telefonszám, email.
- `mt-auto` osztálynak köszönhetően a lap aljára ragad (a `main` flex-column + min-vh-100 miatt).
---
### BarberCardComponent
**Fájlok:** `barber-card.component.ts`, `barber-card.component.html`, `barber-card.component.scss`
```typescript
export class BarberCardComponent {
  @Input() barber!: Barber;
  @Input() nextSlot: string | null = null;
}
```
**Bemeneti property-k:**
| Property | Típus | Kötelező | Leírás |
|---|---|---|---|
| `barber` | `Barber` | Igen | A megjelenítendő borbély adatai |
| `nextSlot` | `string \| null` | Nem | Legkorábbi szabad időpont ISO string formátumban |
**Template főbb elemei:**
- Kártya elrendezés (`card h-100 shadow-sm`).
- `*ngIf="barber.photo_url"` – ha van fotó, `<img>` megjelenik; ha nincs, Bootstrap ikon helyettesíti.
- Specializáció badge (`bi-star` ikonnal).
- Bio szöveg (`small` betűméretben).
- `nextSlot` megjelenítése: `{{ nextSlot | date:'yyyy.MM.dd HH:mm' }}` Angular `date` pipe-pal.
- „Időpontfoglalás" gomb: `[routerLink]="['/booking', barber.id]"`.
---
### ToastComponent
**Fájlok:** `toast.component.ts`, `toast.component.html`, `toast.component.scss`
```typescript
export interface ToastMessage {
  text: string;
  type: 'success' | 'error' | 'info' | 'warning';
}
export class ToastComponent {
  @Input() message: ToastMessage | null = null;
  @Output() dismissed = new EventEmitter<void>();
  get iconClass(): string { /* bi-check-circle-fill, bi-exclamation-triangle-fill stb. */ }
  get bgClass(): string { /* bg-success, bg-danger, bg-warning, bg-info */ }
  dismiss(): void { this.dismissed.emit(); }
}
```
**Bemeneti/kimeneti property-k:**
| Property | Típus | Irány | Leírás |
|---|---|---|---|
| `message` | `ToastMessage \| null` | `@Input` | Megjelenítendő üzenet és típusa |
| `dismissed` | `EventEmitter<void>` | `@Output` | Bezáráskor emitál a szülő komponensnek |
**Értesítés típusok:**
| Típus | Bootstrap osztály | Ikon |
|---|---|---|
| `success` | `bg-success` | `bi-check-circle-fill` |
| `error` | `bg-danger` | `bi-exclamation-triangle-fill` |
| `warning` | `bg-warning text-dark` | `bi-exclamation-circle-fill` |
| `info` | `bg-info` | `bi-info-circle-fill` |
- `position-fixed top-0 end-0` – jobb felső sarokba pozícionált, `z-index: 9999`.
- Bezárás gomb (`btn-close-white`) meghívja a `dismiss()` metódust.
---
## Oldalak (Pages)
### HomeComponent (src/app/pages/home/)
**TypeScript:**
```typescript
export class HomeComponent implements OnInit {
  barbers: Barber[] = [];
  nextSlots: { [barberId: number]: string } = {};
  loading = true;
  ngOnInit(): void {
    this.barbersService.getAll().subscribe({
      next: (barbers) => {
        this.barbers = barbers;
        this.loading = false;
        barbers.forEach(b => {
          this.barbersService.getNextSlot(b.id).subscribe({
            next: (res) => { this.nextSlots[b.id] = res.next_slot; }
          });
        });
      },
      error: () => { this.loading = false; }
    });
  }
}
```
**Szekciók (template):**
1. **Hero szekció** – teljes szélességű banner, „BarberShop" felirattal, „Időpontfoglalás" CTA gombbal. A gomb `routerLink="/" fragment="barbers"` segítségével az oldalon belül a borbélyok szekcióhoz gördül.
2. **SOS Foglalás szekció** – „Legkorábbi szabad időpont" gomb, amely az első borbélyhoz navigál (`[routerLink]="['/booking', barbers[0].id]"`). Csak akkor jelenik meg, ha van legalább egy borbély (`*ngIf="barbers.length > 0"`).
3. **Borbélyok szekció** (`id="barbers"`) – `*ngIf="loading"` spinner, majd rácsos elrendezésű `<app-barber-card>` komponensek. Üres állapot kezelés: ha nincs borbély, szomorú ikon és üzenet jelenik meg.
4. **„Miért minket válassz?" szekció** – sötét háttérrel, háromoszlopos feature lista (Online foglalás, E-mail értesítések, Prémium minőség).
**Adat flow:** `BarbersService.getAll()` → borbélyok betöltése → minden borbélyhoz párhuzamos `getNextSlot()` hívás → `nextSlots` objektum feltöltése → kártyák `nextSlot` inputjának beállítása.
---
### BookingComponent (src/app/pages/booking/)
**TypeScript:**
```typescript
export class BookingComponent implements OnInit {
  barber: Barber | null = null;
  hairstyles: Hairstyle[] = [];
  customerName = '';
  customerEmail = '';
  customerPhone = '';
  startAt = '';
  durationMin = 30;
  note = '';
  selectedHairstyle: number | null = null;
  loading = false;
  loadingBarber = true;
  errorMessage = '';
  errors: any = {};
}
```
**`ngOnInit` logika:**
1. A route paraméterből kinyeri a `barberId`-t: `this.route.snapshot.paramMap.get('barberId')`.
2. `BarbersService.getById(barberId)` – betölti a borbély adatait.
3. `HairstylesService.getAll()` – betölti az elérhető hajstílusokat a legördülő menühöz.
**`onSubmit` logika:**
1. Dátum formátum ellenőrzés: a `datetime-local` input `HH:MM` formátumot ad, de a backend `Y-m-dTH:i:s` formátumot vár, ezért `:00` másodpercet fűz hozzá ha szükséges.
2. `BookingsService.create({...})` hívása az összegyűjtött adatokkal.
3. Siker: navigálás `/booking-success` oldalra, Router Navigation State-en keresztül adja át a foglalás és borbély adatait.
4. Hiba: `errorMessage` és `errors` (mezőnkénti validációs hibák) feltöltése.
**Template főbb elemei:**
- Spinner betöltés közben.
- Borbély fejléc: profilkép (vagy ikon), neve, specializáció badge.
- Foglalási űrlap mezők:
  - Teljes név (`customer_name`) – validációs hiba megjelenítéssel.
  - E-mail cím (`customer_email`).
  - Telefonszám (`customer_phone`).
  - Időpont (`datetime-local` input).
  - Időtartam legördülő (30 / 45 / 60 perc).
  - Hajstílus legördülő (opcionális, csak ha vannak stílusok).
  - Megjegyzés textarea (opcionális).
- „Foglalás véglegesítése" gomb spinnerrel és disabled állapottal.
- Mezőnkénti validációs hibaüzenetek: `[class.is-invalid]="errors['mező_neve']"`.
---
### BookingSuccessComponent (src/app/pages/booking-success/)
**TypeScript:**
```typescript
export class BookingSuccessComponent implements OnInit {
  booking: any = null;
  barber: any = null;
  constructor(private router: Router) {
    const nav = this.router.getCurrentNavigation();
    if (nav?.extras.state) {
      this.booking = nav.extras.state['booking'];
      this.barber = nav.extras.state['barber'];
    }
  }
  ngOnInit(): void {
    if (!this.booking) { this.router.navigate(['/']); }
  }
}
```
- Az adatokat a Router Navigation State-ből veszi (a `BookingComponent` adja át `state` objektumban).
- Ha nincs foglalási adat (pl. direkt URL-ről jött a felhasználó), főoldalra irányít.
**Template főbb elemei:**
- Sikeres visszaigazoló kártya: nagy zöld pipa ikon.
- Visszaigazoló email cím megjelenítése.
- Foglalás részletei táblázat: borbély neve, időpont (`date` pipe), időtartam, ügyfél neve, státusz badge.
- „Vissza a főoldalra" gomb.
---
### GalleryComponent (src/app/pages/gallery/)
**TypeScript:**
```typescript
export class GalleryComponent implements OnInit {
  images: GalleryImage[] = [];
  loading = true;
  selectedImage: GalleryImage | null = null;
  ngOnInit(): void {
    this.galleryService.getAll().subscribe({
      next: (images) => { this.images = images; this.loading = false; },
      error: () => { this.loading = false; }
    });
  }
  openImage(image: GalleryImage): void { this.selectedImage = image; }
  closeImage(): void { this.selectedImage = null; }
}
```
**Template főbb elemei:**
- Rácsos elrendezés (`col-6 col-md-4 col-lg-3`) – reszponzív képrács.
- Minden képnél hover-effektes `.gallery-overlay` (cím megjelenítéssel).
- Forrás (`source`) megjelenítése a kép alatt, ha van.
- Üres állapot kezelés.
**Lightbox:**
- `*ngIf="selectedImage"` – sötét félopaque overlay réteg.
- Kép és metadata (cím, forrás) megjelenítése.
- Bezárás: az overlay-re kattintva (`(click)="closeImage()"`) vagy az X gombbal.
- `(click)="$event.stopPropagation()"` a lightbox-content elemen, hogy a képre kattintás ne csukja be az ablakot.
---
### AboutComponent (src/app/pages/about/)
Egyszerű, statikus tartalmú prezentációs oldal.
**Template szekciók:**
1. **Bemutatkozó szekció** – kétoszlopos elrendezés: balra placeholder-kép (sötét div BarberShop szöveggel és scissors ikonnal), jobbra szöveg a szalonról + „Foglalj időpontot" CTA gomb.
2. **Szolgáltatások szekció** – háromoszlopos kártyák (Hajvágás, Szakálligazítás, Prémium csomag) Bootstrap ikonokkal.
3. **Kreditek szekció** – galéria képek forrásának megjelölése (Csukodi Zoltán), vizsgaremek projekt megjegyzés.
---
### Auth oldalak (src/app/pages/auth/)
Minden auth oldal standalone komponens. Az auth oldalakon a navbar és footer el van rejtve (`AppComponent.showLayout = false`).
#### LoginComponent
**Működés:**
1. Template-driven form: `[(ngModel)]` kétirányú adatkötés `email` és `password` mezőkre.
2. `onSubmit()` meghívja `AuthService.login()`.
3. Siker: navigálás főoldalra (`/`).
4. Hiba: `errorMessage` megjelenítése.
**Template:** Kártyás elrendezés, scissors ikon, email + jelszó input, „Elfelejtett jelszó?" link, bejelentkezés gomb, regisztrációra mutató link, „Tovább vendégként" link.
---
#### RegisterComponent
**Működés:**
1. Négy mező: `name`, `email`, `password`, `password_confirmation`.
2. `onSubmit()` meghívja `AuthService.register()`.
3. Siker: `successMessage` megjelenítése (az űrlap eltűnik, megjelenik a „Bejelentkezés" gomb).
4. Hiba: általános `errorMessage` + mezőnkénti `errors` objektum megjelenítése.
**Template:** Kártyás elrendezés, input validációs hibaüzenetek minden mezőnél, siker üzenet login gombbal.
---
#### ForgotPasswordComponent
**Működés:**
1. Egy mező: `email`.
2. `onSubmit()` meghívja `AuthService.forgotPassword()`.
3. Siker: `successMessage` megjelenítése (backend elküldi a reset emailt).
4. Hiba: `errorMessage` megjelenítése.
---
#### ResetPasswordComponent
**Működés:**
1. `ngOnInit()`: kinyeri a `token` és `email` query paramétereket az URL-ből (ezeket a backend által küldött email tartalmazza).
2. Mezők: `password`, `password_confirmation`.
3. `onSubmit()` meghívja `AuthService.resetPassword()` a tokennel, emailel és az új jelszóval.
4. Siker: `successMessage` megjelenítése.
5. Hiba: `errorMessage` megjelenítése.
---
### AdminComponent (src/app/pages/admin/)
**Guard:** `adminGuard` – kizárólag `admin` role-lal rendelkező felhasználók érhetik el.
**TypeScript:**
```typescript
export class AdminComponent implements OnInit {
  activeTab = 'barbers';
  barbers: Barber[] = [];
  bookings: Booking[] = [];
  hairstyles: Hairstyle[] = [];
  galleryImages: GalleryImage[] = [];
  users: User[] = [];
  editingUser: User | null = null;
  editUserRole: string = 'user';
}
```
**Tabfülek és funkcióik:**
| Tab neve | Tartalom | Műveletek |
|---|---|---|
| `barbers` | Borbélyok táblázata | Törlés (`deleteBarber`) |
| `bookings` | Foglalások táblázata | Lemondás (`deleteBooking`) – ha nem lemondott |
| `hairstyles` | Hajstílusok táblázata | Törlés (`deleteHairstyle`) |
| `gallery` | Galéria képek kártyás nézetben | Törlés (`deleteGalleryImage`) |
| `users` | Felhasználók táblázata | Szerkesztés (role váltás), Törlés |
**Felhasználó szerkesztés inline logika:**
- `startEditUser(user)` – másolatot készít a userről, beállítja az `editingUser`-t.
- Szerkesztés közben a role oszlopban `<select>` legördülő jelenik meg (Felhasználó / Borbély / Admin).
- `saveUserRole()` – `UsersService.update(id, { role })` hívása, majd lista újratöltése.
- `cancelEditUser()` – `editingUser = null`, az eredeti táblázatos nézet visszaáll.
**`getStatusBadgeClass()` és `getStatusLabel()`:** Megegyezik a `BarberDashboardComponent`-ben lévő implementációval.
**Template – felhasználók tab részlet:**
- Megerősített email ikon: `bi-check-circle-fill text-success` vagy `bi-x-circle-fill text-danger`.
- Role badge: `bg-secondary` (user), `bg-primary` (barber), `bg-danger` (admin).
- Inline szerkesztés váltás `ng-container` és `ng-template` (`#showRole`, `#showActions`) segítségével.
---
## Tesztelés
```bash
ng test
```
