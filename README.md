# Kalejdoskop 🔮

Aplikacja webowa (PWA-ready) symulująca kalejdoskop. Sztywne, geometryczne
szkiełka (figury, listki, motylki, klejnoty) opadają, **zderzają się między sobą
i ze ściankami (lustrami)** układając symetryczny wzór na prążkowanym,
podświetlonym tle. Reagują zgodnie z fizyką na:

- **obracanie telefonu** → kierunek grawitacji (szkiełka opadają „w dół" niezależnie od orientacji ekranu),
- **potrząsanie** → impuls rozrzucający szkiełka (`DeviceMotion`),
- **kręcenie palcem** → moment obrotowy z bezwładnością i tarciem (siła odśrodkowa + Coriolis).

Gotowy obraz można **„sfotografować"**, **zapisać na dysk** oraz **wysłać na mail / social media**
przez natywne okno udostępniania (Web Share API).

## Uruchomienie

To pojedynczy plik `index.html` — nie wymaga budowania.

### Lokalnie na komputerze
```bash
# dowolny statyczny serwer, np.:
python3 -m http.server 8000
# potem otwórz http://localhost:8000
```
Na desktopie nie ma czujników ruchu — sterowanie:
- **przeciąganie myszą** = kręcenie obrazem,
- **strzałki ←/→** = obrót kierunku grawitacji,
- **spacja** = wstrząs.

### Na telefonie (pełna fizyka)
Czujniki ruchu (`DeviceMotion`/`DeviceOrientation`) działają **tylko przez HTTPS**.
Najprościej:
1. wrzuć plik na dowolny hosting statyczny (GitHub Pages, Netlify, Vercel), lub
2. użyj tunelu HTTPS do lokalnego serwera (np. `ngrok http 8000`).

Na iOS przy pierwszym uruchomieniu przeglądarka poprosi o zgodę na dostęp do
czujników ruchu — dlatego jest ekran startowy z przyciskiem **Uruchom**
(zgoda musi wynikać z gestu użytkownika).

## Sterowanie / UI
- **Szkiełka** – zestaw kształtów: Geometryczne / Listki / Motylki / Klejnoty
  (każdy ma inne kształty i inne zachowanie: rozmiar, sprężystość, „trzepotanie"),
- **Symetria** – liczba osi symetrii (6 lub 8),
- **Wstrząśnij** – ręczny impuls,
- **Kolory** – zmiana palety szkiełek,
- **Zrób zdjęcie** – zapis PNG + udostępnianie.

## Jak to działa (technicznie)
- **Prawdziwa symetria kalejdoskopu**: wszystkie szkiełka żyją w **jednej komorze**
  (jednym klinie). Ściany klina to lustra — szkiełka fizycznie się od nich odbijają.
  Pozostałe segmenty to lustrzane kopie tej samej komory (`clip` + odbicie co drugi
  segment), więc na granicach szkiełka stykają się ze swoimi odbiciami — dokładnie
  jak w prawdziwym kalejdoskopie.
- **Sztywne szkiełka + kolizje**: szkiełka to nieprzezroczyste figury (nie „świecące
  plamki”) z konturem i szklanym gradientem światła. Zderzają się **między sobą**
  (kolizje kul ograniczających z zachowaniem pędu) i układają wzór, jak realne
  szkiełka w komorze.
- **Fizyka**: własna pętla `requestAnimationFrame` z krokiem czasowym; grawitacja
  jest transformowana do obróconego układu wzoru, więc kręcenie obrazem realnie
  przesuwa szkiełka (odczuwalna bezwładność, siła odśrodkowa i Coriolisa).
- **Sensory**: `accelerationIncludingGravity` → kierunek grawitacji;
  `acceleration` → detekcja wstrząsu. Na komputerze (brak żyroskopu) przechył
  symulują strzałki ←/→, ↑ = reset „w dół”, spacja = wstrząs.
- **Światło / sheen**: rozświetlenie podąża za przechyłem (przeciwnie do grawitacji),
  dając złudzenie wpadającego światła — podstawa pod przyszły czujnik światła aparatu.
- **Eksport**: `canvas.toBlob()` → `navigator.share({files})` z fallbackiem na
  pobranie pliku (`<a download>`).

### Dlaczego grawitacja „na komputerze idzie do środka”?
Dwie przyczyny: (1) komputer nie ma żyroskopu, więc kierunek „w dół” jest domyślny
i stały (na telefonie bierze się realny czujnik); (2) w kalejdoskopie szkiełka
opadają tylko w jednej komorze, a pozostałe segmenty to jej **lustrzane odbicia** —
odbita grawitacja tworzy symetryczną gwiazdę, więc nigdy nie wygląda jak proste
„wszystko w dół”. To jest poprawne zachowanie kalejdoskopu.

## Pomysły na rozwój
- **Więcej kalejdoskopów** = kolejne zestawy szkiełek z własną fizyką i reakcją na
  światło (matowe listki inaczej rozpraszają światło niż fasetowane klejnoty).
- **Czujnik światła aparatu**: przy przesuwaniu telefonu „w stronę światła” (jasność
  z podglądu kamery lub `AmbientLightSensor`, gdzie dostępny) obraz jaśnieje, a sheen
  na szkiełkach i prążkach reaguje jak na realne, wpadające światło.
- **WebGL / PixiJS / Three.js** dla płynniejszej grafiki i prawdziwego szkła (refrakcja, bloom).
- **Matter.js** dla dokładniejszych kolizji (obrót brył, kształt zamiast kul).
- Realny **PWA manifest + service worker** (instalacja „na ekranie głównym", offline).
- Nagrywanie krótkiego **wideo/GIF** zamiast pojedynczej klatki (`MediaRecorder` + `captureStream`).

## Stack docelowy (rekomendacja)
| Warstwa | Technologia |
|---|---|
| Grafika | Canvas 2D (teraz) → WebGL/PixiJS (docelowo) |
| Fizyka | własna pętla (teraz) → Matter.js (opcjonalnie) |
| Czujniki | DeviceMotion / DeviceOrientation API |
| Eksport | Web Share API + `<a download>` |
| Dystrybucja | PWA (bez app store) lub opakowanie w Capacitor dla sklepów |
