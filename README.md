# Let's build a web site in 10 minutes

## Prerequisite

- [Create a Google Cloud Project][link:create-gcp-project]
- Install [Google Cloud CLI][link:gcloud-cli]
- Install [Node.js][link:node_js]
    - recommend use [nvm][link:nvm] to manage your node environment.

[link:create-gcp-project]:https://cloud.google.com/resource-manager/docs/creating-managing-projects
[link:gcloud-cli]:https://cloud.google.com/pubsub/docs/quickstart-cli
[link:node_js]:https://nodejs.org/en/
[link:nvm]:https://github.com/creationix/nvm


## Ready, Set, GO!

Create a directory for our web site.

```sh
mkdir node-app
```

Init npm package with default settings
```sh
cd node-app
npm init -y # -y means say "yes" to all configuration questions.
```

Open `package.json` and add a start script

```diff
{
  "name": "node-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
+   "start": "node app.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Install `Express`

```sh
npm install express
```

Create `app.js` file

```sh
touch app.js
```

Write some express hello world

```javascript
// app.js
const express = require('express')
const app = express()
const PORT = 3000

app.get('/', (req, res) => res.send('Hello World!'))

app.listen(PORT, () => console.log(`Example app listening on port ${PORT}!`))
```

Run it!
```sh
npm start
```


## 

