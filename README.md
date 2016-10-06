## AngularFire 2 with Ionic

Create a new Ionic project

```
ionic start angularfire2-sample blank --v2
```

NPM install any dependencies needed

```
npm install angularfire2 --save
```

Import angularfire2's module in the `./src/app.module.ts`
```
...
import { AngularFireModule, AuthProviders, AuthMethods } from 'angularfire2';
...

@NgModule({
...
imports: [
    IonicModule.forRoot(MyApp),
    AngularFireModule.initializeApp(firebaseConfig, myFirebaseAuthConfig)
  ],
...
})
```

Run `npm run build`, observe the build error.

```
npm run build
```

You should see a lot of output about a namespace error:

```
...
Error at /Users/dan/Dev/angularfire2-test/node_modules/angularfire2/interfaces.d.ts:10:26: Cannot find namespace 'firebase'.
...
```

This is a known Angularfire2 issue. You can fix it easily by adding the following line to `./node_modules/angularfire2/angularfire2.d.ts`

```
import * as firebase from 'firebase';
```

Run `npm run build` to see the progress we've made

```
npm run build
```

Okay, all things `Typescript` are working now. We need to resolve the bundling issue with `Rollup`.

Let's pass a custom `Rollup` configuration.  Open `package.json` and create an entry off of the root node called `config`
with an entry for

```
...
"config": {
  "ionic_rollup": "./scripts/rollup.config.js"
}
...
```
This line above says we're going to provide our own custom rollup config.

Create a directory called `scripts` in the project root, and then create a file called `rollup.config.js`

Copy and paste the content from `./node_modules/@ionic/app-scripts/config/rollup.config.js` to `./scripts/rollup.config.js`


For fun and validation that we're using the correct rollup config, throw a log statement at the top of `./scripts/rollup.config.js`

```
console.log('Hello from the other side; I must have called 1000 times');
```

Run `npm run build` again. You should see the sweet lyrics of Adele logged. We've verified we're using the correct rollup config.

You should also see some errors that look something like this.

```
Module /Users/dan/Dev/angularfire2-test/node_modules/firebase/firebase-browser.js does not export initializeApp (imported by /Users/dan/Dev/angularfire2-test/node_modules/angularfire2/angularfire2.js
```

Let's fix them. Rollup, our next-gen bundler, produces much faster bundles, but it struggles a bit with `commonjs` libraries.

Most of the time, we have found that we can provide `namedExports` to fix this.

```
...
commonjs({
  namedExports: {
    'node_modules/firebase/firebase-browser.js': ['initializeApp', 'auth', 'database']
  }
}),
...
```

If you provide your own, newer dependency on Firebase, your commonjs plugin config could look like this:

```
...
commonjs({
  namedExports: {
    'node_modules/firebase/firebase-browser.js': ['initializeApp', 'auth', 'database'],
    'node_modules/angularfire2/node_modules/firebase/firebase-browser.js': ['initializeApp', 'auth', 'database']
  }
}),
...
```

Firebase has a known issue where it doesn't support strict mode, so disable that in rollup config as well off of the `rollupConfig` object.

```
...
  useStrict: false,
...
  plugins: [
```


Now that we've successfully been able to build the app, let's run `ionic serve` and test it out in the browser.

```
ionic serve
```

You now have a working `Angularfire2` instance.