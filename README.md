# TEST
# RxFire

Firebase and RxJS for all frameworks.

## What is RxFire?

- **Observable creators** - Observables bindings for most Firebase web libraries.
- **Portable** - Use across any framework or no framework at all.
- **Tree shake-able** - Import only what you need. Shake the rest out with your favorite module bundler like Webpack or Rollup.
- **Combine multiple data sources** - Need to join two Firestore references? Want to combine an image from Cloud Storage with a Firestore document? Super easy with observables and operators.
- **Simplify code-splitting of Firebase** - Using RxFire with Webpack makes it easy to load Firebase features on-demand.

Status: Beta

## Install

```bash
# npm
npm i rxfire firebase rxjs --save
# yarn
yarn add rxfire firebase rxjs
```

Make sure to install Firebase and RxJS individually as they are peer dependencies of RxFire.

## Example use:

```ts
import { initializeApp } from 'firebase/app';
import { getFirestore, collection, where, query } from 'firebase/firestore';
import { collectionData } from 'rxfire/firestore';
import { tap } from 'rxjs/operators';

const app = initializeApp({ /* config */ });
const firestore = getFirestore(app);
const citiesRef = query(
    collection(firestore, 'cities'),
    where('state', '==', 'CO')
);

collectionData(citiesRef, { idField: 'id' })
  .pipe(
    tap(cities => console.log('This is just an observable!'))
  )
  .subscribe(cities => { /* update UI */ })
```

## Easily combine multiple Firebase data sources

RxJS provides multiple operators and creation methods for combining observable streams. This makes it easy to combine data from multiple Firebase resources. You can also handle simplify high asynchronous tasks like joins into a flat stream.

The example below streams a list of "cities" from Firestore and then retrieves their image from a Cloud Storage bucket. Both tasks are asynchronous but RxJS makes it easy to combine these tasks together.

```ts
import { initializeApp } from 'firebase/app';
import { getStorage, ref } from 'firebase/storage';
import { getFirestore, collection, where, query } from 'firebase/firestore';
import { collectionData } from 'rxfire/firestore';
import { getDownloadURL } from 'rxfire/storage';
import { combineLatest } from 'rxjs';
import { switchMap } from 'rxjs/operators';

const app = initializeApp({ /* config */ });
const firestore = getFirestore(app);
const storage = getStorage(app);
const citiesRef = query(
    collection(firestore, 'cities'),
    where('state', '==', 'CO')
);

collectionData(citiesRef, { idField: 'id' })
  .pipe(
    switchMap(cities => {
      return combineLatest(...cities.map(c => {
        const ref = ref(storage, `/cities/${c.id}.png`);
        return getDownloadURL(ref).pipe(map(imageURL => ({ imageURL, ...c })));
      }));
    })
  )
  .subscribe(cities => {
    cities.forEach(c => console.log(c.imageURL));
  });
```

## Understanding RxFire imports

RxFire is a complementary library to Firebase. It is not meant to wrap the entire Firebase SDK. RxFire's purpose is to simplify async streams from Firebase. You need to import the Firebase SDK and initialize an app before using RxFire.

```ts
import { initializeApp } from 'firebase/app';
import { getStorage, ref } from 'firebase/storage';
import { getDownloadURL } from 'rxfire/storage';

const app = initializeApp({ /* config */ });
const storage = getStorage(app);
const ref = ref(storage, 'data.json');

// Now you can use RxFire!
const url$ = getDownloadURL(ref);
```

RxFire contains multiple entry points for module imports. Each Firebase library is an entry point.

```ts
import { } from 'rxfire/firestore';
import { } from 'rxfire/database';
import { } from 'rxfire/storage';
import { } from 'rxfire/auth';
import { } from 'rxfire/functions';
import { } from 'rxfire/performance';
import { } from 'rxfire/remote-config';
```

## Simple functions
RxFire is a set of functions. Most functions create observables and from there you can use regular RxJS operators. Some functions are custom operators. But at the end of the day, it's all just functions. This is important for **tree shaking**. Any unused functions are stripped from your final build if you use a module bundler like Webpack or Rollup.

```ts
import { initializeApp } from 'firebase/app';
import { getStorage, ref } from 'firebase/storage';
import { getDownloadURL, put /* not used! */ } 'rxfire/storage';

const app = initializeApp({ /* config */ });
const storage = getStorage(app);
const ref = ref(storage, 'data.json');

const url$ = getDownloadURL(ref);
```

## Documentation

- [Firestore](docs/firestore.md)
- [Authentication](docs/auth.md)
- [Storage](docs/storage.md)
- [Realtime Database](docs/database.md)
- [Cloud Functions](docs/functions.md)

## Examples

[Examples](https://github.com/davideast/rxfire-samples): See this example repository for a list of ways to configure and use RxFire.
