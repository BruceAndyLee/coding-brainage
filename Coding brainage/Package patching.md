---
Status: Completed
tags:
- chore
---
### Install & setup

```Bash
yarn add patch-package postinstall-postinstall # --dev
```

```JSON
// package.json
"scripts": {
+  "postinstall": "patch-package"
 }
```

### Use

```Bash
# to create or expand 'patches' folder
yarn patch-package <package-name>
# this command will check the current state of the specified npm-provided module
# and keep track of the changes to be applied once the app is install with yarn
```