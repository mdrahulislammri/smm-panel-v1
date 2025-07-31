# smm-panel-v1


SMM প্যানেল তৈরির ফুল গাইড (কোড + ডিজাইন)
১. প্রাথমিক পরিকল্পনা

টার্গেট ফিচার:
ইউজার ম্যানেজমেন্ট: রেজিস্ট্রেশন, লগইন, প্রোফাইল আপডেট।
সার্ভিস ম্যানেজমেন্ট: সোশ্যাল মিডিয়া সার্ভিস (ইনস্টাগ্রাম ফলোয়ার, ইউটিউব ভিউ)।
অর্ডার সিস্টেম: অর্ডার প্লেস, ট্র্যাকিং।
পেমেন্ট সিস্টেম: Stripe, PayPal, বা বিকাশ।
অ্যানালিটিক্স: অর্ডার পারফরম্যান্স ট্র্যাকিং।
অ্যাডমিন প্যানেল: সবকিছু ম্যানেজ করার জন্য।
সাপোর্ট সিস্টেম: টিকিট সিস্টেম।


ডিজাইন গোল:
মডার্ন, ক্লিন, এবং রেসপনসিভ ডিজাইন।
কালার স্কিম: নীল, সাদা, এবং অরেঞ্জ (তুমি চাইলে কাস্টমাইজ করতে পারো)।
ইউজার-ফ্রেন্ডলি UI (বাটন, ফর্ম, টেবিল)।


টেকনোলজি স্ট্যাক:
Backend: Laravel (PHP 8.1+)
Frontend: Tailwind CSS, JavaScript (Alpine.js বা Vue.js)
Database: MySQL
API: SMMValy (SMM API), Stripe (পেমেন্ট)
Tools: VS Code, XAMPP, GitHub, Postman



২. প্রস্তুতি
২.১ লোকাল সেটআপ

XAMPP ইনস্টল:
XAMPP ডাউনলোড করে Apache এবং MySQL চালু করো।


VS Code ইনস্টল: ডাউনলোড।
Composer ইনস্টল: ডাউনলোড।
Git ইনস্টল: ডাউনলোড এবং GitHub-এ রিপোজিটরি তৈরি করো।
SMM API অ্যাকাউন্ট: SMMValy-তে সাইনআপ করো এবং API কী নাও।
Stripe অ্যাকাউন্ট: Stripe এ সাইনআপ করো এবং API কী নাও।

২.২ ডোমেইন এবং হোস্টিং

ডোমেইন: Namecheap থেকে কিনো (৮০০-১২০০ টাকা/বছর)।
হোস্টিং: প্রথমে লোকালে কাজ করো। পরে DigitalOcean VPS নাও (৫০০-২০০০ টাকা/মাস)।
SSL: Let’s Encrypt (ফ্রি)।

৩. Laravel প্রজেক্ট সেটআপ

Laravel ইনস্টল:composer create-project laravel/laravel smm-panel
cd smm-panel
php artisan serve


ব্রাউজারে http://localhost:8000 খোলো।


MySQL ডাটাবেস:
phpMyAdmin-এ smm_panel_db নামে ডাটাবেস তৈরি করো।
.env ফাইল এডিট করো:DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=smm_panel_db
DB_USERNAME=root
DB_PASSWORD=




Tailwind CSS ইনস্টল:npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p


tailwind.config.js:module.exports = {
    content: [
        './resources/**/*.blade.php',
        './resources/**/*.js',
        './resources/**/*.vue',
    ],
    theme: { extend: {} },
    plugins: [],
}


resources/css/app.css:@tailwind base;
@tailwind components;
@tailwind utilities;


npm run dev চালাও।


Laravel Authentication:composer require laravel/breeze --dev
php artisan breeze:install blade
npm install && npm run dev
php artisan migrate


এটা রেজিস্ট্রেশন, লগইন, এবং ড্যাশবোর্ড তৈরি করে দেবে।



৪. ডাটাবেস ডিজাইন
৪.১ মাইগ্রেশন ফাইল

users:// database/migrations/xxxx_create_users_table.php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password');
    $table->decimal('balance', 8, 2)->default(0);
    $table->string('role')->default('user'); // user, admin
    $table->timestamps();
});


