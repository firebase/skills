# Bug Bash Testing Guide: Firebase Rules Specialist Antigravity Plugin

This guide explains how to install, verify, and test the **Firebase Rules Specialist** Antigravity plugin and its specialized **`firestore-rules-author`** subagent.

______________________________________________________________________

## 1. Overview & Objectives

### What is Being Tested?

- **Plugin Manifest**: `plugins/antigravity/plugin.json` — defines plugin metadata and registers the subagent.
- **Rules Symlink**: `plugins/antigravity/agents/rules.md` — symlinked directly to the enterprise Firestore rules specification (`skills/firebase-firestore/references/enterprise/security_rules.md`).
- **Subagent**: `plugins/antigravity/agents/firestore-rules-author.md` — an expert subagent that authors and audits Cloud Firestore Security Rules following the mandatory 5/5 auditor criteria.

### Testing Goals

1. Verify smooth installation into Antigravity / Jetski without pulling in unintended skills or starting unneeded MCP servers.
1. Verify that Antigravity discovers and registers the `firestore-rules-author` subagent.
1. Validate that the subagent reads `rules.md` via the symlink and adheres to all CEL rules and security invariants.

______________________________________________________________________

## 2. Installation & Setup

Choose from the following installation options:

### Method A: Download Pre-Packaged Zip from x20 (Fastest for Testers)

A self-contained zip file of the plugin is hosted on x20 and user storage with all symlinks resolved and packaged.

1. **Option 1: Direct unzip from corp filesystem**:

   ```bash
   mkdir -p ~/.gemini/plugins/firebase-rules
   unzip -o /google/data/ro/users/ch/christhompson/www/firebase-rules-plugin.zip -d ~/.gemini/plugins/firebase-rules
   ```

