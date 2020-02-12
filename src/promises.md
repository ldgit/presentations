---
title: Promises
marp: true
theme: default
class: invert
# Staviti pod prvi slide za 4:3 aspect ratio: <!-- size: 4:3 -->
---

<style>.hljs-string { color: darkgreen; } .hljs-subst { color: black; }</style>

<!-- _class: lead -->
<!-- _paginate: false -->

# :fireworks: Promises :confetti_ball:

Prevedeno/ukradeno sa https://www.youtube.com/watch?v=hf1T_AONQJU.

---

<!-- _class: -->

## Exhibit :a:

```javascript
function getUserSync(name) {
  const response = ajaxSync(`/user/${name}`);
  const { user } = response;
  if (!user) throw new Error('User not found');
  return user;
}
```

<!-- Sinkrona funkcija: tražimo korisnika preko sync ajax poziva i ako ga nema bacamo iznimku -->

---

<!-- _class: -->

## Exhibit :b:

<!--
Verzija iste funkcije sa async callback funkcijama.

Argumenti callback funkcije (error, result) su bazirani na Node.js konvenciji:
- ako je došlo do greške, callback se poziva samo sa greškom kao prvim argumentom
- ako je pronađen rezultat, callback se poziva sa `null` kao prvim argumentom i rezultatom kao drugim argumentom
-->

```javascript
function getUserAsync(name, callback) {
  ajax(
    `/user/${name}`,
    response => {
      const { user } = response;
      if (!user) {
        callback(new Error('User not found'));
      } else {
        callback(null, user);
      }
    },
    error => callback(error),
  );
}
```

---

<!-- _class: -->

<!--
Možemo primjetiti kako async funkciji fale dvije ključne riječi koje sinkrona funkcija ima:
- return
- throw
-->

```javascript
function getUserSync(name) {
  const response = ajaxSync(`/user/${name}`);
  const { user } = response;
  if (!user) throw new Error('User not found');
  return user;
}
```

```javascript
function getUserAsync(name, callback) {
  ajax(
    `/user/${name}`,
    response => {
      const { user } = response;
      if (!user) {
        callback(new Error('User not found'));
      } else {
        callback(null, user);
      }
    },
    error => callback(error),
  );
}
```

---

# Async callback problemi

Funkcije mogu raditi tri stvari:
:one: Vratiti vrijednost
:two: Baciti iznimku
:three: Side effect

Sa callback funkcijama izgubili smo :one: i :two: te se oslanjamo samo na :three:

---

<!-- _class: -->

## Još problema

```javascript
// Pronađite bug!
function getUsers(names, callback) {
  const users = [];

  names.forEach(name => {
    ajax(
      `/user/${name}`,
      ({ user }) => {
        if (!user) {
          callback(new Error('User not found'));
        } else {
          users.push(user);
          if (users.length === names.length) {
            callback(null, users);
          }
        }
      },
      error => callback(error),
    );
  });
}
```

<!--
Sync verzija funkcije getUsers():
- dohvaća korisnike sa zadanim imenima
- svi korisnici moraju postojati, ako ijedan fali bacamo iznimku greška!

Bug: ako pošaljemo dva imena za koje nema korisnika, callback će biti pozvan dva puta
Čak i ako bacimo iznimku ne možemo prekinuti izvođenje funkcije, ostali callbacks će se nastaviti izvoditi.
-->

---

# Async callback trade-off

- Izgubili smo `return` :fish:
- Izgubili smo `throw` :blowfish:
- Izgubili smo **Stack** :tropical_fish:
- Ne znamo hoće li **callback** biti pozvan više puta i sa različitim argumentima :crab:
  - Sync funkcije mogu vratiti najviše jednu vrijednost
  - Sync funkcije mogu baciti najviše jednu iznimku
- Ali naša funkcija je sad **non-blocking** :tada:

<!-- Pod gubitkom stacka se misli i na mogućnost da bacanje iznimke prekine izvođenje funkcije -->
<!-- Ne znamo koliko će se puta callback pozvati - sync funkcija vrati vrijednost najviše jednom -->

---

# Promise