services:// database/migrations/xxxx_create_services_table.php
Schema::create('services', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('category');
    $table->decimal('price', 8, 2);
    $table->string('api_id');
    $table->text('description')->nullable();
    $table->string('status')->default('active');
    $table->timestamps();
});


orders:// database/migrations/xxxx_create_orders_table.php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained();
    $table->foreignId('service_id')->constrained();
    $table->string('link');
    $table->integer('quantity');
    $table->decimal('price', 8, 2);
    $table->string('status')->default('pending');
    $table->timestamps();
});


transactions:// database/migrations/xxxx_create_transactions_table.php
Schema::create('transactions', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained();
    $table->decimal('amount', 8, 2);
    $table->string('payment_method');
    $table->string('status')->default('pending');
    $table->timestamps();
});


মাইগ্রেশন চালাও:php artisan migrate



৫. Backend ডেভেলপমেন্ট
৫.১ মডেল তৈরি

Service:// app/Models/Service.php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
class Service extends Model {
    protected $fillable = ['name', 'category', 'price', 'api_id', 'description', 'status'];
}


Order:// app/Models/Order.php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
class Order extends Model {
    protected $fillable = ['user_id', 'service_id', 'link', 'quantity', 'price', 'status'];
}


Transaction:// app/Models/Transaction.php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
class Transaction extends Model {
    protected $fillable = ['user_id', 'amount', 'payment_method', 'status'];
}



৫.২ কন্ট্রোলার

ServiceController:// app/Http/Controllers/ServiceController.php
namespace App\Http\Controllers;
use App\Models\Service;
use Illuminate\Support\Facades\Http;
class ServiceController extends Controller {
    public function index() {
        $services = Service::where('status', 'active')->get();
        return view('services.index', compact('services'));
    }
    public function fetchServices() {
        $apiKey = env('SMM_API_KEY');
        $url = 'https://api.smmprovider.com/v1/services';
        $response = Http::get($url, ['key' => $apiKey]);
        if ($response->successful()) {
            foreach ($response->json() as $service) {
                Service::updateOrCreate(
                    ['api_id' => $service['service_id']],
                    ['name' => $service['name'], 'category' => $service['category'], 'price' => $service['rate']]
                );
            }
            return redirect()->route('services.index')->with('success', 'Services updated!');
        }
        return back()->with('error', 'API error!');
    }
}


OrderController:// app/Http/Controllers/OrderController.php
namespace App\Http\Controllers;
use App\Models\Service;
use App\Models\Order;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Http;
class OrderController extends Controller {
    public function create(Service $service) {
        return view('orders.create', compact('service'));
    }
    public function store(Request $request) {
        $validated = $request->validate([
            'service_id' => 'required|exists:services,id',
            'link' => 'required|url',
            'quantity' => 'required|integer|min:1',
        ]);
        $service = Service::find($request->service_id);
        $price = $service->price * $request->quantity;
        if (Auth::user()->balance < $price) {
            return back()->with('error', 'Insufficient balance!');
        }
        $apiKey = env('SMM_API_KEY');
        $url = 'https://api.smmprovider.com/v1/order';
        $response = Http::post($url, [
            'key' => $apiKey,
            'action' => 'add',
            'service' => $service->api_id,
            'link' => $request->link,
            'quantity' => $request->quantity,
        ]);
        if ($response->successful()) {
            $order = Order::create([
                'user_id' => Auth::id(),
                'service_id' => $service->id,
                'link' => $request->link,
                'quantity' => $request->quantity,
                'price' => $price,
                'status' => 'pending',
            ]);
            Auth::user()->decrement('balance', $price);
            return redirect()->route('orders.index')->with('success', 'Order placed!');
        }
        return back()->with('error', 'API error!');
    }
    public function index() {
        $orders = Order::where('user_id', Auth::id())->get();
        return view('orders.index', compact('orders'));
    }
}