1. **Option 2: Download via browser / x20 URL**:

   - Download the zip from: [https://x20.corp.google.com/~christhompson/firebase-rules-plugin.zip](https://x20.corp.google.com/~christhompson/firebase-rules-plugin.zip) (or `http://x20/~christhompson/firebase-rules-plugin.zip`)
   - Unzip it into your plugin directory: `~/.gemini/plugins/firebase-rules` (or your project's plugin directory).

1. **Register in configuration**:
   Add the unzipped path to your `~/.gemini/config/plugins.json` (or workspace `.gemini/config/plugins.json`):

   ```json
   {
     "entries": [
       {
         "path": "/usr/local/google/home/<USERNAME>/.gemini/plugins/firebase-rules"
       }
     ]
   }
   ```

*(Note: The zip file is also mirrored in google3 experimental storage at: `/google/src/cloud/christhompson/firebase-rules-subagent/google3/experimental/users/christhompson/firebase-rules-plugin.zip`)*

### Method B: Local Workspace Registration (Git Clone)

1. Ensure you have the `antigravity-plugin` branch checked out:

   ```bash
   cd /path/to/firebase/agent-skills
   git checkout antigravity-plugin
   ```

1. Register the plugin directory in your Antigravity / Jetski configuration:

   - **Project-level** — in your target test workspace's `.gemini/config/plugins.json` (or `plugins.json`):
     ```json
     {
       "entries": [
         {
           "path": "/absolute/path/to/agent-skills/plugins/antigravity"
         }
       ]
     }
     ```
   - **User-level** — in your personal `~/.gemini/config/plugins.json`:
     ```json
     {
       "entries": [
         {
           "path": "/usr/local/google/home/<USERNAME>/firebase/agent-skills/plugins/antigravity"
         }
       ]
     }
     ```

### Method C: Antigravity CLI Import

Run the import command from your terminal:

```bash
# Import from local path
agy plugin import /path/to/agent-skills/plugins/antigravity

# Or import from GitHub branch
agy plugin import https://github.com/firebase/agent-skills --path plugins/antigravity
```

______________________________________________________________________

## 3. Verifying Installation

Before running test prompts, verify that Antigravity loaded the plugin correctly:

1. **Verify Plugin Registration**:

   ```bash
   agy plugin list
   ```

   *Expected Output:*

   ```text
   firebase-rules (Firebase Rules Specialist) - v1.0.0 [ENABLED]
   ```

1. **Verify Subagent Discovery in Chat**:
   In your Antigravity conversation window, type:

   > "List available subagents and describe firestore-rules-author"

   *Expected Result:*
   The agent confirms `firestore-rules-author` is registered with capabilities to design, author, and audit Cloud Firestore Security Rules.

1. **Verify Symlink Health**:
   Ensure `plugins/antigravity/agents/rules.md` resolves properly:

   ```bash
   ls -l plugins/antigravity/agents/rules.md
   head -n 5 plugins/antigravity/agents/rules.md
   ```

   *Expected Output:* The symlink points to `../../../skills/firebase-firestore/references/enterprise/security_rules.md` and outputs the `# 1. Generate Firestore Rules` header.

______________________________________________________________________

## 4. Test Scenarios (Critical User Journeys)

Execute the following 3 test cases to evaluate the subagent's performance.

### Test Case 1: Green-Field Rules Generation (E-Commerce App)

#### Prompt:

> "Use the @firestore-rules-author subagent to write production-ready Firestore security rules for an e-commerce platform with the following requirements:
>
> - `users/{userId}`: Contains user profile data (email, displayName, shippingAddress, role: 'customer' | 'admin').
> - `products/{productId}`: Publicly readable by anyone. Only admins can create or update products. Fields: title (string \<= 120 chars), price (numeric positive), stockCount (int >= 0), description (optional string \<= 2000 chars).
> - `orders/{orderId}`: Placed by customers. Only the order owner (`userId`) or an admin can read an order. Customers can create their own order with status 'pending'. Only admins can update order status. Immutable fields: userId, createdAt, items."

#### Verification Checklist:

- [ ] **Default Deny**: Starts with `match /{document=**} { allow read, write: if false; }`.
- [ ] **Symlink Consultation**: Agent accesses or refers to `rules.md` guidance.
- [ ] **CEL Language Compliance**:
  - Validates numeric fields using `is int` or `(data.price is int || data.price is float)`.
  - Does **NOT** use JavaScript `is number`.
- [ ] **PII Isolation ("No Mixed Content")**:
  - `users/{userId}` contains `email` and `shippingAddress`. Read access **MUST** be restricted to `isOwner(userId)`. No blanket `allow read: if request.auth != null`.
- [ ] **Update Bypass Protection**:
  - Validator function `isValidProduct(...)` is invoked in **both** `create` and `update` rules.
  - Validator function `isValidOrder(...)` is invoked in **both** `create` and `update` rules.
- [ ] **Immutable Fields**:
  - Checks that `order.userId` and `order.createdAt` cannot be altered on update (`areImmutableFieldsUnchanged(['userId', 'createdAt'])`).
- [ ] **Authority Source**:
  - Admin check uses `request.auth.token.admin == true` or bootstrapped admin verification, **NOT** `request.resource.data.role == 'admin'`.
- [ ] **Humble Delivery**:
  - Concludes with prototype disclaimer: *"I've set up prototype Security Rules to keep the data in Firestore safe..."*

______________________________________________________________________

### Test Case 2: Vulnerability Audit & Hardening (Penetration Test)

#### Prompt:

> "Use the @firestore-rules-author subagent to audit and harden the following vulnerable rules file:
>
> ```
> rules_version = '2';
> service cloud.firestore {
>   match /databases/{database}/documents {
>     match /profiles/{userId} {
>       allow read: if request.auth != null;
>       allow create: if request.auth.uid == userId;
>       allow update: if request.auth.uid == userId;
>     }
>     match /posts/{postId} {
>       allow read: if true;
>       allow write: if request.auth != null;
>     }
>   }
> }
> ```
>
> The profiles collection stores full name, email address, bio, and role ('member' or 'moderator'). The posts collection stores authorId, title, content, likesCount, and createdAt."

#### Verification Checklist:

- [ ] **Flags PII Leak**: Identifies that `allow read: if request.auth != null` on `/profiles/{userId}` exposes all users' emails to any logged-in user.
- [ ] **Flags Privilege Escalation**: Notes that an owner updating their own profile can set `role: 'moderator'` without restriction.
- [ ] **Flags Missing Update Validation**: Identifies that lack of schema validation allows users to wipe required fields or inject megabyte payloads (DoS/Resource Exhaustion).
- [ ] **Flags Posts Vulnerability**: Identifies `allow write: if request.auth != null` allows any user to overwrite or delete other users' posts.
- [ ] **Applies Hardened Solution**:
  - Replaces with granular `create`, `update`, `delete`.
  - Locks down role escalation.
  - Adds string size boundaries on `title` and `content`.
  - Requires `authorId` immutability.
  - Limits non-author updates on posts strictly to `likesCount` counter increments.

______________________________________________________________________

### Test Case 3: Unit Test Suite Generation

#### Prompt:

> "Generate automated security rules test cases using `@firebase/rules-unit-testing` for the hardened rules from Test Case 2. Include tests for:
>
> 1. Legitimate user profile update.
> 1. Attempted role self-escalation (customer -> moderator).
> 1. Attempted PII snooping (user A reading user B's profile).
> 1. Attempted update bypass (creating valid post, then updating with oversized payload)."

#### Verification Checklist:

- [ ] Uses modern `@firebase/rules-unit-testing` API (`initializeTestEnvironment`, `assertFails`, `assertSucceeds`).
- [ ] Tests negative cases (privilege escalation, snooping, update bypass) with `assertFails`.
- [ ] Tests positive cases with `assertSucceeds`.

______________________________________________________________________

## 5. Troubleshooting & Bug Reporting

| Issue                                                | Likely Cause                                | Solution                                                                                                          |
| :--------------------------------------------------- | :------------------------------------------ | :---------------------------------------------------------------------------------------------------------------- |
| **Agent not recognized (`@firestore-rules-author`)** | Plugin not indexed or invalid JSON manifest | Run `agy plugin list` to check status. Validate JSON with `python3 -m json.tool plugins/antigravity/plugin.json`. |
| **Agent does not reference `rules.md`**              | Broken symlink or path resolution error     | Run `test -e plugins/antigravity/agents/rules.md && echo OK`. Check git symlink mode (`120000`).                  |
| **Rules use JavaScript syntax (e.g. `is number`)**   | Subagent drifted from CEL guidelines        | Confirm subagent prompt includes CEL type directives (`is int`, `is float`).                                      |

### How to File Feedback

When filing bugs or observations during the bug bash, include:

1. The exact prompt used.
1. The generated rules snippet.
1. The specific checklist item that failed (e.g., "Update bypass not caught", "Used `is number` instead of `is int`").
