# How to keep a Glitch project in sync with a GitHub repo

You can follow these first six steps or [remix this glitch project](https://glitch.com/~glitchub) and jump to step #7.

1. Go to [glitch.com](http://glitch.com) and create a new `hello-express` app.
2. Open the `package.json` file and add the following packages:

  - `node-cmd`
  - `body-parser`

3. Open the `server.js` file and load the following libraries:

```js
const cmd = require('node-cmd');
const crypto = require('crypto'); 
const bodyParser = require('body-parser');
```

4. Look for `app.use(express.static('public'));` and add the following line below:

`app.use(bodyParser.json());`

5. Create a post endpoint to receive the GitHub webhook:

```js
const onWebhook = (req, res) => {
  let hmac = crypto.createHmac('sha1', process.env.SECRET);
  let sig  = `sha1=${hmac.update(JSON.stringify(req.body)).digest('hex')}`;

  if (req.headers['x-github-event'] === 'push' && sig === req.headers['x-hub-signature']) {
    cmd.run('chmod 777 ./git.sh'); 
    
    cmd.get('./git.sh', (err, data) => {  
      if (data) {
        console.log(data);
      }
      if (err) {
        console.log(err);
      }
    })

    cmd.run('refresh');
  }

  return res.sendStatus(200);
}

app.post('/git', onWebhook);
```

6. Create a `git.sh` file adding the following lines:

```bash
#/bin/sh

# Fetch the newest code
git fetch origin master

# Hard reset
git reset --hard origin/master

# Force pull
git pull origin master --force
```

*This file will be in charge of pulling the changes from your Github repo.*

----

7. Open the `.env` file and set a SECRET:

```bash
SECRET=cirrus-socrates-particle-decibel-hurricane-dolphin-tulip
```

8. Open the glitch console for your project and generate a new SSH key with:

```bash 
ssh-keygen
```

*Just press `<Enter>` to accept all the questions.*

9. Read the content of the `.ssh/id_rsa.pub` file with:

```bash 
cat .ssh/id_rsa.pub
```

*Then copy the string to your clipboard.*

10. Create a new GitHub repo with a `README.me` file (or any other file, it's important that the project is not empty).
11. Go to the `deploy keys` section of the settings page of your GitHub project.
12. Click on `Add deploy key` and add the SSH key you generated in the step #8; add `glitch.com` in the title field.
13. Check the `Allow write access` box too.
14. Now open the `webhooks` section and add a new webhook (replacing `PROJECT_NAME` with your actual project name):

```html
Payload URL: https://PROJECT_NAME.glitch.me/git
Content type: application/json
Secret: use the SECRET you set up in your glitch project
```
*The rest of the defaults are OK.*

15. Go back to your Glitch project and export your project to GitHub (that will create a new `glitch` branch in your GitHub repo): Tools > Git, Import, and Export > Export to GitHub.

16. Clone your repo in your dev machine and merge the newly created `glitch` branch with the master branch:

```bash 
git merge glitch
```

17. In the Glitch console, set the remote origin to the master branch of your GitHub project:

```bash
git remote add origin git@github.com:USERNAME/PROJECT.git
git branch --set-upstream-to=origin/master master
```

18. In your dev machine, add a dummy test file (like `hello.txt`) and commit it.
19. Push the changes with `git push`. 

If everything is ok, your Glitch project should be updated and a `hello.txt` file should appear in the list of files.   
If you don't see it, checkout the log of your Glitch project.

# Updating your project from GitHub → Glitch

This should be the default method of updating your project. Develop in your dev machine, commit the changes and do a `git push`

# Updating your project from Glitch → GitHub

If you change something in Glitch, first you'll need to export your project to GitHub and then merge the `glitch` branch with your master branch.

```bash 
git merge glitch
git push
```

# Credits

While I've detailed all the steps and added  information on how to add a deploy key, the overall solution (including the `git.sh` and webhook sync code) comes from [this post by @jarvis394 on the glitch forum](https://support.glitch.com/t/tutorial-how-to-auto-update-your-project-with-github/8124). 



