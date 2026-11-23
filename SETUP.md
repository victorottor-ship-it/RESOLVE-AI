# RESOLVE AI — Firebase + Gemini starter

This folder contains the RESOLVE AI web application and its Firebase backend. It is a real deployable starter, not a live service: you must connect your own Firebase project and Gemini API key, then deploy it.

## What is included

- Responsive, mobile-first RESOLVE AI interface.
- Firebase email/password account creation and sign-in.
- Firestore investigation history, stored under each signed-in user's UID.
- Firebase Cloud Function that verifies the user's Firebase ID token before requesting an AI response.
- Gemini API key stored as a Firebase server-side secret, never in browser JavaScript.
- PWA manifest and service worker so the deployed site can be installed from Chrome on Android.

## Important: opening the files locally

You can open `public/index.html` in a browser to preview the layout. The CSS and image paths are relative so the visual design can load from a local folder. However, a local `file://` preview is not a working application: browser module restrictions prevent some Firebase imports, and login, Firestore, the AI endpoint and PWA installation require the deployed HTTPS site.

Do not open `app.js` as the application. `index.html` is the entry page. Edit the source files with a code editor, then deploy the project.

## 1. Create and configure Firebase

1. Open the Firebase Console: https://console.firebase.google.com/
2. Create a Firebase project.
3. In Project settings, add a Web App.
4. Copy the web app configuration object.
5. Open `public/app.js` in a code editor and replace the placeholder values in `firebaseConfig` with the values from your own Firebase project. Keep the object syntax and quotation marks.
6. In Firebase Console → Authentication, click **Get started** if prompted. Then open **Sign-in method** and enable **Email/Password**. The Firebase web-app config in `public/app.js` must come from a web app registered in this same Firebase project.
7. In Firestore Database, create a database. Deploy this project's `firestore.rules` file with the Firebase CLI as described below.

The Firebase web configuration is intended to be present in client-side code. Access control must be enforced by Firebase Authentication and Firestore Security Rules. Never put a Gemini API key or other private server credential in `app.js`.

## 2. Create a Gemini API key

1. Open Google AI Studio: https://aistudio.google.com/app/apikey
2. Create an API key for a project you control.
3. Keep the key private. Do not paste it into this chat, `app.js`, GitHub, or any public file.

This starter currently calls the `gemini-3.5-flash-lite` model through the Gemini API. Model availability, quotas and pricing can change; check the current Google AI documentation before launch: https://ai.google.dev/gemini-api/docs

## 3. Deploy the backend and website

A computer is recommended for this step. The project uses Firebase CLI and Cloud Functions; doing the full deployment from an Android phone can be difficult.

Install Node.js 20 or newer and Firebase CLI, then open a terminal in the project folder:

```bash
npm install -g firebase-tools
firebase login
firebase use --add
```

Choose the Firebase project you created. Install the Cloud Function dependencies:

```bash
cd functions
npm install
cd ..
```

Set the Gemini key as a Firebase secret. The CLI prompts for the value; paste it into the prompt, not into a source file:

```bash
firebase functions:secrets:set GEMINI_API_KEY
```

Deploy Hosting, Firestore rules and the function:

```bash
firebase deploy
```

Firebase Hosting will display an HTTPS website address when deployment succeeds. The `/api/investigate` path is routed to the authenticated Cloud Function.

**Billing:** Firebase Cloud Functions deployment generally requires the Firebase project to use the Blaze billing plan. Cloud Functions, network use and Gemini API usage may incur charges. Review current pricing and set budget alerts before deploying. A budget alert is not a hard spending cap.

## 4. Test the application

1. Open the Firebase Hosting HTTPS address in Chrome.
2. Choose **Create account** and register with an email you control.
3. Sign in.
4. Submit a non-sensitive test problem with at least 20 characters.
5. Confirm that the investigation report appears and is saved under Recent investigations.
6. Refresh the page and confirm the history still appears.

If sign-in fails with `auth/configuration-not-found`, open Firebase Console → Authentication for the project in `public/app.js`, click **Get started** if prompted, and enable **Email/Password**. Also confirm that the web-app config belongs to that same project. For `auth/unauthorized-domain`, add the website's hostname under Authentication → Settings → Authorized domains.

If the AI request fails, check that the `GEMINI_API_KEY` secret was set, that the Gemini API key is valid, that the model is available to your project, and that billing/quota limits have not been reached. Check Cloud Functions logs for server-side details. Never expose the key in browser logs or source code.

## 5. Install on Android without Play Store

After the HTTPS site is deployed and working:

1. Open the Firebase Hosting address in Chrome on Android.
2. Open Chrome's three-dot menu.
3. Tap **Install app** or **Add to Home screen** (the wording varies by Chrome version).
4. Confirm. RESOLVE AI will appear on the home screen and open in an app-style window.

This is a Progressive Web App (PWA), not a native APK. It does not require Play Store or Android Studio. The app shell may be cached, but sign-in, saved history and AI investigations require internet access.

## Security and launch checklist

This is an early-stage starter, not an audited production service. Before allowing public users:

- Configure Firebase App Check and enforce it where supported.
- Add per-user request quotas and abuse/rate limiting to the AI endpoint.
- Add account email verification, password reset, account deletion and data export flows.
- Publish a clear privacy notice and explain what data is sent to the AI provider.
- Review Firebase and Gemini billing, quotas, logs and retention settings.
- Test Firestore rules and backend authorization.
- Add monitoring and a support/contact route.

The AI creates suggestions and a plan. It does not send messages, spend money, change accounts, or perform external actions. Review its output before acting, especially for high-stakes decisions.
