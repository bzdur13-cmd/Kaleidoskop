# Kalejdoskop 🔮

Aplikacja webowa (PWA-ready) symulująca kalejdoskop. Obraz o symetrii lustrzanej
i „szkiełka" reagują zgodnie z fizyką na:

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
- **Symetria** – liczba osi symetrii (4/6/8/10/12),
- **Wstrząśnij** – ręczny impuls,
- **Kolory** – zmiana palety szkiełek,
- **Zrób zdjęcie** – zapis PNG + udostępnianie.

## Jak to działa (technicznie)
- **Canvas 2D**: szkiełka (cząstki) rysowane są raz na „scenie" offscreen, a obraz
  kalejdoskopu powstaje przez wycięcie jednego klina (`clip`) i powielenie go
  z lustrzanym odbiciem co drugi segment → pełna symetria radialna.
- **Fizyka**: własna pętla `requestAnimationFrame` z krokiem czasowym; grawitacja
  jest transformowana do obróconego układu wzoru, więc kręcenie obrazem realnie
  przesuwa szkiełka (odczuwalna bezwładność).
- **Sensory**: `accelerationIncludingGravity` → kierunek grawitacji;
  `acceleration` → detekcja wstrząsu.
- **Eksport**: `canvas.toBlob()` → `navigator.share({files})` z fallbackiem na
  pobranie pliku (`<a download>`).

## Pomysły na rozwój
- **WebGL / PixiJS / Three.js** dla płynniejszej grafiki i efektów szkła (refrakcja, bloom).
- **Matter.js** dla dokładniejszych kolizji między szkiełkami.
- Realny **PWA manifest + service worker** (instalacja „na ekranie głównym", offline).
- Nagrywanie krótkiego **wideo/GIF** zamiast pojedynczej klatki (`MediaRecorder` + `captureStream`).
- Kalibracja orientacji względem `DeviceOrientation` (kompas) zamiast samego akcelerometru.

## Stack docelowy (rekomendacja)
| Warstwa | Technologia |
|---|---|
| Grafika | Canvas 2D (teraz) → WebGL/PixiJS (docelowo) |
| Fizyka | własna pętla (teraz) → Matter.js (opcjonalnie) |
| Czujniki | DeviceMotion / DeviceOrientation API |
| Eksport | Web Share API + `<a download>` |
| Dystrybucja | PWA (bez app store) lub opakowanie w Capacitor dla sklepów |