PaymentController:// app/Http/Controllers/PaymentController.php
namespace App\Http\Controllers;
use App\Models\Transaction;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Stripe\Charge;
use Stripe\Stripe;
class PaymentController extends Controller {
    public function deposit(Request $request) {
        $validated = $request->validate(['amount' => 'required|numeric|min:1']);
        Stripe::setApiKey(env('STRIPE_SECRET'));
        try {
            $charge = Charge::create([
                'amount' => $request->amount * 100,
                'currency' => 'usd',
                'source' => $request->stripeToken,
                'description' => 'Deposit for SMM Panel',
            ]);
            Auth::user()->increment('balance', $request->amount);
            Transaction::create([
                'user_id' => Auth::id(),
                'amount' => $request->amount,
                'payment_method' => 'stripe',
                'status' => 'completed',
            ]);
            return redirect()->route('dashboard')->with('success', 'Deposit successful!');
        } catch (\Exception $e) {
            return back()->with('error', $e->getMessage());
        }
    }
}



৫.৩ রাউট

routes/web.php:use App\Http\Controllers\ServiceController;
use App\Http\Controllers\OrderController;
use App\Http\Controllers\PaymentController;
Route::get('/', function () { return view('welcome'); });
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', function () { return view('dashboard'); })->name('dashboard');
    Route::get('/services', [ServiceController::class, 'index'])->name('services.index');
    Route::get('/services/fetch', [ServiceController::class, 'fetchServices'])->name('services.fetch');
    Route::get('/orders/create/{service}', [OrderController::class, 'create'])->name('orders.create');
    Route::post('/orders', [OrderController::class, 'store'])->name('orders.store');
    Route::get('/orders', [OrderController::class, 'index'])->name('orders.index');
    Route::post('/deposit', [PaymentController::class, 'deposit'])->name('deposit');
});
require __DIR__.'/auth.php';



৬. Frontend ডেভেলপমেন্ট (সুন্দর ডিজাইন)
৬.১ লেআউট

resources/views/layouts/app.blade.php:<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>SMM Panel</title>
    <link href="{{ asset('css/app.css') }}" rel="stylesheet">
    <script src="https://js.stripe.com/v3/"></script>
</head>
<body class="bg-gray-100 font-sans">
    <nav class="bg-blue-600 text-white p-4">
        <div class="container mx-auto flex justify-between">
            <a href="/" class="text-xl font-bold">SMM Panel</a>
            <div>
                @auth
                    <a href="{{ route('dashboard') }}" class="px-4">Dashboard</a>
                    <a href="{{ route('services.index') }}" class="px-4">Services</a>
                    <a href="{{ route('orders.index') }}" class="px-4">Orders</a>
                    <a href="{{ route('logout') }}" class="px-4">Logout</a>
                @else
                    <a href="{{ route('login') }}" class="px-4">Login</a>
                    <a href="{{ route('register') }}" class="px-4">Register</a>
                @endauth
            </div>
        </div>
    </nav>
    <main class="container mx-auto py-6">
        @yield('content')
    </main>
    <script src="{{ asset('js/app.js') }}"></script>
</body>
</html>



৬.২ ড্যাশবোর্ড

resources/views/dashboard.blade.php:@extends('layouts.app')
@section('content')
    <div class="bg-white p-6 rounded-lg shadow-md">
        <h1 class="text-2xl font-bold mb-4">Welcome, {{ Auth::user()->name }}</h1>
        <p class="text-lg">Balance: ${{ Auth::user()->balance }}</p>
        <a href="{{ route('services.index') }}" class="mt-4 inline-block bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">View Services</a>
        <!-- Deposit Form -->
        <h2 class="text-xl mt-6">Deposit Funds</h2>
        <form id="payment-form" action="{{ route('deposit') }}" method="POST" class="mt-4">
            @csrf
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700">Amount (USD)</label>
                <input type="number" name="amount" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm" required>
            </div>
            <div id="card-element" class="p-4 border rounded-md"></div>
            <button type="submit" class="mt-4 bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700">Deposit</button>
        </form>
        <script>
            var stripe = Stripe('{{ env('STRIPE_KEY') }}');
            var elements = stripe.elements();
            var card = elements.create('card');
            card.mount('#card-element');
            var form = document.getElementById('payment-form');
            form.addEventListener('submit', function(event) {
                event.preventDefault();
                stripe.createToken(card).then(function(result) {
                    if (result.error) {
                        alert(result.error.message);
                    } else {
                        var tokenInput = document.createElement('input');
                        tokenInput.setAttribute('type', 'hidden');
                        tokenInput.setAttribute('name', 'stripeToken');
                        tokenInput.setAttribute('value', result.token.id);
                        form.appendChild(tokenInput);
                        form.submit();
                    }
                });
            });
        </script>
    </div>
    <!-- Analytics Chart -->
    <div class="bg-white p-6 rounded-lg shadow-md mt-6">
        <h2 class="text-xl font-bold mb-4">Order Analytics</h2>
        <canvas id="orderChart"></canvas>
        <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
        <script>
            const ctx = document.getElementById('orderChart').getContext('2d');
            new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Jan', 'Feb', 'Mar', 'Apr'],
                    datasets: [{
                        label: 'Orders',
                        data: [10, 20, 15, 30],
                        borderColor: '#2563eb',
                        fill: false
                    }]
                }
            });
        </script>
    </div>
