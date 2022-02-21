---
title: ESLint Setup
layout: post
comments: true
published: false
description: 
keywords: 
---

* Download and install [Node.js](https://nodejs.org/en/download/) (Windows Installer .msi)
* Open `cmd` and install ESLint globally: 

```sh
npm install -g eslint
```

* Setup a configuration file (.eslintrc.json)

```sh
eslint --init
```

* Select with the arrow keys (up and down)
> Answer questions about your style

* Example config file (.eslintrc.json):

```json
{
    "env": {
        "browser": true,
		"jquery": true,
        "es6": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "sourceType": "script"
    },
    "rules": {
        "indent": [
            "error",
            4
        ],
        "linebreak-style": [
            "error",
            "unix"
        ],
        "semi": [
            "error",
            "always"
        ]
    }
}
```

After that, you can run ESLint on any file or directory like this:

A file
```bash
eslint yourfile.js
```

A directory

```bash
eslint assets/js
```

Try to fix as many issues as possible

```bash
eslint --fix assets/js
```