- aka _future_, _delay_ ili _deferred_
- uvedeni u [ECMAScript 2015](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- `Promise` predstavlja **asinkronu** operaciju
- Rezultat operacije nije nužno poznat u trenutku kada je Promise objekt stvoren

---

<!-- _class: -->

# Ilustracija<sup>\*</sup>

- Kolega :turtle: je obećao predavanje, tema: **Event loop & Promises**
- Dao je to obećanje kolegama
- Sa danim obećanjem kolege barataju kako god žele:
  - Kolega :rabbit: kaže da će prijeći u frontend tim nakon izvršenog obećanja (:white_check_mark:)
  - Kolega :elephant: sumnja da će :turtle: izvršiti obećanje, ako ne uspije dati će mu [:fish:](https://youtu.be/T8XeDvKqI4E?t=13) (:x:)
  - Kolegica :honeybee: je obećala svom prijatelju da će mu, kad posluša to predavanje, objasniti kako **Promise** radi (:white_check_mark:)
- Svi kolege mogu nastaviti sa drugim poslovima dok obećanje nije izvršeno/odbačeno

<sup>\* svaka sličnost sa stvarnim osobama je slučajna</sup>

<!-- :rabbit: je postavio onFulfilled handler -->
<!-- :elephant: je postavio onRejected handler -->
<!-- :honeybee: je postavila onFulfilled handler koji stvara novo obećanje i poslala to obećanje svom prijatelju, koji na to obećanje može postaviti svoj onFulfilled/onRejected handler, itd... -->
<!-- Svi kolege mogu nastaviti sa drugim poslovima jer obećanje predstavlja rezultat asinkrone operacije -->
<!-- Ako :turtle: kaže da neće izvršiti, ni obećanje koje je :honeybee: dala prijatelju neće biti ispunjeno, osim ako nije postavila i error handler u kojem bi npr. naučila o Promise-ima sa interneta -->

---

## Stanja `Promise` objekta

<!-- prettier-ignore -->
- :alarm_clock: _pending_ - u tijeku
  - Početno stanje
  - Jedino koje može prijeći u neko drugo stanje
<!-- prettier-ignore -->
- :white_check_mark: _fulfilled_ - ispunjeno
  - Operacija je uspješno završena, s najviše jednom vrijednošću
  - Vrijednost se **neće** promijeniti
- :x: _rejected_ - odbačeno 
  - Operacija neuspješno završena, opcionalno s nekim razlogom (error)
  - Razlog se **neće** promijeniti

<!--
Pod "neće se promijeniti" se *ne misli* na duboku nepromjenjivost (npr. ako je objekt, property-ji se mogu promijeniti).
Analogno varijabli definiranoj sa `const`.
-->

---

## Metode `Promise` objekta

`.then(onFulfilled, onRejected)`

- Prima dva handler argumenta (ignoriraju se ako nisu funkcije)
  - `onFulfilled` - izvršava se kada je obećanje ispunjeno
  - `onRejected` - izvršava se kada je obećanje odbačeno
- Vraća novi `Promise` objekt ovisno o tome što se dogodilo u handler-ima

<!-- onFulfilled će se izvršiti čak i ako je obećanje već ispunjeno u trenutku kada smo pozvali then() -->
<!-- Analogno vrijedi i za onRejected -->

`.catch(onRejected)`

- Ekvivalent `aPromise.then(undefined, onRejected)`

---

## [Primjer stvaranja](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise)

<!-- Svaku async callback funkciju možemo vrlo jednostavno pretvoriti u funkciju koja vraća Promise. -->

Pretvorba `getUserAsync()` funkcije iz [primjera :b:](#3) u funkciju koja vraća `Promise`:

```javascript
function getUserPromise(name) {
  return new Promise((resolve, reject) => {
    getUserAsync(name, (error, user) => {
      if (error) {
        reject(error);
      } else {
        resolve(user);
      }
    });
  });
}
```

<!--
Callback koji šaljemo u `new Promise` se izvršava odmah
- pozivom resolve metode pozvati će se svi zakačeni onFulfilled handleri
- pozivom reject metode pozvati će se svi zakačeni onRejected handleri
-->

---

<!-- _class:  -->

## Konverzija poziva callback funkcije u Promise funkciju

```javascript
getUser('ldgit', (error, user) => {
  // ...
});
```

Postaje

```javascript
getUser('ldgit').then(
  user => {
    // ...
  },
  error => {
    // ...
  },
);
```

---

<!-- _class:  -->

## Brzi zadaci

S kojom vrijednošću su ispunjeni/odbijeni ovi `Promise` objekti?

```javascript
new Promise((resolve, reject) => {
  resolve(5);
  reject(new Error('Ne valja'));
});
```

<!-- Prvi je ispunjen sa vrijednošću 5, error handler se neće pozvati -->

```javascript
new Promise((resolve, reject) => {
  reject(new Error('Stani!'));
  resolve('Riješeno');
});
```

<!-- Drugi je odbijen sa greškom "Stani!", success handler se neće pozvati -->

```javascript
new Promise((resolve, reject) => {
  resolve(5);
  resolve('Riješeno');
});
```

<!-- Treći je ispunjen sa vrijednošću 5, success handler će pozvati samo jednom -->

---

## Promise/Synchronous paralele

- `onFulfilled` i `onRejected`:
  - Izvršit će se najviše jednom
  - S najviše jednom vrijednošću/greškom
- `Promise` se može ispuniti ili odbaciti samo jednom: daljni `resolve` i `reject` pozivi nakon prvoga ne rade ništa

:exclamation:Paralele su `return` i `throw` :exclamation:

---

## Promise/Synchronous paralele

:zap: Cilj Promise-a je dobiti nazad mogućnosti sinkronog kôda u async funkcijama. :zap:

Ponašanje `onFulfilled` i  `onRejected` handlera pokriva četiri osnovna scenarija, ovisno o stanju `Promise` objekta:
1. Ispunjeno obećanje, `onFullfiled` vraća vrijednost <=> jednostavna funkcionalna transformacija
2. Ispunjeno obećanje, `onFullfiled` baca iznimku <=> bacanje iznimke kada dobijemo neispravnu vrijednost
3. Odbačeno obećanje, `onRejected` vraća vrijednost <=> `catch` unutar kojeg smo handle-ali grešku
4. Odbačeno obećanje, `onRejected` baca iznimku <=> `catch` unutar kojeg smo ponovo bacili istu (ili novu) grešku

---

<!-- _class:  -->

## Sync patterni sa Promise objektima :one:

Dohvaćanje vrijednosti, bacanje iznimke ako vrijednost ne postoji:

```javascript
const user = getUser('ldgit');
if (!user) {
  throw new Error('User not found');
}
const name = user.name;
```

Promise verzija:

```javascript
getUser('ldgit').then(user => {
  if (!user) {
    throw new Error('User not found');
  }
  return user.name;
});
```

<!--
Async kôd sada funkcionira vrlo slično sync kôdu:
- `return user.name` će vratiti novi promise koji ispunjava sa korisnikovim imenom
- ako je došlo do greške, `throw` će vratiti novi promise koji je odbačen sa greškom
- vraćenu vrijednost, ili bačenu grešku, imamo u idućem .then() pozivu: unutar success handlera, ili error handlera u slučaju greške
-->

---

<!-- _class:  -->

## Sync patterni sa Promise objektima :two:

Obrada iznimki:

```javascript
try {
  notifyUser('ldgit');
} catch (error) {
  handleError(error);
}
```

Promise verzija:

```javascript
notifyUser('ldgit').then(undefined, error => {
  handleError(error);
});
```

---

<!-- _class:  -->

## Sync patterni sa Promise objektima :three:

Re-throw iznimki:

```javascript
try {
  notifyUser('ldgit');
} catch (error) {
  throw new Error(`Došlo je do greške: ${error.message}`);
}
```

Promise verzija:

```javascript
notifyUser('ldgit').then(undefined, error => {
  throw new Error(`Došlo je do greške: ${error.message}`);
});
```

<!-- Sve operacije iz sinkronog svijeta su nam dostupne i u asinkronom sa Promise-ima -->

---

<!-- _class: -->

# Niz operacija

```javascript
// Sinkroni kôd
const user = getUser('ldgit');
const reviews = getReviews(user);
renderReviews(reviews);
```

```javascript
// Callbacks
getUser('ldgit', user => {
  getReviews(user, reviews => {
    renderReviews(reviews);
  });
});
```

```javascript
// Promises
getUser('ldgit')
  .then(getReviews)
  .then(renderReviews);
```

---

<!-- _class: -->

# Baratanje iznimkama

Sinkroni kôd

```javascript
try {
  const user = getUser('ldgit');
  const reviews = getReviews(user);
  renderReviews(reviews);
} catch (error) {
  handleError(error);
}
```

---

<!-- _class: -->

# Baratanje iznimkama

Callbacks :volcano:

```javascript
getUser('ldgit', (error, user) => {
  if (error) {
    handleError(error);
  } else {
    getReviews(user, (error, reviews) => {
      if (error) {
        handleError(error);
      } else {
        renderReviews(reviews);
      }
    });
  }
});
```

---

<!-- _class: -->

# Baratanje iznimkama

Promises

```javascript
getUser('ldgit')
  .then(getReviews)
  .then(renderReviews)
  .then(undefined, handleError);
```

&nbsp;&nbsp;:sunny:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;:bird:

:evergreen_tree::herb::rabbit2::deciduous_tree::blossom:

<!-- Sa Promises smo na određeni način dobili nazad stack: greške koje bacimo duboko u stack-u će "isplivati" na vrh, do prvog catch-a ili onRejected handlera -->

---

# Paralelne asinkrone operacije

Postižemo [koristeći](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) `Promise.all(iterable)`

Prima (najčešće) `array` `Promise` objekata, a vraća novi `Promise` koji je:

- Ispunjen kada su **svi** `Promise` objekti ispunjeni
- Odbačen ako je **bilo koji** u nizu promise-a odbačen
  - Greška je greška prvog odbačenog promise objekta

---

<!-- _class: -->

Konverzija `getUsers` funkcije iz [ranijeg](#6) primjera:

```javascript
function getUsers(names, callback) {
  const userPromises = names.map(name => {
    // ajax funkcija je asinkrona i vraća promise
    return ajax(`/user/${name}`).then(({ user }) => {
      if (!user) {
        throw new Error('User not found');
      }
      return user;
    });
  });

  return Promise.all(userPromises);
}
```

<!--
Promise.all() prima array promise objekata, a vraća novi promise koji:
- je ispunjen ako su *svi* promise objekti ispunjeni
- oodbačen ako je bilo koji u nizu promise-a odbačen, greška je greška prvog odbačenog promise objekta
-->

---

<!-- _class: -->
<!-- _paginate: false -->

# Budućnost :rocket:

---

<!-- _paginate: false -->

![bg 90%](images/tis_fine.png)

---

<!-- _class: -->
<!-- _paginate: false -->

![bg 90% invert:100%](images/tis_fine.png)

---

# Async Await :tada:

- Uvedeni u [ECMAScript 2017](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Async_await)
- Sintaksni šećer za `Promise` :cake:
- Async kôd izgleda sličnije tradicionalnom sync kôdu
- Ograničen na [novije preglednike](https://caniuse.com/#feat=async-functions)

---

## Async funkcije

- Označene su `async` prefiksom
- Implicitno vraćaju `Promise` objekt
- I arrow funkcije mogu biti async

---

## Async funkcije

Bilo koju običnu funkcije možemo pretvoriti u asinkronu ako joj dodamo `async` prefix:

```javascript
function getFishCount() {
  return 42; // vraća integer 42
}

async function getFishCount() {
  return 42; // vraća Promise koji se resolve-a sa 42
}
```

---

## Await

- Ključna riječ `await` se može staviti ispred bilo kojeg **poziva** asinkrone funkcije
- Ako `await` koristimo unutar funkcije, ta funkcija **mora** biti označena kao asinkrona kroz `async`
- Pauzira izvođenje funkcije unutar koje se nalazi dok se vraćeni `Promise` ne ispuni

<!-- Ako koristimo await unutar ne-async funkcije dobiti ćemo sintaksnu grešku -->

---

<!-- _class: -->

## Promise chain napisan preko Async Await (sync verzija [ovdje](#21))

```javascript
try {
  const user = await getUser('ldgit');
  const reviews = await getReviews(user);
  renderReviews(reviews);
} catch (error) {
  handleError(error);
}
```

Unutar funkcije:

```javascript
async function getReviewsForUser(username) {
  const user = await getUser(username);
  return await getReviews(user);
}

renderReviews(await getReviewsForUser('ldgit'));
```

---

<!-- _class: -->

# Izvori

- [Redemption from Callback Hell](https://www.youtube.com/watch?v=hf1T_AONQJU) | Michael Jackson & Domenic Denicola
- [You're Missing the Point of Promises](https://blog.domenic.me/youre-missing-the-point-of-promises/) | Domenic Denicola :raised_hands:
- [Promises/A+ spec](https://promisesaplus.com/)

# For fun

- Stvarno cool [test suite](https://github.com/promises-aplus/promises-tests) s kojim i vi možete napisati pravu `Promise` implementaciju.