@endsection



৬.৩ সার্ভিস পেজ

resources/views/services/index.blade.php:@extends('layouts.app')
@section('content')
    <div class="bg-white p-6 rounded-lg shadow-md">
        <h1 class="text-2xl font-bold mb-4">Services</h1>
        @if (session('success'))
            <div class="bg-green-100 text-green-700 p-4 rounded mb-4">{{ session('success') }}</div>
        @endif
        <a href="{{ route('services.fetch') }}" class="mb-4 inline-block bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">Update Services</a>
        <div class="overflow-x-auto">
            <table class="w-full table-auto">
                <thead>
                    <tr class="bg-gray-200">
                        <th class="px-4 py-2">Name</th>
                        <th class="px-4 py-2">Category</th>
                        <th class="px-4 py-2">Price</th>
                        <th class="px-4 py-2">Action</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach ($services as $service)
                        <tr class="border-b">
                            <td class="px-4 py-2">{{ $service->name }}</td>
                            <td class="px-4 py-2">{{ $service->category }}</td>
                            <td class="px-4 py-2">${{ $service->price }}</td>
                            <td class="px-4 py-2">
                                <a href="{{ route('orders.create', $service->id) }}" class="bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700">Order</a>
                            </td>
                        </tr>
                    @endforeach
                </tbody>
            </table>
        </div>
    </div>
@endsection



৬.৪ অর্ডার ফর্ম

resources/views/orders/create.blade.php:@extends('layouts.app')
@section('content')
    <div class="bg-white p-6 rounded-lg shadow-md">
        <h1 class="text-2xl font-bold mb-4">Place Order: {{ $service->name }}</h1>
        @if (session('error'))
            <div class="bg-red-100 text-red-700 p-4 rounded mb-4">{{ session('error') }}</div>
        @endif
        <form method="POST" action="{{ route('orders.store') }}" class="space-y-4">
            @csrf
            <input type="hidden" name="service_id" value="{{ $service->id }}">
            <div>
                <label class="block text-sm font-medium text-gray-700">Link</label>
                <input type="url" name="link" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm" required>
            </div>
            <div>
                <label class="block text-sm font-medium text-gray-700">Quantity</label>
                <input type="number" name="quantity" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm" required min="1">
            </div>
            <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">Place Order</button>
        </form>
    </div>
@endsection



৬.৫ অর্ডার লিস্ট

resources/views/orders/index.blade.php:@extends('layouts.app')
@section('content')
    <div class="bg-white p-6 rounded-lg shadow-md">
        <h1 class="text-2xl font-bold mb-4">Your Orders</h1>
        @if (session('success'))
            <div class="bg-green-100 text-green-700 p-4 rounded mb-4">{{ session('success') }}</div>
        @endif
        <div class="overflow-x-auto">
            <table class="w-full table-auto">
                <thead>
                    <tr class="bg-gray-200">
                        <th class="px-4 py-2">Service</th>
                        <th class="px-4 py-2">Link</th>
                        <th class="px-4 py-2">Quantity</th>
                        <th class="px-4 py-2">Price</th>
                        <th class="px-4 py-2">Status</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach ($orders as $order)
                        <tr class="border-b">
                            <td class="px-4 py-2">{{ $order->service->name }}</td>
                            <td class="px-4 py-2">{{ $order->link }}</td>
                            <td class="px-4 py-2">{{ $order->quantity }}</td>
                            <td class="px-4 py-2">${{ $order->price }}</td>
                            <td class="px-4 py-2">{{ ucfirst($order->status) }}</td>
                        </tr>
                    @endforeach
                </tbody>
            </table>
        </div>
    </div>
