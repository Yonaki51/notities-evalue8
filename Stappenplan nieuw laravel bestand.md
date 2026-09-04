# Stappenplan: Nieuwe pagina maken in Laravel

## Stap 1 — Bepaal wat de pagina moet doen

Voordat je begint, bedenk kort:

- Wat moet er op de pagina komen?
    
- Welke data heb ik nodig?
    
- Bestaat er al een controller of model die hierbij hoort?
    
- Is er een vergelijkbare pagina in het project?
    

> **Tip:** Kijk vooral naar een bestaande pagina die ongeveer hetzelfde doet. In een bestaand project is het meestal slim om dezelfde structuur aan te houden.

---

## Stap 2 — Maak de route

Open:

`routes/web.php`

Maak een route voor de nieuwe pagina:

```php
Route::get('/orders', [OrderController::class, 'index'])
    ->name('orders.index');
```

Hiermee zeg je eigenlijk:

> Als iemand naar `/orders` gaat, voer dan de `index()` methode van `OrderController` uit.

---

## Stap 3 — Maak of gebruik een Controller

Controleer eerst of er al een geschikte controller bestaat.

Zo niet, maak er één:

```bash
php artisan make:controller OrderController
```

De controller komt vervolgens in:

`app/Http/Controllers/OrderController.php`

---

## Stap 4 — Maak de Controller-methode

Maak in de controller de methode die je bij de route hebt opgegeven:

```php
public function index()
{
    return view('orders.index');
}
```

`orders.index` verwijst naar:

`resources/views/orders/index.blade.php`

De `.` staat hierbij eigenlijk voor een map.

Dus:

```text
orders.index
```

wordt:

```text
orders/index.blade.php
```

---

## Stap 5 — Haal eventueel data op

Als de pagina gegevens uit de database nodig heeft, haal je deze meestal op in de controller.

Bijvoorbeeld:

```php
public function index()
{
    $orders = Order::all();

    return view('orders.index', compact('orders'));
}
```

Hier gebeurt het volgende:

1. `Order::all()` haalt alle orders op.
    
2. Deze worden opgeslagen in `$orders`.
    
3. Met `compact('orders')` wordt `$orders` beschikbaar gemaakt in de Blade-view.
    

Je hoeft niet voor iedere nieuwe pagina een nieuw Model te maken. Als de pagina bestaande gegevens gebruikt, gebruik je meestal gewoon het bestaande Model.

---

## Stap 6 — Maak de Blade-view

Maak het bestand:

`resources/views/orders/index.blade.php`

Gebruik dezelfde layout als vergelijkbare pagina's in het project.

Bijvoorbeeld:

```blade
@extends('layouts.app')

@section('content')
    <h1>Orders</h1>
@endsection
```

---

## Stap 7 — Toon de data in Blade

Als je vanuit de controller `$orders` hebt meegegeven, kun je deze in Blade gebruiken:

```blade
@foreach ($orders as $order)
    <p>{{ $order->name }}</p>
@endforeach
```

Laravel loopt hier door alle orders heen en toont voor iedere order de `name`.

---

## Stap 8 — Voeg eventuele vertalingen toe

Als het project gebruikmaakt van Laravel translations, kom je bijvoorbeeld dit tegen:

```blade
{{ __('orders.title') }}
```

Voeg de bijbehorende tekst dan toe aan het juiste translation-bestand.

Gebruik hiervoor dezelfde structuur als bestaande pagina's in het project.

---

## Stap 9 — Voeg de pagina toe aan de navigatie

Als gebruikers via een menu of knop naar de pagina moeten kunnen gaan, voeg je daar een link toe.

Gebruik bij voorkeur de naam van de route:

```blade
<a href="{{ route('orders.index') }}">
    Orders
</a>
```

Dit is beter dan rechtstreeks `/orders` invullen, omdat Laravel dan zelf de URL van de route bepaalt.

---

## Stap 10 — Controleer permissions en middleware

Bij een bestaande applicatie mag niet iedere gebruiker automatisch iedere pagina zien.

Controleer daarom vergelijkbare routes en controllers op bijvoorbeeld:

- Middleware
    
- Roles
    
- Permissions
    
- Policies
    

Neem dezelfde aanpak over als dat voor jouw pagina van toepassing is.

---

## Stap 11 — Test de pagina

Open de pagina in de browser en controleer:

- Werkt de URL?
    
- Wordt de juiste pagina geladen?
    
- Wordt de juiste data getoond?
    
- Werkt de pagina als er geen data is?
    
- Werken knoppen en links?
    
- Kloppen de permissions?
    
- Zijn er errors in de browserconsole?
    

---

## Stap 12 — Controleer je code

Formatteer je PHP-code bijvoorbeeld met Pint:

```bash
./vendor/bin/pint
```

Bekijk daarna wat je daadwerkelijk hebt aangepast:

```bash
git diff
git status
```

Als alles goed werkt, kun je de wijzigingen committen.

---

# Kort onthouden

De belangrijkste volgorde is:

**Route → Controller → Model/Data → Blade → Navigatie → Testen**

Of iets uitgebreider:

```text
1. Bepalen wat de pagina nodig heeft
        ↓
2. Route maken
        ↓
3. Controller maken/gebruiken
        ↓
4. Controller-methode maken
        ↓
5. Data ophalen
        ↓
6. Blade-view maken
        ↓
7. Data weergeven
        ↓
8. Vertalingen toevoegen
        ↓
9. Navigatie toevoegen
        ↓
10. Permissions controleren
        ↓
11. Testen
        ↓
12. Code controleren
```