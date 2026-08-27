# Roles & permissions — what's here and what isn't

## Setup status: deployed and verified (2026-08-27)

`firestore.rules` is published. Verified directly against live Firestore:
profile auto-creation on first login, role storage, and real enforcement —
a plain "user" account was confirmed unable to list all users, self-promote
to admin, self-unban, or create an invite (all correctly rejected with
`permission-denied`). `notes_saves`, `saves`, and `ctmwbc_saves` self-access
all still work, so Mining Sim and CTMWBC are unaffected.

**This Firebase project is shared with Mining Sim and CTMWBC.** Publishing
rules *replaces* the whole ruleset, not merges it — `firestore.rules` in
this repo includes their existing `saves/{userId}` and
`ctmwbc_saves/{userId}` rules unchanged specifically so publishing it
doesn't break those apps' cloud save. If you ever hand-edit rules directly
in the console instead of pasting this file, make sure those two blocks are
still in there before you publish.

What hasn't been verified yet (needs the real owner account, which I don't
have credentials for): signing in as miss.scumerica@gmail.com and
confirming the Manage Users panel actually lists every account, role
changes/bans take effect for other users, and the owner-only invite flow
works end to end.

1. Go to the [Firebase Console](https://console.firebase.google.com/) → project `scumericas-mining-sim`
2. Firestore Database → **Rules** tab
3. Replace the contents with `firestore.rules` from this repo
4. Click **Publish**

Until this is done: the Manage Users panel will show a permission error, and
new signups can't get their profile/role created.

## Role capability matrix (given no Cloud Functions)

Building real backend-enforced roles from a static page requires the
Firebase Admin SDK for anything touching *other people's* accounts — that
SDK needs a secret credential that can never be shipped to a browser, so it
only runs in Cloud Functions (a paid-plan backend, opted out of for now).
Everything below is what's achievable with just Firestore + security rules.

| Requested capability | What's actually implemented |
|---|---|
| Owner = only miss.scumerica@gmail.com | Enforced in rules via `request.auth.token.email` — this part is fully real, can't be spoofed |
| View full user list | ✅ Owner/Admin: Settings → Manage Users |
| Add/remove roles | ✅ Owner/Admin, with hierarchy: nobody can grant "owner", admins can't touch the owner's row |
| View a user's notes | ✅ Read-only note-title list (Manage Users → "View notes") |
| Edit a user's notes | ❌ Not built — would need a full second note-editing surface loading another account's data; flagged as future work |
| Change/reset a user's password | ⚠️ Triggers a password-reset **email** to them (Settings → "Reset a user's password") — nothing can set a password directly without Admin SDK |
| Banning | ⚠️ App-level ban: a `banned` flag rejected by both the UI and Firestore rules. Their login still technically works but every read/write and the app itself is blocked. Their account isn't disabled at the platform level. |
| Deleting accounts | ❌ Can wipe their profile/notes data on request, but the underlying login can't be deleted without Admin SDK |
| Creating accounts "through the back end" | ⚠️ Invite system instead: Owner pre-assigns a role to an email (Manage Users → Invite, owner only); that role is granted automatically the first time that email signs up normally |
| User role: 50 note limit | ❌ Not yet — needs notes restructured into individual Firestore documents first (currently one blob per user), which is the next phase |
| User+: no note limit | ➖ Same as above — nothing currently enforces a limit at all, so this is trivially true for now |
| Test: bug reports + Testers chat | ❌ Not built — depends on chat, a separate later phase |

## Not built yet (later phases, per plan)

Global chat, "who's online" presence, group note sessions, note sharing via
username#0000, the profile menu, the note counter, and theming were
explicitly sequenced *after* this foundation. This file will get updated as
those land.
