# Ngiti CRM: Connect to Facebook Messenger

Do the parts in order. Part 1 and 2 put your CRM online with a login and a saving database.
Part 3 connects Messenger. Take your time. Each part is small.

What you need: a Google account, a GitHub account (free), a Netlify account (free),
and admin access to your clinic's Facebook Page.

---

## Part 1. Firebase (database + staff login)

1. Go to https://console.firebase.google.com and click **Create a project**. Name it `ngiti-clinic`. You can turn Google Analytics off.
2. Left menu: **Build > Firestore Database > Create database**.
   - Location: pick `asia-southeast1` (Singapore). This cannot be changed later.
   - Choose **Production mode**.
3. Left menu: **Build > Authentication > Get started > Email/Password > Enable > Save**.
4. In Authentication, open the **Users** tab and click **Add user**. Add an email and password for each staff member who will use the CRM.
5. Open the file `firestore.rules` from this folder. Replace the sample emails with your staff emails (the same ones as step 4, all lowercase).
   Then in Firestore click the **Rules** tab, paste the whole file, and click **Publish**.
6. Click the gear icon > **Project settings**. Scroll to **Your apps** and click the web icon `</>`. Name it `crm` and register it.
   Copy the four values (apiKey, authDomain, projectId, appId) into the file `firebase-config.js`.
7. Still in Project settings, open the **Service accounts** tab and click **Generate new private key**. A small file downloads. Keep it private. Never upload it to GitHub.
   Open it in a text editor. You need three values for Part 2:
   - `project_id`
   - `client_email`
   - `private_key` (the long text between the quotes, including the `\n` marks)

## Part 2. Put the CRM online (GitHub + Netlify)

1. On https://github.com click **New repository**. Name it `ngiti-crm`. Choose **Private**.
2. Click **uploading an existing file** and drag in everything from this folder
   (`index.html`, `firebase-config.js`, `netlify.toml`, `package.json`, `firestore.rules`, and the `netlify` folder). Click **Commit changes**.
3. On https://app.netlify.com click **Add new site > Import an existing project > GitHub**, and pick `ngiti-crm`. Leave the build settings as they are.
4. Before you deploy, open **Site configuration > Environment variables** and add these:

| Name | Value |
|---|---|
| `FIREBASE_PROJECT_ID` | `project_id` from the key file |
| `FIREBASE_CLIENT_EMAIL` | `client_email` from the key file |
| `FIREBASE_PRIVATE_KEY` | `private_key` from the key file (keep the `\n` marks) |
| `ALLOWED_EMAILS` | your staff emails, separated by commas |
| `META_VERIFY_TOKEN` | any secret word you invent, for example `ngiti-verify-2026` |

   You will add two more values in Part 3 (`META_APP_SECRET` and `META_PAGE_TOKEN`).
5. Click **Deploy**. When it finishes, open your site link (`something.netlify.app`).
   You should see a sign-in screen. Sign in with a staff email from Part 1, step 4.
6. Test: in the CRM, open the **Facebook** tab and press **Try it with a simulated message**. A new chat should appear. Refresh the page and the chat should still be there. That means saving works.

## Part 3. Connect Meta

1. Go to https://developers.facebook.com > **My Apps > Create App**. Choose the **Business** type, and name it `Ngiti CRM`.
2. In the app, **Add product > Messenger > Set up**.
3. In **Messenger > API setup**, under access tokens, click **Add or remove Pages** and pick your clinic's Page. Then click **Generate token** and copy it.
   In Netlify add it as `META_PAGE_TOKEN`.
4. Go to **App settings > Basic**. Click **Show** next to **App secret** and copy it.
   In Netlify add it as `META_APP_SECRET`.
5. In Netlify click **Deploys > Trigger deploy > Deploy site**, so the new values are picked up.
6. Back in Meta, **Messenger > API setup > Configure webhooks**:
   - Callback URL: `https://YOUR-SITE.netlify.app/webhook`
   - Verify token: the same secret word you used for `META_VERIFY_TOKEN`
   - Click **Verify and save**.
7. Under **Webhook fields**, subscribe the Page to **messages**.
8. Test: with a Facebook account that has a role on your app (you are one automatically), send a message to your Page.
   Keep the CRM open and signed in. The chat appears in the Inbox within a few seconds, and the automatic reply goes back on Messenger.

## Part 4. Open it to real patients

Until Meta approves your app, only people with a role on the app can message it.

1. In the app, open **App Review > Permissions and features**.
2. Request **pages_messaging** (and **pages_utility_messaging** if Meta lists it).
   Meta will ask for a short screen recording that shows your CRM receiving and answering a message, and a privacy policy link.
3. Meta may also ask you to complete **Business verification**.
4. After approval, switch the app from **Development** to **Live** mode.

Meta changes these steps often. Follow their screens if they look a little different.

---

## Good to know

- **Keep the CRM open.** New Messenger messages wait in a queue and are scored when a signed-in staff browser has the CRM open. If nobody has it open, patients get no automatic reply until someone opens it.
- **24-hour rule.** You can reply on Messenger only within 24 hours of the patient's last message. After that the CRM tells you to call or text instead.
- **Staff only.** The sign-in and the rules in `firestore.rules` block everyone who is not on your list. Keep that list up to date.
- **Privacy.** Patient messages are personal data. Do not share the Netlify site's admin access or the Firebase key file.
- **Graph API version.** The server uses `v23.0`. If Meta retires it, add an environment variable `GRAPH_VERSION` with a newer version, for example `v24.0`.
