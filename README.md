# glitchub

Here's how to connect Glitch and Github together:

1. Go to [glitch.com](http://glitch.com) and create a new `hello-express` app.
2. Open the `package.json` file and add the following packages:

  - `node-cmd`
  - `body-parser`

3. Go to your `server.js` file load the following libraries:

```js
const cmd = require('node-cmd');
const crypto = require('crypto'); 
const bodyParser = require('body-parser');
```

4. Look for `app.use(express.static('public'));` and add the following line below:

`app.use(bodyParser.json())`

5. Create a post endpoint to receive the Github webhook:

```js
const onWebhook = (req, res) => {
  let hmac = crypto.createHmac('sha1', process.env.SECRET)
  let sig  = `sha1=${hmac.update(JSON.stringify(req.body)).digest('hex')}`

  if (req.headers['x-github-event'] == 'push' && sig == req.headers['x-hub-signature']) {
    cmd.run('chmod 777 ./git.sh') 
    cmd.get('./git.sh', (err, data) => {  // Run our script
      if (data) console.log(data)
      if (err) console.log(err)
    })

    cmd.run('refresh')
  }

  return res.sendStatus(200)
}

app.post('/git', onWebhook)
```

6. Open the `.env` file and set a SECRET
7. Create a `git.sh` file and add

```bash
#/bin/sh

# Fetch the newest code
git fetch origin master

# Hard reset
git reset --hard origin/master

# Force pull
git pull origin master --force
```

8. Open the glitch console and execute:

`ssh-keygen`

8. Copy the content of the `.ssh/id_rsa.pub` file with

`cat .ssh/id_rsa.pub`

9. Create a new Github repo with a `README.me` file
10. Go to the 'deploy keys' section of the settings page
11. Click on 'add deploy key' and add the `id_rsa.pub` string in the key field; set the title to 'glitch'.
12. Check the 'Allow write access' box too.
13. Go to the settings page of your project, open the webhooks section and add a new webhook

- Payload URL: https://PROJECT_NAME.glitch.me/git
- Content type: `application/json`
- Secret: use the SECRET you set up in your glitch project
- The rest of the defaults are ok
    - Enable SSL verification
    - Just the push event.
    - Active


14. Go to glitch and click on tools > Git, import, export > export to GitHub
15. Clone your repo in your dev machine and merge the `glitch` branch
16. Go back to the glitch console and add the github remote:

`
git remote add origin git@github.com:USERNAME/PROJECT.git
git branch --set-upstream-to=origin/master master
`

17. On your dev machine, do a `git push`


