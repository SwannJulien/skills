# Map changes — <branch> since <sha>

<!-- Printed in the terminal, and offered for the PR description. Short by
     contract: one screen. The maps are the reference; this is the thing that
     actually gets read, and it stops being read the moment it grows.

     Vocabulary, used everywhere below:
       +  added
       ~  changed
       -  removed

     Derive every line from `git diff <sha>..HEAD -- docs/map/`, not from what a
     subagent said it did. -->

<n> commits, <n> files changed. Maps regenerated: <db, api>. Unchanged: <front, deploy>.

## Database

```
+ invitations          6 cols, FK org_id → orgs (cascade), UQ (org_id, email)
~ users.role           enum member_role gained 'owner'
~ orgs                 + column invite_policy text not null default 'admins'
- legacy_sessions      dropped in 20261018_drop_sessions
```

## API

```
+ POST /invitations              admin   → invitations w, orgs r
+ POST /invitations/:id/accept   public  → invitations w, users w
~ GET  /orgs/:id                 now includes invite_policy in the response
- GET  /legacy/users
```

## Front end

```
+ AcceptInvite    page       route /invite/:token
~ InviteModal     container  now owns the invite form state (was in OrgSettings)
```

## Seams that moved

<!-- The section that justifies the skill. Not a restatement of the lists above:
     the relationships between them, which is what you lose first when an agent
     writes the code. Each line crosses at least two layers. -->

- `POST /invitations` is called only from `InviteModal`, and is the only writer of `invitations`.
- `users` gained a second writer: `POST /invitations/:id/accept` now inserts, where only `POST /users` did before.
- `authStore` lost `OrgSwitcher` as a writer; `OrgSettings` writes it instead.

## Worth a look

<!-- Facts with an obvious question attached. No advice, no recommendations —
     the user decides. Omit the section when it would be empty; a padded
     "worth a look" teaches them to skip it. -->

- `POST /invitations/:id/accept` is unauthenticated and writes `users`. Intentional per its `Why`, worth a second read.
- `invitations.expires_at` has no index and is filtered on in `GET /orgs/:id/invitations`.
- Nothing calls `GET /legacy/users` — it survived the removal of its caller.
- `INVITE_TTL_DAYS` is read by the server and set in no environment or example file.

## Not covered

<!-- Changes in the diff that fall outside every map's source globs, so this
     report says nothing about them. One line, with the directories. Prevents
     the report being mistaken for a review of the whole PR. -->

<n> changed files are outside the mapped sources: `<paths>`.
