# Organizations and Team Members

Everything in Komuta lives inside an organization: your services, managed services, wallet and spending limit all belong to it. This page explains how to create organizations, switch between them, invite teammates and decide who can manage what.

---

## What an organization is

An organization is your workspace in Komuta. The following belong to the organization and are not visible from any other organization:

- Projects and apps, managed services, domains, integrations (Git account and registry connections) and alerts.
- The wallet, the spending limit, the spending tier and statements.
- The members and each member's permissions.

Signing up creates an organization that belongs to you, with you as its admin. Its name comes from your email address (for example `ayse-3f9c2a1b`); use **Rename** to give it the name you want.

| Rule | Details |
|---|---|
| Number of organizations | Once your account is in 5 organizations, you cannot create another one. Organizations you created and organizations you joined by invitation count together. |
| Welcome credit | Only the first organization, the one created at sign-up, gets it: USD 10, valid for 30 days. Organizations you create later get no credit. |
| Spending limit | Each organization has its own limit. A new organization also starts with a USD 20 monthly limit. |
| Free plan | One organization per person. The personal Git account (GitHub, GitLab or Azure DevOps) that owns a free service's repo can be tied to only one organization; other organizations use paid plans. |

> **Note:** Integrations belong to the organization too. Before you deploy a service in a new organization, connect your Git account on that organization's **Integrations** page.

---

## The Organizations page

The **Account → Organizations** page (**My Account** in the user menu at the top right) is open to organization admins. It shows:

- At the top, the organization you are in, labeled **Current organization**. **Rename** next to it changes the organization's display name in the console. The name must be 2 to 64 characters.
- Below it, the other organizations you belong to, each with a **Switch** button.
- The **Create organization** button.
- The organization's access protection settings: **Allow external sharing**, **Verified domains**, **Visitor sign-in with your identity provider**, **Protect new services** and **Keep access logs for**. These are covered in [Access Protection](service-access-protection.md).

Members cannot see this page; they switch organizations from the user menu.

---

## Creating an organization

1. Open the user menu at the top right, click the row that shows your organization's name, and choose **Create organization**. The same button is on the **Account → Organizations** page.
2. In the **Create a new organization** window, type a name in **Organization name (optional)** (up to 64 characters). If you leave it empty, the name comes from your email address.
3. Click **Create**.

The organization is created and the console switches you into it right away; you are its admin. If the switch fails, the button becomes **Retry switch**. It does not create a second organization; it only tries again to switch into the one that was created.

You need a confirmed email address to create an organization. If yours is not confirmed, the window shows: "Verify your email address before creating an organization. Open the confirmation link we sent to your inbox, then try again."

> **Note:** You only see **Create organization** when you are an admin of the organization you are in. If you are a member of someone else's organization, switch to your own organization first.

---

## Switching between organizations

If you belong to more than one organization, you can switch from two places:

- **User menu (everyone):** Open the user menu at the top right and click the row that shows your organization's name. Click the one you want under **Other Organizations**.
- **Account → Organizations (admins):** Click **Switch** next to the organization in the list.

While switching you see "Switching organization...". The console then reloads and opens the home page in the new organization; menus, lists and your permissions now belong to that organization.

> **Note:** The list shows the organizations directly linked to the one you are in. In your first organization, the one created at sign-up, you see every organization you created there and every organization you joined by invitation. In any other organization the list shows only the organizations linked to it, usually your first organization. To move to a different one, go back to your first organization first. To keep them all in one list, create new organizations while you are in your first organization.

---

## The Access Control page

