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

---

<!-- _class: -->

## Exhibit :b:

<!--
Callback signature (error kao prvi argument, rezultat kao drugi) je baziran na Node.js konvenciji:
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
<!-- <style scoped>.hljs-string { color: #00c8f3; } .hljs-subst { color: white; }</style> -->

<!--
Proučimo ove dvije funkcije:
- Async funkciji fale dvije ključne riječi koje Sync funkcija ima, koje?
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

<!-- _class: -->
<!-- <style scoped>.hljs-string { color: #00c8f3; } .hljs-subst { color: white; }</style> -->

## Zagonetka :four:

```javascript
// Pronađite bug:
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
Funkcija getUsers() dohvaća korisnike sa zadanim imenima, svi korisnici moraju postojati, ako ijedan fali => greška!

Bug: ako pošaljemo dva imena za koje nema korisnika, callback će biti pozvan dva puta
-->

---

# Problemi async callback-a

- Izgubili smo `return` :fish:
- Izgubili smo `throw` :blowfish:
- Izgubili smo korisne **Stack** informacije :tropical_fish:
- Ne znamo hoće li **callback** biti pozvan više puta i sa različitim argumentima :crab:
- Ali naša funkcija je sad **non-blocking** :tada:

<!-- Ne znamo koliko će se puta callback pozvati - sync funkcija vrati vrijednost najviše jednom -->
<!--
Funkcije mogu napraviti tri stvari: vratiti vrijednost, baciti iznimku ili imati side-effect.
Sa callbacks smo ograničeni samo na side-effects
-->

---

# Promise

- aka _future_, _delay_ ili _deferred_
- uvedeni u [ECMAScript 2015](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- `Promise` predstavlja **asinkronu** operaciju
- Rezultat operacije nije nužno poznat u trenutku kada je Promise objekt stvoren

---

## Stanja `Promise` objekta

<!-- prettier-ignore -->
- :alarm_clock: _pending_ - u tijeku, početno stanje
  - Jedino koje može prijeći u neko drugo stanje
<!-- prettier-ignore -->
- :white_check_mark: _fulfilled_ - ispunjeno, operacija je uspješno završena, opcionalno sa najviše jednom vrijednošću
  - Vrijednost se **neće** promijeniti
- :x: _rejected_ - odbačeno, operacija neuspješno završena, opcionalno sa nekim razlogom (error)
  - Razlog se **neće** promijeniti

<!-- Pod "neće se promijeniti" se *ne misli* na duboku nepromjenjivost (npr. ako je objekt, property-ji se mogu promijeniti) -->

---

## Metode `Promise` objekta

`.then(onFulfilled, onRejected)`

- Prima dva opcionalna argumenta (ignoriraju se ako nisu funkcije)
  - `onFulfilled` - izvršava se kada je obećanje ispunjeno
  - `onRejected` - izvršava se kada je obećanje odbačeno
- Vraća novi `Promise` objekt

<!-- onFulfilled će se izvršiti čak i ako je obećanje već ispunjeno u trenutku kada smo pozvali then() -->
<!-- Analogno vrijedi i za onRejected -->

`.catch(onRejected)`

- Ekvivalent `Promise.prototype.then(undefined, onRejected)`

---

## [Primjer stvaranja](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise)

Konvertirali smo async `getUserAsync()` funkciju primjera :b: u funkciju koja vraća `Promise`:

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

<!-- _class:  -->

## Konverzija iz Async callbacks u Promise

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

## Paralele su `return` i `throw`

- `onFulfilled` i `onRejected` su async verzije `return` i `throw`:
  - Izvršit će se samo jednom
  - S najviše jednom vrijednošću
  - Svi daljni `resolve` i `reject` pozivi nakon prvoga ne rade ništa

---

<!-- _class:  -->

## Kratki primjeri

```javascript
new Promise((resolve, reject) => {
  resolve(5);
  reject(new Error('Ne valja'));
});
```

```javascript
new Promise((resolve, reject) => {
  reject(new Error('Stani!'));
  resolve('Riješeno');
});
```

```javascript
new Promise((resolve, reject) => {
  resolve(5);
  resolve('Riješeno');
});
```

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
- `return user.name` će vratiti novi promise koji resolve-a sa korisnikovim imenom
- ako je došlo do greške, `throw` će vratiti novi promise koji je reject-an sa greškom
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

## Zagonetka :five:

Što će se ispisati u konzolu?

```javascript
new Promise(resolve => resolve(42))
  .then(value => { throw new Error(value); })
  .then(value => value + 5)
  .then(
    value => console.log(`Vrijednost je ${value}`),
    error => console.log('Došlo je do greške', error),
  );
```

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

Callbacks

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

<!-- Sa Promises smo na određeni način dobili nazad stack: greške koje bacimo duboko u stack-u će "isplivati" na vrh, do prvog catch-a ili onRejected handlera -->

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

![bg 90% saturate:5.0 invert:100%](images/tis_fine.png)

---

# Async Await :tada:

- Uvedeni u [ECMAScript 2017](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Async_await)
- Sintaksni šećer za `Promise` :cake:
- Async kôd izgleda sličnije tradicionalnom sync kôdu
- Ograničen na [novije preglednike](https://caniuse.com/#feat=async-functions)

---

## Async funkcije

- Označene su `async` prefiksom
- Vraćaju `Promise` objekt
- I arrow funkcije mogu biti async

<!-- 
Bilo koju običnu funkcije možemo pretvoriti u asinkronu ako joj dodamo `async` prefix
- ta funkcija će onda zapravo vratiti `Promise` koji je resolvan sa vraćenom vrijednošću
-->

---

## Await

- Ključna riječ `await` se može staviti ispred bilo kojeg **poziva** asinkrone funkcije
- Ako `await` koristimo unutar funkcije, ta funkcija **mora** biti označena kao asinkrona kroz `async`
- Pauzira izvođenje funkcije unutar koje se nalazi dok se vraćeni `Promise` ne ispuni

<!-- Ako koristimo await unutar ne-async funkcije dobiti ćemo sintaksnu grešku -->

---

<!-- _class: -->

## Promise chain napisan preko Async Await

```javascript
try {
  const user = await getUser('ldgit')
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
- [You're Missing the Point of Promises](https://blog.domenic.me/youre-missing-the-point-of-promises/) | Domenic Denicola
- [Promises/A+ spec](https://promisesaplus.com/)
