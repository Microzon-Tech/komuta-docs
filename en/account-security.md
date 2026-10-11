# Account Security

This page covers where you manage the sign-in security of your Komuta account: your password, two-factor authentication, open sessions, security logs, signing in with Google and GitHub, and API keys. These settings are personal; they don't affect other members of your organization. Organization settings such as inviting members and assigning roles are on the [Organizations and Team](organizations-team.md) page.

All of them are in the **Personal** group of the **Account** page:

| Page | Address | What it's for |
|---|---|---|
| **Password** | https://console.komuta.io/account/password | Change your password. |
| **Two-Factor** | https://console.komuta.io/account/two-factor | Turn the authenticator app code at sign-in on or off. |
| **Sessions** | https://console.komuta.io/account/sessions | See the devices signed in to your account and close the ones you don't want. |
| **Security Logs** | https://console.komuta.io/account/security-logs | Review your account's sign-in and security events. |
| **External Logins** | https://console.komuta.io/account/external-logins | See and remove linked Google and GitHub sign-ins. |
| **API Keys** | https://console.komuta.io/account/api-keys | Create and revoke keys that let your own tools use the Komuta API. |

---

## Changing Your Password

1. Open **Account → Password**.
2. Type your current password in **Current Password**.
3. Type the new password twice, in **New Password** and **Confirm New Password**. The form asks for at least 6 characters and expects both fields to match.
4. Click **Reset Password**. When the change succeeds, you see **Password changed successfully** and the form clears.

If your current password is wrong, or the new password doesn't meet Komuta's password rules, you see **Failed to change password**. The message doesn't say why; check your current password first, then try a longer password.

> **Note:** If you created your account with Google or GitHub, you don't have a password yet. To set one, sign out and follow the **Forgot password?** steps below.

---

## If You Forgot Your Password

1. On the sign-in page, click **Forgot password?**.
2. Enter your account's email address and click **Send reset link**.
3. The **Check your email** page opens. It looks the same whether or not the address belongs to an account, so nobody can use it to find out whether an email address is registered with Komuta.
4. Click **Reset password** in the email you receive. The link is valid for 24 hours. If the email doesn't arrive, check your spam folder.
5. On the **Reset Password** page, type your new password twice and click **Reset my password**.
6. When you see "Your password is successfully reset.", sign in with **Login to the application**.

If the link has expired or is broken, the page says "This password reset link is invalid or has expired. Request a new link to continue." Start again with **Request a new reset link**.

### If your account gets locked

After many wrong passwords in a row, the account is locked for a while and the **Locked Out!** page opens. It shows when the lock ends, in UTC ("Your account will be unlocked on"). Try again after that time. If you aren't sure of your password, set a new one with **Forgot password?** once the lock has ended.

---

## Two-Factor Authentication

With two-factor authentication on, signing in with your email and password also asks for the 6-digit code from the authenticator app on your phone. Even if someone gets your password, they can't sign in without the code. It is optional and each user turns it on for themselves; it is off by default.

You can use Google Authenticator, Authy or any authenticator app that supports TOTP.

### Turning it on

1. Open **Account → Two-Factor**. The **Two-Factor Authentication** status is at the top of the page; while it's off, it says "Two-factor authentication is not enabled" with a **Not enabled** badge.
2. Scan the QR code in the **Authenticator Setup** section with your authenticator app. On a phone you can use the **Open in authenticator app** link. If you can't scan, type the key under **Or enter this key manually** into the app.
3. Type the 6-digit code shown in the app into **Enter the 6-digit code from your app to verify** and click **Verify**. You see **Authenticator verified**.
4. The **Recovery Codes** section opens. Copy the codes with **Copy All** and store them somewhere safe (see below).
5. If the status still shows **Not enabled**, click **Enable 2FA**. After **2FA enabled**, the status reads "Two-factor authentication is enabled" with an **Enabled** badge.

The **Enable 2FA** button doesn't appear until an authenticator app has been verified. If the code is wrong, you see an error; try again with the current code from the app.

### Recovery codes

When **Verify** succeeds, Komuta creates 10 recovery codes. If you lose access to your phone, you use one of them instead of the app code when you sign in. Each code works only once.

> **Warning:** Recovery codes are shown only right after **Verify**. Once you leave the page, they can't be shown again. If you lose them, run **Verify** again with the current code from the app; 10 new codes are created and the old ones stop working.

### The code at sign-in

1. On the sign-in page, enter your email address and password.
2. The **Two factor verification** page opens. In the **Selected provider** list choose **Authenticator** and continue. The list can also show **Email**; if you choose it, the code is sent to your email address.
3. Type the 6-digit code from the authenticator app and continue. If the page shows a **Remember this browser** box and you're on your own device, you can tick it; this browser won't be asked for the code again for a while.

### If you can't reach your phone

