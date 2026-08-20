# FirebaseUI Auth for Web

FirebaseUI for Web (v7) provides composable authentication components for React,
Shadcn, and Angular, plus a framework-agnostic core package if you want to bring
your own UI. Use it when the app needs a working sign-in flow quickly; use
[client SDK setup](client_sdk_web.md) directly when the UI must be fully custom.

v7 is a complete rewrite. v6 lives on the `v6-archive` branch and has a separate
migration guide.

Canonical docs: https://github.com/firebase/firebaseui-web/blob/main/README.md

## 1. Install and Initialize

Install `firebase` plus the package for your framework, then pass a configured
Firebase App instance to `initializeUI`:

```bash
npm install firebase @firebase-oss/ui-react
```

```ts
import { initializeApp } from 'firebase/app';
import { initializeUI } from '@firebase-oss/ui-core';

const app = initializeApp({ /* your Firebase config */ });

const ui = initializeUI({ app });
```

Angular installs
`@angular/fire @firebase-oss/ui-angular @firebase-oss/ui-core @firebase-oss/ui-styles`
instead and requires AngularFire to be set up first. Shadcn pulls components
from a registry rather than a package — see the README.

## 2. Provide the UI Instance

**React / Shadcn**: wrap the app in `FirebaseUIProvider`.

```tsx
import { FirebaseUIProvider } from '@firebase-oss/ui-react';

function App() {
  return <FirebaseUIProvider ui={ui}>{/* your app */}</FirebaseUIProvider>;
}
```

**Angular**: add `provideFirebaseUI` alongside `provideFirebaseApp`.

```ts
import { provideFirebaseApp, initializeApp } from '@angular/fire/app';
import { initializeUI } from '@firebase-oss/ui-core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideFirebaseApp(() => initializeApp({ /* your Firebase config */ })),
    provideFirebaseUI((apps) => initializeUI({ app: apps[0] })),
  ],
};
```

## 3. Import the Styles

Required for React and Angular. Skip this for Shadcn — those components inherit
your Shadcn configuration.

```css
@import "@firebase-oss/ui-styles/dist.min.css";
/* Or, for Tailwind users */
@import "@firebase-oss/ui-styles/tailwind";
```

If your bundler cannot import CSS from `node_modules`, the README covers the CDN
option.

## 4. Render a Screen

```tsx
import { SignInAuthScreen } from '@firebase-oss/ui-react';

export function MySignInPage() {
  return <SignInAuthScreen onSignIn={() => { /* redirect */ }} />;
}
```

The Angular equivalent is
`<fui-sign-in-auth-screen (signIn)="onSignIn($event)" />` from
`SignInAuthScreenComponent`.

For the full component list, theming, behaviors, translations, and the
`FirebaseUIStore` core API, see the README linked above.
