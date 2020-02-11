---
title: Event loop
marp: true
theme: default
class: invert
# Staviti pod prvi slide za 4:3 aspect ratio: <!-- size: 4:3 -->
---

<style>.hljs-string { color: darkgreen; } .hljs-subst { color: black; }</style>

<!-- _class: lead -->

# :cyclone: JavaScript Event loop :recycle:

Prevedeno/ukradeno sa https://www.youtube.com/watch?v=8aGhZQkoFbQ

---

<!-- paginate: true -->

# JavaScript

<!-- Što je JavaScript? -->

- Single threaded :slightly_smiling_face:
- Concurrent :thinking:
- Non-blocking :upside_down_face:
- Asynchronous :worried:

---

<!-- _class:  -->

![bg 90% left:58%](images/js_runtime_complex.png)

<!-- JavaScript building blocks -->

## JavaScript engine

- Call stack
- Memory Heap

## Runtime environment

- Event Loop
- Callback Queue
- Web<sup>\*</sup> API-s

[Source](https://medium.com/@olinations/the-javascript-runtime-environment-d58fa2e60dd0)

<!-- Call stack - govori nam gdje smo trenutno trenutno u kôdu -->
<!-- Memory Heap - gdje JavaScript sprema objekte, varijable -->
<!-- JS engine: V8 (Chrome), SpiderMonkey (Firefox) -->
<!-- JS Runtime environment: Blink (Chrome), Gecko (Firefox) -->
<!-- * Web API-s u preglednicima, C++ API-s u Node-u -->

---

# Single threaded

- Jedan call stack
- Naredbe se izvršavaju jedna po jedna

---

## Call Stack

Radi na principu Last In, First Out (LIFO)

<!-- Zanemarimo za sada Web Apis i Callback Queue kvadrate -->

[Demo](http://latentflip.com/loupe/?code=ZnVuY3Rpb24gYW5vbnltb3VzKCkgewogICAgZnVuY3Rpb24gbXVsdGlwbHkoYSwgYikgewogICAgICAgIHJldHVybiBhICogYjsKICAgIH0KICAgIAogICAgZnVuY3Rpb24gc3F1YXJlKGEpIHsKICAgICAgICByZXR1cm4gbXVsdGlwbHkoYSwgYSk7CiAgICB9CiAgICAKICAgIGZ1bmN0aW9uIHByaW50U3F1YXJlKG51bWJlcikgewogICAgICAgIGNvbnN0IHJlc3VsdCA9IHNxdWFyZShudW1iZXIpOwogICAgICAgIGNvbnNvbGUubG9nKHJlc3VsdCk7CiAgICB9CiAgICAKICAgIHByaW50U3F1YXJlKDUpOwp9Cgphbm9ueW1vdXMoKTsK!!!Cg%3D%3D)

```javascript
const multiply = (a, b) => a * b;
const square = a => multiply(a, a);
const printSquare = number => {
  const result = square(number);
  console.log(result);
};

printSquare(5);
```

---

# Concurrency

- Istovremenost - više događaja se izvršava u isto vrijeme (naizmjenično, paralelno)
- JavaScript istovremenost = neke naredbe se izvršavaju van programskog toka
  - Ne idu na Call stack
- Zašto nam je istovremenost potrebna?

---

## Blokirajuće operacije :vertical_traffic_light:

- :snail: Neučinkoviti i spori kôd
- I/O (Input/Output) operacije
- :no_entry: Blokiraju izvođenje ostatka programa
- Problem u preglednicima:
  - Onemogućavaju korištenje UI stranice
  - Loš User Experience (UX)
- [Primjer](https://jsfiddle.net/ro10gam5/6/)

<!-- Spori kôd je predvidiv i konzistentno spor, može se popraviti direktnom intervencijom programera -->
<!-- I/O operacije ovise o drugim faktorima na koje programer ne može uvijek utjecati -->

---

<!-- _class: -->
<!-- Kôd iz primjera sa prošlog slide-a -->

```html
Polje za unos: <input type="text" /><br /><br />
<button id="runSlowButton">Klikni za spori kôd</button>
<button id="storeButton">Spremi</button>
```

```javascript
function verySlowFunction(miliseconds) {
  const startingTime = Date.now();
  while (true) {
    if (Date.now() - startingTime > miliseconds) {
      break;
    }
  }
}

document
  .getElementById('runSlowButton')
  .addEventListener('click', () => verySlowFunction(4000));
document
  .getElementById('storeButton')
  .addEventListener('click', () => console.log('Spremljeno'));
```

---

# Non-blocking

- Većina JavaScript I/O operacija **ne blokira** izvršavanje drugog JS kôda
- I/O
  - Mrežni zahtjevi
  - Operacije nad diskom
    <!-- Ne možemo predvidjeti koliko dugo će trajati HTTP request, ne želimo da za to vrijeme ne radi UI -->

---

# Asynchronous

- Asinkrone funkcije
  - izvršiti će se nakon što se završi neki drugi (non-blocking) zadatak
  - ne znamo kada će (i hoće li) taj zadatak biti završen
- Na primjer, za AJAX poziv možemo odmah odrediti:
  - što će se dogoditi ako se završi uspješno
  - što će se dogoditi ako ne uspije

<!-- Dobro objašnjenje ovog koncepta se nalazi ovdje: http://www.michael-richardson.com/processes/rup_for_sqa/core.base_rup/guidances/concepts/concurrency_EE2E011A.html#Asynchronous%20vs.%20synchronous%20interaction -->

---

![bg 90% left:60%](images/js_runtime_complex.png)

## Kako JavaScript postiže istovremenost

- Web APIs
- Queue
- Event loop

<!-- Non-blocking operacije se izvršavaju kroz WEB API i ne idu na call stack -->
<!-- Sve WEB API funkcije i metode primaju tzv. callback funkcije koje se šalju na tzv. Queue kad WEB API obavi svoj zadatak -->
<!-- Event loop određuje kako i kada će se poruke u Queue-u procesuriati -->

---

## Web APIs

- Nisu [dio Javascripta](https://developer.mozilla.org/en-US/docs/Web/API), ali JS ih najčešće koristi
  - Timing funkcije poput [setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout)
  - Mrežni zahtjevi: XMLHttpRequest (AJAX), [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
  - [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) (Document Object Model)
  - Rad nad diskom (npr. [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API))
- Primaju async callback funkcije i šalju ih kao poruke u **Queue** kad izvrše svoj zadatak

<!-- Činjenica da su dio preglednika, a ne JavaScripta, je ono što omogućava istovremenost: preglednici su multi-threaded -->

---

## Queue

# :snail::turtle::baby_chick::rabbit2::sheep::sheep::goat::cow2::dog2::cat2:

- Sadrži poruke koje je potrebno procesuriati
- Svaka poruka je JS funkcija - Async callback
- Poruke se procesuiraju po FIFO (First In, First Out) principu

---

## Event loop :arrows_counterclockwise:

- Promatra **Call Stack** i **Queue**
- Poruke iz **Queue**-a stavlja na **Call Stack** :exclamation:**kad je Call Stack prazan**:exclamation:
- Dok god **Call Stack** nije prazan:
  - **Event loop** ne može procesuirati druge poruke iz **Queue**-a
  - Korisnik ne može koristiti UI preglednika

---

<!-- _class:  -->

# :warning: Ne blokirajte Event Loop :warning:

---

## Async callback primjer

- `setTimeout` [funkcija](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout)
- Što će sljedeći kôd ispisati u konzolu?

  ```javascript
  console.log(1);
  setTimeout(function cb() {
    console.log(2);
  }, 0);
  console.log(3);
  ```

  [Demo](http://latentflip.com/loupe/?code=ZnVuY3Rpb24gYW5vbnltb3VzKCkgew0KICBjb25zb2xlLmxvZygnMScpOw0KICBzZXRUaW1lb3V0KGZ1bmN0aW9uIGNiKCkgew0KICAgIGNvbnNvbGUubG9nKCcyJyk7DQogIH0sIDApOw0KICBjb25zb2xlLmxvZygnMycpOyAgICANCn0NCg0KYW5vbnltb3VzKCk7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)

---

<!-- _class: -->

## Zagonetka :one:

<!-- Što će se ispisati u konzolu? -->

```javascript
function verySlowFunction(miliseconds) {
  const start = Date.now();
  while (true) {
    if (Date.now() - start > miliseconds) {
      break;
    }
  }
}

const startingTime = Date.now();
setTimeout(function cb() {
  // Što će ovo ispisati?
  console.log(`Prošlo je ${Date.now() - startingTime} milisekundi`);
}, 1000);

verySlowFunction(2000);
```

---

<!-- _class: -->

## Zagonetka :two:

```javascript
try {
  throw new Error('Ajme');
} catch (error) {
  console.log('Došlo je do greške: ', error);
}
```

```javascript
try {
  setTimeout(() => {
    throw new Error('Jao');
  }, 0);
} catch (error) {
  console.log('Došlo je do greške: ', error);
}
```

---

Ako želimo uhvatiti grešku, moramo `try-catch` staviti unutar callback funkcije

<!-- _class: -->

```javascript
setTimeout(() => {
  try {
    throw new Error('Jao');
  } catch (error) {
    console.log('Došlo je do greške: ', error);
  }
}, 0);
```

---

<!-- _class: -->

## Zagonetka :three:

```javascript
function verySlowFunction(miliseconds) {
  const start = Date.now();
  while (true) {
    if (Date.now() - start > miliseconds) {
      break;
    }
  }
}

const startingTime = Date.now();

setTimeout(() => {
  verySlowFunction(1000);
  console.log(`1. Prošlo je ${Date.now() - startingTime} milisekundi`);
}, 0);

setTimeout(() => {
  console.log(`2. Prošlo je ${Date.now() - startingTime} milisekundi`);
}, 500);
```

---

<!-- _class: -->

# Izvori

- [What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ) | Philip Roberts
- [Concurrency model and the event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop) | MDN
- [The Javascript Runtime Environment](https://medium.com/@olinations/the-javascript-runtime-environment-d58fa2e60dd0) | Jamie Uttariello
- [Asynchronous programming](https://www.computerhope.com/jargon/a/asynchro.htm#asynchronous-programming), [Concurrency](https://www.computerhope.com/jargon/c/concurre.htm) | Computer Hope