1. On the **Two factor verification** page, with **Authenticator** selected, click "Don't have access to your authenticator device? You can log in with a recovery code".
2. On the **Login with recovery code** page, type one of your codes into **Recovery code** and continue.
3. After signing in, set up the authenticator again on your new phone (the **Reset Authenticator** steps below) and save the new recovery codes.

If you've lost both your phone and your recovery codes and a Google or GitHub sign-in is linked to your account, you can sign in with it (see the note below). If not, contact Komuta support.

### Turning it off and resetting

- **Disable 2FA**: stops asking for the code at sign-in; you see **2FA disabled**. The authenticator app stays linked to your account, so you can turn it back on later with **Enable 2FA** without scanning again.
- **Reset Authenticator**: removes the app link and the recovery codes and creates a new QR code. Use it when you change phones or suspect someone else has the key. It takes effect right away without a confirmation, and you see **Authenticator reset**. The old Komuta entry in your app stops working; scan the new QR code and set it up again right away with **Verify**.

**Refresh QR Code** doesn't create a new key; it only reloads the information on screen.

> **Note:** The code is asked when you sign in with your email and password. When you sign in with **Continue with Google** or **Continue with GitHub**, Komuta doesn't ask for it; that sign-in relies on the security of your account at that provider. Turn on two-factor authentication in your Google and GitHub accounts as well.

---

## Active Sessions

The **Active Sessions** list on **Account → Sessions** shows the devices and browsers currently signed in to your account.

| Column | What it shows |
|---|---|
| **Device** | The type of sign-in, with browser or client details below it. The row of this browser is marked **Current**. |
| **IP Address** | The IP addresses the session came from. |
| **Last Accessed** | When the session was last used. |
| **Signed In** | When the sign-in happened. |

To close a session you don't recognize or no longer use, click **Revoke** on its row. There is no confirmation; you see **Session revoked**, and that device has to sign in again.

- The row marked **Current** has no **Revoke** button. To close this session, sign out.
- Sessions that aren't used for 7 days are closed automatically.
- API keys don't open sessions and don't appear in this list. To stop a key, revoke it on the **API Keys** page.

---

## Security Logs

The **Security Logs** list on **Account → Security Logs** shows the recent sign-in and security events recorded for your account. This is where you check for sign-ins you didn't expect.

| Column | What it shows |
|---|---|
| **Action** | The name of the event (see the table below). |
| **Application** | The Komuta component that recorded the event. |
| **IP Address** | The IP address the request came from. |
| **Date** | When the event happened. |

The page loads the newest 100 events and shows 10 per page. The "Search logs..." box searches the loaded events by action name, application and IP address. Click a column header to sort.

Events you can see in the **Action** column:

| Action | Meaning |
|---|---|
| `LoginSucceeded` | Successful sign-in. |
| `LoginFailed` | Failed sign-in attempt with a wrong password. |
| `LoginRequiresTwoFactor` | The password was correct and the two-factor code was requested. |
| `LoginLockedout` | Sign-in refused because the account was locked. |
| `LoginNotAllowed` | Sign-in not allowed, for example because the email address isn't confirmed yet. |
| `Logout` | Signed out. |
| `ChangePassword` | The password was changed on the **Password** page. |
| `ChangeEmail` | The email address was changed in the profile. |
| `ExternalLoginRegister` | The account was created with Google or GitHub. |
| `ExternalLoginLink` | A Google or GitHub account was linked to your account the first time it was used to sign in. |
| `GitHubLinked` | A GitHub account was linked to your account from the console. |

If you see `LoginSucceeded` from an IP address you don't recognize, change your password right away, revoke the sessions you don't recognize on the **Sessions** page and review your API keys.

Turning two-factor authentication on or off, removing an external login, and creating or revoking API keys don't appear in this list. Sign-in events for all members of the organization are on the **Login Activity** page of the [Security Center](security-center-guide.md), if you have access to it.

---

## External Logins: Google and GitHub

Besides email and password, you can sign in to Komuta with your Google or GitHub account. The **External Logins** list on **Account → External Logins** shows the providers linked to your account. Each row shows the provider's name and your account ID at that provider. When nothing is linked, it says **No external logins linked**.

### Linking

There is no separate link button in the console. When you sign in on the sign-in page with **Continue with Google** or **Continue with GitHub**, and the provider's verified email address is the same as your Komuta account's email address, the provider is linked to your account automatically. You can use the same button for later sign-ins.

If linking isn't possible, the sign-in page shows one of these messages:

| Message | What to do |
|---|---|
| "This external account is already linked to another Komuta account. Sign in with that account instead." | This Google/GitHub account is used by another Komuta account. Sign in with that account, or remove it there. |
| "We could not get a verified email address from your provider. Verify your primary email address with the provider and try again." | Verify your primary email address at the provider and try again. |
| "An unverified account already uses this email address. …" | The account opened with this address isn't confirmed yet. First click the link in that account's confirmation email. |

