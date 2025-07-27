# Event Listener Example (HTML+JS)

## Opis

Ten projekt to czysty, szkoleniowy przykład pokazujący działanie niestandardowego systemu zdarzeń w przeglądarce. Symuluje działanie modułu GPS, który co sekundę zwiększa dystans i generuje zdarzenie `near` co 5 jednostek.

Umożliwia dynamiczne dodawanie i usuwanie nasłuchiwaczy zdarzeń przez interfejs HTML oraz pokazuje logi w czasie rzeczywistym. Stylizacja CSS sprawia, że nowe wpisy pojawiają się na górze loga.

Kod nie korzysta z żadnych frameworków. Nadaje się do wykorzystania w środowiskach o ograniczonych zasobach, np. w interfejsach WWW hostowanych bezpośrednio z pamięci EEPROM mikrokontrolera (ESP32, STM32 itd.).

---

## Demo online

Możesz zobaczyć działanie tego przykładu bez instalacji, otwierając link poniżej:  
[https://volter2pl.github.io/js-examples/event-listener.html](https://volter2pl.github.io/js-examples/event-listener.html)

## Funkcje

### Klasa `GPS`

* `start()` - uruchamia symulację GPS (co sekundę zwiększa dystans)
* `stop()` - zatrzymuje symulację
* `distance` - licznik dystansu widoczny na przycisku
* `EventTarget` - obsługa zdarzeń `near`, `on`, `off`
* co 5 jednostek generuje zdarzenie `near`

### Interfejs

* Przycisk `loop on/off` - włącza/wyłącza symulację GPS
* `notice` - dodaje/usuwa nasłuchiwacz, który wypisuje: "you are close to gate"
* `open` - dodaje/usuwa nasłuchiwacz, który wypisuje: "i'm opening the gate"
* Dwa osobne zdarzenia `near` mogą mieć różne reakcje
* Podwójne kliknięcie w licznik dystansu czyści log

### Logowanie

* `document.log()` i `document.warn()` dodają wpisy do listy `#log`
* Kolorystyka i animacje pokazują że zdarzenie zostało zarejestrowane
* CSS obraca listę `ul`, by nowe zdarzenia były widoczne na górze

---

## Sekcja szkoleniowa: jak to działa

### 1. EventTarget w praktyce

Klasa `GPS` nie tylko symuluje ruch, ale wewnętrznie korzysta z mechanizmu `EventTarget` do zarządzania zdarzeniami. To pozwala tworzyć obiekty, które zachowują się jak DOM-owe elementy pod względem zdarzeń:

```js
this.eventTarget = new EventTarget();
```

Wszystkie metody `addEventListener`, `removeEventListener` i `dispatchEvent` są po prostu delegowane do `eventTarget`, co pozwala użytkownikowi klasy rejestrować zdarzenia tak jakby pracował z DOM:

```js
gps.addEventListener("near", () => { ... });
```

### 2. Symulacja pętli czasowej

Funkcja `loop()` wywołuje się rekurencyjnie przez `setTimeout()` co 1000 ms. To klasyczna forma nieskończonej pętli symulacyjnej bez blokowania UI.

```js
setTimeout(() => {
  if (this.on) {
    this.distance++;
    ...
    this.loop();
  }
}, 1000);
```

### 3. Zdarzenia `near`, `on`, `off`

* `near` wywoływane co 5 jednostek dystansu
* `on` generowane przy `start()`
* `off` generowane przy `stop()`

Możesz rejestrować osobne listenery dla różnych zdarzeń. W tym przykładzie mamy dwa dla `near`: jeden do wypisywania komunikatu "close", drugi do otwierania bramy.

### 4. Dynamiczna obsługa przycisków

Przyciski `off`/`on` są parami, a kliknięcie jednego ukrywa go i pokazuje drugi. Dzięki temu możesz dynamicznie dodać lub usunąć nasłuchiwacza bez przeładowania strony:

```js
button.noticeAdd.addEventListener("click", () => {
    gps.addEventListener("near", gateHandlerNotice);
    button.noticeAdd.style.display = "none";
    button.noticeRemove.style.display = "inline";
});
```

---

## Zgodność z SOLID – litera "O"

Zgodnie z zasadą **Open/Closed Principle** (OCP), klasa `GPS` jest **otwarta na rozszerzanie**, ale **zamknięta na modyfikację**. Pozwala to na dodawanie nowych funkcji bez zmieniania kodu klasy:

### Co można dodać bez modyfikacji GPS?

* ✅ **nowy listener** logujący dystans do `localStorage`:

  ```js
  gps.addEventListener("near", () => {
      localStorage.setItem("lastNearEvent", gps.distance);
  });
  ```

* ✅ **listener `on`** wyświetlający powiadomienie:

  ```js
  gps.addEventListener("on", () => alert("GPS tracking started!"));
  ```

* ✅ **UI progres-bar** aktualizowany przy `near`:

  ```js
  gps.addEventListener("near", () => {
      document.getElementById("bar").style.width = gps.distance + "%";
  });
  ```

* ✅ **komunikacja z backendem**:

  ```js
  gps.addEventListener("near", () => {
      fetch("/api/log", {
          method: "POST",
          body: JSON.stringify({distance: gps.distance})
      });
  });
  ```

## Przeznaczenie / zastosowania

Współcześnie kod nie ma zastosowania produkcyjnego, ale może być przydatny w:

* nauce czystego JavaScriptu
* prezentacjach mechanizmu `EventTarget`
* prototypowaniu interfejsów dla systemów embedded (ESP8266/ESP32)
* aplikacjach HTML+JS hostowanych lokalnie z EEPROM/Flash (np. ESP32 WebServer)
* sytuacjach, gdzie frameworki typu React/Vue są niedostępne lub nieopłacalne

---

## Jak uruchomić

1. Skopiuj zawartość pliku do lokalnego pliku `index.html`
2. Otwórz w przeglądarce (działa offline)
3. Eksperymentuj z przyciskami, obserwuj logi

Nie wymaga połączenia z internetem ani serwera.

---

## Uwagi

* Kod jest w pełni autonomiczny
* Może być przystosowany do pracy z `WebSerial`, `WebSocket`, `Fetch` lub innymi API dla mikrokontrolerów
* Do osadzenia w ESP32 można go minifikować i wrzucić do SPIFFS/LittleFS