Teammates, roles and permissions are managed on **Administration → Access Control** in the left menu ([console.komuta.io/access-control](https://console.komuta.io/access-control)). Only organization admins see this page.

- At the top, three numbers: **Total users**, **Active** and **Administrators**.
- On the left, the **Users** list: a search box, the **Invite member** button and an **Admin** or **Member** badge next to each person. Your own row says **You**.
- On the right, the selected person's role, resource permissions and **Module capabilities**; at the bottom, **Pending invitations**.

---

## Inviting a teammate

1. Open **Administration → Access Control**.
2. On the **Users** card, click **Invite member**.
3. Type the person's address in **Email**.
4. Choose **Member** or **Admin** as the **Role**. The default is **Member**.
5. Click **Send invitation**.

You see "Invitation sent" and the invitation is added to **Pending invitations**. The person receives an invitation email.

- An invitation is valid for 72 hours. The person is not part of the organization until they accept.
- You do not set a password for them; they accept with their own Komuta account, or create one while accepting.
- If you invite the same address again, the earlier pending invitation is revoked; only the link in the latest email works.
- If the organization already has a user with this email address, no invitation is sent.

---

## What the invited person sees

1. **Email.** The subject reads "You're invited to join {organization} on Komuta". The email shows who invited them, an **Accept invitation** button and the date the invitation expires (UTC).
2. **Join page.** The link opens a page titled "Join {organization}". It shows the role they were invited with (**Member** or **Admin**) and the address the invitation was sent to.
   - Someone with a Komuta account clicks **Sign in to accept**.
   - Someone without an account clicks **Create an account to accept**.
3. **Sign in or sign up.** The sign-in and sign-up pages show the banner "You're accepting an invitation to {organization}." The email field is filled with the invited address and cannot be changed. Continuing with Google or GitHub works too, as long as that account's email address is the invited address.
4. **Accept.** Once signed in, the page shows **Accept invitation**. **Not now** goes to the console without accepting.
5. **Console.** After accepting, the console opens. The person starts in their own organization; the organization they joined appears under **Other Organizations** in the user menu, and they switch to it to work there.

Good to know:

- Someone who signs up with email and password from the invitation link skips the email confirmation step: the invited address counts as confirmed.
- Like everyone who signs up, someone who creates an account through an invitation also gets an organization of their own.
- Only the owner of the invited address can accept. If the person is signed in with another address, the page says "This invitation is for {address}, but you're signed in as {other address}."; **Sign in with a different account** switches to the right one. A forwarded link cannot be used with a different account.
- If the invitation has expired, was revoked or was resent, the old link opens a page titled **We couldn't accept this invitation**. The person should ask an admin for a new invitation.

> **Note:** The invitation email and the join page show the organization by the short name it was created with (for example `ayse-3f9c2a1b`), not by its display name in the console. If you renamed your organization, tell the person you invite.

---

## Pending invitations

The **Pending invitations** card at the bottom of the **Access Control** page lists invitations that have not been accepted and have not expired: **Email**, **Roles**, **Status**, **Expires** and **Invited by**. From the menu at the end of a row:

| Action | What happens |
|---|---|
| **Resend** | A new link is sent and the 72 hours start again. The link in the earlier email stops working. |
| **Revoke** | The invitation becomes invalid right away; the link can still be opened but not accepted. |

An accepted invitation leaves this list and the person appears in the **Users** list. An expired invitation leaves the list too; to invite the same person again, use **Invite member**.

---

## Roles: Member and Admin

Every person in an organization has one of two roles:

- An **Admin** has full access to every resource, manages members and permissions, and sees the billing pages.
- A **Member** only reaches the resources an admin grants. A new member sees empty project and app lists until they are given access.

| Action | Member | Admin |
|---|---|---|
| View projects, apps, managed services, domains, integrations and alerts | Only the ones granted **Read** or **Edit** | All of them |
| Change and delete them | Only the ones granted **Edit** | All of them |
| Create new projects, apps, managed services, domains, integrations or alerts | If creating that type is allowed | Yes |
| View environment groups | Yes | Yes |
| Create, edit and delete environment groups | If the module capability is granted | Yes |
| Service security settings, access protection and shares, advanced observability, API gateway callers | If the module capability is granted | Yes |
| Create, plan and apply Stacks | No | Yes |
| Create and edit notification channels | No | Yes |
| Create API keys | No | Yes |
| Connect an MCP agent to this organization | No | Yes |
| Open support tickets | No | Yes |
| **Wallet**, **Current Spend**, **Statements**, **Payment Methods** and **Billing Profile** | No | Yes |
| Invite members, change roles, remove members and grant permissions | No | Yes |
| Rename the organization and the **Account → Organizations** settings | No | Yes |
| Create a new organization while in this one | No | Yes |

When a member creates a resource with a create permission, they get **Edit** on that resource automatically.

---

## Giving a member access to resources

1. On the **Access Control** page, select the member in the **Users** list.
2. Pick a resource type from the tabs at the top: **Projects**, **Apps**, **Integrations**, **Domains**, **Alerts** or **Managed Services**.
3. In the table, tick **Read** or **Edit** for each resource. Clicking a column header ticks or clears every resource shown in the list.
4. If the member should be able to create new resources of this type, turn on **Allow this user to create new …** above the table.
5. Click **Save**. You see "Permissions saved".

- **Read** is view-only. **Edit** also covers changing and deleting, and turns **Read** on as well. Clearing **Read** clears **Edit** too.
- On the **Apps** and **Managed Services** tabs, resources are grouped by project; the box on a project row ticks every resource in that project at once. Access to a project does not give access to the apps in it. Resources added to the project later need their own access.
- If you move to another tab or person without saving, you are asked **Discard unsaved changes?**
- If the selected person is an admin, the table and switches are locked; admins already reach every resource.

---

## Module capabilities

The **Module capabilities** card below the resource table grants permissions that are not tied to a single resource. Each switch is saved as soon as you change it.

| Module | Capabilities you can grant |
|---|---|
| **Service Security** | View security posture, Manage Linux capabilities, Manage writable mounts, Manage run as root, Manage access protection, Manage access shares |
| **Advanced observability** | View observability, Edit SLOs, View anomalies, Natural-language query, Edit custom dashboards, Activate subscription |
| **Environment Groups** | View environment groups, Create environment groups, Edit groups and their variables, Delete environment groups |
| **API Gateways** | Manage API gateway callers and usage plans |

These capabilities work on resources the member can already reach. For example, to push an environment group into an app the member also needs **Edit** on that app, and API gateway callers can only be managed on gateways the member can see.

---

## Changing a role and removing a member

**To change a role:**

1. On the **Access Control** page, select the person.
2. In the role list next to their name, choose **Member** or **Admin**.
3. In the window that opens, click **Change role**.

A person made admin gets full access to every resource and can manage members. When an admin is made a **Member**, their access is reset: every resource grant and module capability they hold is removed; grant what they need again afterwards.

**To remove someone from the organization:** Select the person, click **Remove from organization** and confirm with **Remove from organization** in the window. They lose access immediately and are signed out of this organization. Their own Komuta account and their own organization stay; the organization they were removed from disappears from their list. You can invite them again later.

Two rules apply:

- You cannot change your own role or remove yourself. Ask another admin to do it.
- The organization must keep at least one active admin. For the only active admin, the card says "This is the only active admin. Make someone else an admin first."

---

## Leaving or deleting an organization

You cannot leave an organization yourself in the console. Ask an admin of that organization to remove you with **Remove from organization**. If you are the organization's only admin, make someone else an admin first.

The console has no way to delete an organization. If you need to close one, open a [support ticket](support-tickets.md).

---

## Billing and limits when you work in another organization

Billing and limits belong to the organization, not to the person. When you switch into someone else's organization:

- Everything you create there is paid from that organization's wallet and counts toward that organization's spending limit. Your own organization's balance and limit are not used.
- When you create something or move it to a bigger plan, that organization's limit is checked.
- The free plan rules follow that organization too; each organization can run one service on the free package.
- Only that organization's admins see the billing pages and the balance card in the user menu.

Prices, spending tiers and how the limit goes up are covered in [Billing](billing.md) and [Limits](limits.md).

---

## Sharing services with other organizations

You can open a service that has access protection to everyone in another organization you belong to with an **A linked organization** share. This is an external share and needs **Allow external sharing** turned on in your organization. See [Sign-in and Sharing](access-protection-sign-in-sharing.md) for details.

> **Note:** **Access Control** decides who can see and manage which resources in the Komuta console. Who can open a deployed service's web address is a separate matter, set with [Access Protection](service-access-protection.md). A service's shares decide whether a person can open a protected service; if the service is shared with **Your organization**, every active member of the organization can open it.

---

## Related Documents

- [Quick Start](quick-start.md) — Signing up, your first organization and your first service.
- [Access Protection](service-access-protection.md) — Who can open a deployed service, and the organization settings.
- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — Sharing services with people, organizations and domains.
- [Billing](billing.md) — Wallet, statements and payments.
- [Limits](limits.md) — The spending limit and per-organization limits.
- [Environment Groups](environment-groups.md) — Creating groups and pushing them into apps.
