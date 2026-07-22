# Kalejdoskop 🔮

Aplikacja webowa (PWA-ready) symulująca kalejdoskop. Sztywne, geometryczne
szkiełka (figury, listki, motylki, klejnoty) opadają, **zderzają się między sobą
i ze ściankami (lustrami)** układając symetryczny wzór na prążkowanym,
podświetlonym tle. Reagują zgodnie z fizyką na:

- **obracanie telefonu** → kierunek grawitacji (szkiełka opadają „w dół" niezależnie od orientacji ekranu),
- **potrząsanie** → impuls rozrzucający szkiełka (`DeviceMotion`),
- **kręcenie palcem** → moment obrotowy z bezwładnością i tarciem (siła odśrodkowa + Coriolis),
- **światło** → jasność z kamery rozświetla obraz i blask szkiełek.

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

### Na telefonie bez instalowania niczego — GitHub Pages
Czujniki ruchu i kamera działają **tylko przez HTTPS**. Najprościej włączyć
GitHub Pages (wszystko w przeglądarce, zero narzędzi):

1. w repo: **Settings → Pages**,
2. **Source: Deploy from a branch**, wybierz gałąź `claude/kaleidoskop-physics-app-buof29`
   (lub `main` po scaleniu) i katalog `/ (root)`, **Save**,
3. po chwili dostajesz adres `https://<user>.github.io/<repo>/` — otwórz go na telefonie,
4. dotknij **Uruchom** (zgoda na czujniki wymaga gestu), a przycisk **Światło** poprosi o kamerę.

Alternatywy (też bez instalacji apki): wrzucenie pliku na Netlify/Vercel drag&drop,
albo tunel HTTPS do lokalnego serwera (`ngrok http 8000`).

Na iOS ekran startowy z przyciskiem **Uruchom** jest konieczny, bo Safari wymaga
gestu użytkownika do włączenia czujników ruchu.

## Sterowanie / UI
- **Szkiełka** – zestaw kształtów: Geometryczne / Listki / Motylki / Klejnoty
  (każdy ma inne kształty i inne zachowanie: rozmiar, sprężystość),
- **Symetria** – liczba osi symetrii (6 lub 8),
- **Światło** – włącza/wyłącza czujnik światła z kamery (wymaga HTTPS i zgody),
- **Wstrząśnij** – ręczny impuls,
- **Kolory** – zmiana palety szkiełek,
- **Zrób zdjęcie** – zapis PNG + udostępnianie.

## Jak to działa (technicznie)
- **Symetria kalejdoskopu**: wszystkie szkiełka żyją w **jednej komorze** (jednym
  klinie). W trybie WebGL symetria składana jest w shaderze (zawinięcie kąta z
  odbiciem), a w fallbacku 2D przez `clip` + lustrzane odbicie co drugi segment —
  w obu na granicach szkiełka stykają się ze swoimi odbiciami.
- **WebGL (szkło)**: obraz komory (tło + szkiełka) idzie jako tekstura do shadera,
  który dokłada falowanie (refrakcja), aberrację chromatyczną, pseudo-bloom i sheen
  w stronę światła. Gdy WebGL niedostępny — automatyczny fallback na Canvas 2D.
- **Fizyka pojemnika (stertki)**: solver pozycyjny (PBD) z kilkoma iteracjami —
  szkiełka układają się w **stabilne stosy** pod ściankami, a przy obrocie /
  wstrząsie / przechyle **zsypują się** jak realne szkiełka w pojemniku. Ściany klina
  to lustra: środek szkiełka trzymany jest o `r` od ściany, więc styka się ze swoim
  odbiciem.
- **Lekki plastik**: mały luz (~0,1–0,2 mm na ~1 mm szkiełku) + tarcie modelujemy
  jako **mikro-drgania** rosnące ze zmianą przechyłu (sub-pikselowe w spoczynku) —
  szkiełka nigdy nie stoją idealnie martwo.
- **Światło**: kamera (`getUserMedia`) → średnia jasność klatki steruje natężeniem
  (przesuwasz telefon do światła → obraz jaśnieje, mocniejszy blask). Dochodzi lekkie
  migotanie. Kierunek światła (sheen) podąża za przechyłem.
- **Sensory ruchu**: `accelerationIncludingGravity` → grawitacja; `acceleration` →
  wstrząs. Na komputerze (brak żyroskopu) przechył symulują strzałki ←/→, ↑ = reset,
  spacja = wstrząs.
- **Eksport**: `canvas.toBlob()` → `navigator.share({files})` z fallbackiem na
  pobranie pliku (`<a download>`).

### Dlaczego grawitacja „na komputerze idzie do środka”?
Dwie przyczyny: (1) komputer nie ma żyroskopu, więc kierunek „w dół” jest domyślny
i stały (na telefonie bierze się realny czujnik); (2) w kalejdoskopie szkiełka
opadają tylko w jednej komorze, a pozostałe segmenty to jej **lustrzane odbicia** —
odbita grawitacja tworzy symetryczną gwiazdę, więc nigdy nie wygląda jak proste
„wszystko w dół”. To jest poprawne zachowanie kalejdoskopu.

## Dane materiałowe (lekki plastik — do strojenia fizyki)
Szkiełka w tanich kalejdoskopach to zwykle lekki plastik (PS/akryl), nie szkło:

| Wielkość | Wartość orientacyjna |
|---|---|
| Gęstość | polistyren ~1,05 g/cm³, akryl (PMMA) ~1,18 g/cm³, PET ~1,38 g/cm³ |
| Tarcie plastik–plastik | statyczne µ ≈ 0,3–0,4, kinetyczne µ ≈ 0,2–0,3 |
| Restytucja (odbicie) | ~0,4–0,6 pojedynczo; w stercie efektywnie niżej (tłumienie) |
| Grubość szkiełka | ~1 mm |
| Luz w komorze | ~0,1–0,2 mm → mikro-ruch przy zmianie przechyłu |

W kodzie odwzorowane jako: niska restytucja (`rest≈0,3`), silne tłumienie prędkości,
„drzemka” prawie stojących szkiełek oraz mikro-drgania rosnące ze zmianą przechyłu.

## Pomysły na rozwój
- **✅ WebGL** (szkło: refrakcja, aberracja, bloom) — jest; dalej: prawdziwa refrakcja
  z mapą normalnych per-szkiełko, głębia/parallax.
- **✅ Czujnik światła z kamery** — jest (jasność klatki); dalej: **kierunek** światła
  z jasnego obszaru kadru, różne reakcje materiałów (mat vs faseta).
- **Matter.js** dla dokładniejszych kolizji (obrót brył, realny kształt zamiast kul).
- **PWA manifest + service worker** (instalacja „na ekranie głównym”, offline).
- Nagrywanie **wideo/GIF** zamiast klatki (`MediaRecorder` + `captureStream`).

## Stack
| Warstwa | Technologia |
|---|---|
| Grafika | **WebGL** (shader) + Canvas 2D (komora, fallback) |
| Fizyka | własny solver pozycyjny (PBD) → Matter.js (opcjonalnie) |
| Czujniki | DeviceMotion / DeviceOrientation + kamera (`getUserMedia`) |
| Eksport | Web Share API + `<a download>` |
| Dystrybucja | PWA (bez app store) lub Capacitor dla sklepów |