@endsection



৭. ডিজাইন কাস্টমাইজেশন
তুমি যেহেতু সুন্দর ডিজাইন চাও, নিচে কিছু টিপস এবং কাস্টমাইজেশন আইডিয়া দিচ্ছি:

কালার স্কিম:
প্রাইমারি: #2563eb (নীল)
সেকেন্ডারি: #f97316 (অরেঞ্জ)
ব্যাকগ্রাউন্ড: #f3f4f6 (হালকা ধূসর)
resources/css/app.css-এ যোগ করো::root {
    --primary: #2563eb;
    --secondary: #f97316;
    --background: #f3f4f6;
}
.bg-primary { background-color: var(--primary); }
.text-primary { color: var(--primary); }
.bg-secondary { background-color: var(--secondary); }
.text-secondary { color: var(--secondary); }




ফন্ট:
Google Fonts থেকে Inter বা Poppins ব্যবহার করো:<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">


resources/css/app.css:body { font-family: 'Inter', sans-serif; }






কাস্টম UI:
বাটনে হোভার ইফেক্ট:button:hover { transform: scale(1.05); transition: transform 0.2s; }


কার্ডে শ্যাডো এবং বর্ডার:.card { @apply bg-white p-6 rounded-lg shadow-md; }




লোগো:
তুমি আগে SMM BAZAR-এর জন্য লোগো চেয়েছিলে (200x80px, নীল-অরেঞ্জ)। Fiverr থেকে লোগো তৈরি করো বা আমাকে বলো, আমি প্রম্পট দিয়ে হেল্প করব।


আইকন:
Font Awesome বা Heroicons ব্যবহার করো:<script src="https://kit.fontawesome.com/your-kit-id.js"></script>


উদাহরণ: <i class="fas fa-user"></i>





৮. টেস্টিং

লোকাল টেস্টিং:
php artisan serve চালাও।
রেজিস্ট্রেশন, লগইন, সার্ভিস লিস্ট, অর্ডার ফর্ম, এবং পেমেন্ট টেস্ট করো।
Postman দিয়ে SMM API টেস্ট করো।


সিকিউরিটি টেস্ট:
ইনপুট ভ্যালিডেশন চেক করো।
CSRF টোকেন কাজ করছে কিনা দেখো।


ডিজাইন টেস্ট:
মোবাইল এবং ডেস্কটপে রেসপনসিভনেস চেক করো।
বাটন এবং ফর্মের ইউজার এক্সপেরিয়েন্স টেস্ট করো।



৯. ডিপ্লয়মেন্ট

DigitalOcean VPS:
Ubuntu 20.04 ইনস্টল করো।
Nginx, PHP, MySQL ইনস্টল:sudo apt update
sudo apt install nginx php-fpm php-mysql mysql-server




কোড ডিপ্লয়:
GitHub থেকে:git clone your-repo-url /var/www/smm-panel
cd /var/www/smm-panel
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate




Nginx কনফিগ:
/etc/nginx/sites-available/smm-panel:server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/smm-panel/public;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}


sudo ln -s /etc/nginx/sites-available/smm-panel /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx


SSL:sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com



১০. সিকিউরিটি এবং মেইনটেন্যান্স

সিকিউরিটি:
.env ফাইলে API কী রাখো:SMM_API_KEY=your_smm_api_key
STRIPE_KEY=pk_test_your_key
STRIPE_SECRET=sk_test_your_key


Rate limiting:// app/Http/Kernel.php
'api' => [
    'throttle:60,1',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],




মেইনটেন্যান্স:
ডাটাবেস ব্যাকআপ:mysqldump -u root -p smm_panel_db > backup.sql


Dependencies আপডেট:composer update
npm update





১১. খরচ

ডোমেইন: ৮০০-১২০০ টাকা/বছর
VPS: ৫০০-২০০০ টাকা/মাস
SMM API: অর্ডার প্রতি ১-৫০০ টাকা
পেমেন্ট গেটওয়ে: ২-৫% ফি
লোগো: ৫০০-৫০০০ টাকা (Fiverr)
