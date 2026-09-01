# Firebase Rules Specialist Plugin for Antigravity

A dedicated Antigravity/Jetski plugin providing the `firestore-rules-author`
subagent.

This plugin is designed as a standalone, lightweight hosting vehicle for
Firebase Security Rules authoring and auditing workflows without bundling full
SDK skills or MCP background processes.

## Features

- **`firestore-rules-author` Subagent**: A specialized AI subagent that authors,
  audits, and hardens Cloud Firestore Security Rules to achieve a 5/5 score
  against the Firebase Security Rules Validator.

## Installation / Usage

### Option 1: Antigravity CLI Import

```bash
agy plugin import https://github.com/firebase/agent-skills --path plugins/antigravity
```

### Option 2: Project / Workspace Configuration (`plugins.json`)

To enable this plugin in your workspace or personal configuration:

```json
{
  "entries": [
    {
      "path": "path/to/firebase-skills/plugins/antigravity"
    }
  ]
}
```
