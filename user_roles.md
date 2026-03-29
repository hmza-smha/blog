[Back](./README.md)

---

## 📑 Table of Contents

| Section | Topic |
|---------|-------|
| [1](#1-introduction-defining-the-problem) | Problem Definition |
| [2](#1️⃣-teamleadid-on-the-team-the-obvious-model) | TheadId Model |
| [3](#2️⃣-membership--role-the-grown-up-default) | Membership + Role |

---

# User Roles

# Problem: Designing User, Team, Team Lead, and Team Member Relationships

### A Comprehensive Guide to Database Modeling Approaches

---

## 1. Introduction: Defining the Problem

Many software systems need to model collaboration between people. A very common scenario is:

* There are **Users** (people using the system).
* Users can belong to **Teams**.
* Teams contain **Members**.
* Some members may be **Team Leads** (or hold other roles).

At first glance, this looks simple. However, the challenge arises when deciding **how to represent these relationships in a database**. The way the data is structured affects:

* System flexibility
* Query complexity
* Future scalability
* Ability to track history
* Ease of maintenance

For example, the following questions must be supported:

* Can a user belong to multiple teams?
* Can a team have multiple leads?
* Can a user be a lead in one team and a member in another?
* Do we need to track role history?
* Will permissions evolve over time?

There is **no single universal design**. Instead, there are several architectural approaches, each optimized for different priorities.

This article explains the major approaches and how each one can be implemented.

---

I got you. Let’s slow this *way* down and treat this like a whiteboard session where I’m explaining it to a teammate, not “design theory” 😌
I’ll explain **each approach one by one**, in **plain English**, with:

* what the idea is
* how you’d actually store the data
* how you’d answer basic questions like *“who is the lead?”*
* why teams choose it
* where it usually breaks

No tables, no jargon overload. Just clarity.

---

# 1️⃣ TeamLeadId on the Team (the “obvious” model)

### The idea

A team has one special person: the lead.
So you just store that person directly on the team.

### How you implement it

You have:

* a **users table**
* a **teams table**
* maybe a **team_members table**

In the teams table, you add a column like `team_lead_id` that points to a user.

Plain English:

> “This team has a lead, and it’s user #42.”

Members might live in a separate table, or sometimes the lead is *not even considered a member explicitly*.

### How you answer questions

* Who is the lead? → Look at `teams.team_lead_id`
* Who are the members? → Query `team_members`
* Is the lead a member? → 🤷 depends on your rules

### Why people choose this

* Super easy to understand
* Very few joins
* Fast to implement
* Feels “natural” at first

### Where it breaks

* A lead is still a member, but now they live in a different place
* You can’t easily support:

  * co-leads
  * acting leads
  * lead history
* You’re hard-coding a role into the schema

This is fine for **tiny or temporary systems**, but it doesn’t age well.

---

# 2️⃣ Membership + Role (the “grown-up default”)

### The idea

A team doesn’t *have* a lead.
People **have roles inside a team**.

### How you implement it

You create a **join table** that connects users and teams.
Each row answers one question:

> “What is this user’s role in this team?”

So instead of:

* “team → lead”
  You model:
* “user ↔ team (with role info)”

Roles are stored as data, not columns.

### How you answer questions

* Who is the lead? → Find membership rows where role = LEAD
* Who are the members? → All rows for that team
* Can someone be a lead in one team and a member in another? → Yes, naturally

### Why people choose this

* Matches reality
* Extremely flexible
* Easy to extend later
* No duplication of concepts

### Where it can feel “weak”

* If you want **history**, you have to add timestamps
* If you want **exactly one lead**, you must enforce it with constraints

This is the model most experienced teams reach for first.

---

# 3️⃣ Hybrid Model (membership + shortcut)

### The idea

Use the clean model **and** make common queries fast.

Think:

> “The membership table is the truth, but I want instant access to the lead.”

### How you implement it

You still use a membership table with roles.

But on the team, you store a reference to *the specific membership row* that represents the current lead.

Plain English:

> “This team’s lead is *that* membership record.”

### How you answer questions

* Who is the lead? → One FK lookup
* Who are the members? → Membership table
* Want to change lead? → Update the membership reference

### Why people choose this

* Clean model
* Faster reads
* Still flexible

### Where it can go wrong

* Slightly more complex
* You must keep the FK and role data in sync

This shows up in **performance-sensitive systems**.

---

# 4️⃣ RBAC (roles as permission bundles)

### The idea

Stop thinking in terms of “lead vs member”.
Instead, think:

> “Who is allowed to do what in this team?”

A lead is just someone with more permissions.

### How you implement it

You define:

* roles (Lead, Member, Admin)
* permissions (Create Task, Remove Member, Rename Team)
* mappings between them

Then you assign roles to users **per team**.

### How you answer questions

* Who is the lead? → Who has the “manage team” permission
* Can there be multiple leads? → Yes
* Can roles change? → Easily

### Why people choose this

* Very expressive
* Scales to complex orgs
* Security-friendly

### Where it hurts

* Overkill for simple apps
* Lots of joins
* Harder to reason about mentally

Great for **enterprise or SaaS products**.

---

# 5️⃣ Event Sourcing (nothing is ever updated)

### The idea

You don’t store “who is the lead”.
You store **everything that ever happened**.

### How you implement it

Instead of tables like “memberships”, you have a log:

> “Alice joined Team A”
> “Alice became lead”
> “Bob became lead”

The current lead is derived by replaying events.

### How you answer questions

* Who is the lead now? → Replay events
* Who was the lead last year? → Replay events until that date

### Why people choose this

* Perfect audit trail
* Easy to answer historical questions
* Very powerful

### Where it’s hard

* Mental overhead
* Needs projections for performance
* Hard for newcomers

Usually seen in **finance, compliance, or complex domains**.

---

# 6️⃣ Graph Model (relationships first)

### The idea

Users and teams are nodes.
Roles are **edges** between them.

> “Alice LEADS Team A”
> “Bob MEMBER_OF Team A”

### How you implement it

You use a graph database.
Each relationship carries a type.

### How you answer questions

* Who leads this team? → Traverse LEADS edges
* What teams does Alice lead? → Traverse outgoing edges

### Why people choose this

* Natural modeling of org structures
* Flexible
* Excellent for traversal queries

### Where it’s weak

* Reporting is harder
* Operational overhead
* Not ideal for transactional systems

Best for **org charts and social structures**.

---

# 7️⃣ Inheritance / Polymorphism (role = identity)

### The idea

A lead is a *kind of user*.
A member is another kind.

### How you implement it

You split users by type:

* TeamLead
* TeamMember

### How you answer questions

* Who is the lead? → Query team_leads
* Can someone change roles? → Painfully

### Why people try this

* Feels clean in OOP
* Simple mental model

### Why it fails

People change roles.
People have multiple roles.
Reality breaks this model immediately.

Avoid unless the domain is frozen in time.

---

# 8️⃣ NoSQL Embedded Model (team owns everything)

### The idea

A team document contains its members.

### How you implement it

Each team stores a list of users with roles inside it.

### How you answer questions

* Who is the lead? → Read the team document
* Who are all Alice’s teams? → Scan many documents 😬

### Why people choose this

* No joins
* Simple reads
* Fits document databases

### Where it hurts

* User-centric queries are painful
* Hard to enforce constraints
* Updates can conflict

Good for **small teams, read-heavy systems**.

---

# 9️⃣ Temporal Roles (roles change over time)

### The idea

Roles aren’t permanent.
They exist **during time ranges**.

### How you implement it

Every role assignment has:

* start date
* optional end date

### How you answer questions

* Who is the lead now? → Role where end date is null
* Who was the lead on June 1? → Role active on that date

### Why people choose this

* Compliance
* Rotations
* Audit requirements

### Where it costs you

* Every query needs time logic
* More indexes
* More complexity

Used in **HR and regulated industries**.

---

# 🔟 Policy-driven (rules decide leadership)

### The idea

Nobody is *assigned* as lead.
Rules determine it dynamically.

> “Most senior engineer becomes lead.”

### How you implement it

You store rules, not assignments.
Leadership is computed at runtime.

### How you answer questions

* Who is the lead? → Evaluate the rule
* Why them? → 😅 good luck explaining

### Why people choose this

* Extreme flexibility
* Zero manual assignments

### Why it’s dangerous

* Hard to debug
* Hard to explain
* Hard to trust

Rare, experimental, high-risk.

---

## Final grounding thought

If you remember **one thing**, remember this:

> **Roles are relationships, not properties.**

Everything else is just different ways of honoring or violating that truth.

---
[Back](./README.md)