On this page you may also see the **Free plan Git account** card. If the card shows a **Link GitHub** button, the GitHub account you link is tied to the organization's free plan; if it isn't already a sign-in method, you confirm on GitHub and it is also added to your account as one. For the free plan rules, see [Billing](billing.md).

### Removing

1. Click **Remove** on the row of the provider you want to remove.
2. The **Remove external login** dialog says the provider will be unlinked from your account. Confirm with **Remove**.
3. You see **External login removed**.

> **Warning:** If you created your account with Google or GitHub and never set a password, set one with **Forgot password?** before removing it. Also, if you sign in again on the sign-in page with a provider you removed and its email address matches your account, it is linked again. If you think one of your Google or GitHub accounts is compromised, secure that account at the provider first.

---

## API Keys

An API key lets your own agent, scripts or CI jobs use the Komuta API (`https://api.komuta.io`) on your behalf. The key acts as your user:

- It works in the organization where you created it, with your current role and permissions. If your role changes, what the key can do changes too.
- It is valid only in that organization. For another organization, switch to it and create a separate key there.
- It can't be used to sign in to the console; it doesn't replace your password.

If you only want to deploy one service from CI, use a **deploy token** instead of an API key: it is bound to that service and limited to what you allow, see [Deploying from CI](ci-deploy.md). To connect Claude or Codex to the Komuta MCP, signing in through the browser is enough, see [Installing the Komuta MCP Server](mcp-setup.md).

### Who can create keys

With the default roles, creating and revoking keys is open to organization admins. Members see the **API Keys** page but not the **Create API Key** button. For roles, see [Organizations and Team](organizations-team.md).

### Creating a key

1. On **Account → API Keys**, click **Create API Key**.
2. In **Name**, type a name that says where the key will be used (up to 128 characters), for example "CI pipeline".
3. Choose a duration in the **Expiry** list (see the table below).
4. Click **Create Key**.
5. In the **API Key Created** dialog, copy the key and store it somewhere safe, then close with **Done**.

> **Warning:** The key is shown only once, in this dialog, and can't be shown again. If you lose it, create a new key and revoke the old one.

| **Expiry** option | How long the key actually stays valid |
|---|---|
| **No expiry** (selected by default) | 90 days. Keys without an expiry are not created. |
| **30 days** | 30 days |
| **90 days** | 90 days |
| **1 year** | 365 days. This is the longest possible. |

An expired key can't be extended; create a new one.

### Using a key

A key looks like `kmt_<key id>_<secret>`. Send it in the `Authorization` header with `Bearer`:

```bash
curl -H "Authorization: Bearer $KOMUTA_API_KEY" https://api.komuta.io/...
```

Copy the key exactly as shown; the `kmt_` prefix must be lowercase. A request with an invalid, expired or revoked key is refused with `401`.

### The list

| Column | What it shows |
|---|---|
| **Name** | The key's name, with the `kmt_<key id>…` start below it. The secret part is never shown. |
| **Status** | **Active**, **Expired** or **Revoked**. |
| **Used** | When the key was last used, or **Never used**. Updated at most every 5 minutes. |
| **Created** | When the key was created. |
| **Expires** | When the key stops being valid. |

The search box at the top filters by name and key ID.

### Revoking a key

1. Open the menu on the row of an active key and click **Revoke**.
2. In the **Revoke API Key** dialog, confirm with **Revoke Key**.

The key stops working immediately, and every tool using it loses access. Revoking can't be undone. Revoked and expired keys stay in the list with their status.

A key also stops working when it expires, when your user is deactivated, or when you are removed from the organization.

---

## Good Practices

- Choose a long password for Komuta that you don't use anywhere else, and keep it in a password manager.
- Turn on two-factor authentication, and keep the recovery codes somewhere other than your phone.
- If you sign in with Google or GitHub, turn on two-factor authentication in those accounts too.
- Check the **Security Logs** and **Active Sessions** lists regularly, and revoke sessions you don't recognize.
- On shared or public computers, don't tick **Remember this browser**, and sign out when you're done.
- Create a separate API key for each tool or pipeline, with a name that says what it's for, and pick the shortest duration that does the job.
- Don't put API keys in code or in a repository; keep them in your CI's secret variables or a secrets manager.
- Revoke keys right away when they're unused, when you suspect a leak, or when their owner leaves the team.
- To deploy a service from CI, use a [deploy token](ci-deploy.md) instead of an API key.

---

## Related Documents

- [Organizations and Team](organizations-team.md) — Members, roles and invitations.
- [Security Center](security-center-guide.md) — Organization sign-in activity and audit records.
- [Deploying from CI](ci-deploy.md) — Deploy tokens bound to a single service.
- [Installing the Komuta MCP Server](mcp-setup.md) — Connecting Claude and Codex to Komuta.
- [Support](support-tickets.md) — Opening a request with Komuta support.
- [Quick Start](quick-start.md) — Creating an account and first steps.
