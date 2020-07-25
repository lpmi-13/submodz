# Submodz

The idea of this activity is to provide a low-risk situation to gain experience
working with git submodules.

The example scenario will be that you have a project that needs to include a submodule (perhaps from another team at your company), and you would like to include that
in your main repository.

In theory, it might be easier for that other team to publish their code as a private
package in the package manager for whatever language you're working in (npm/pip/cargo/etc), but in some cases, it won't be that easy.

This is a chance to practice what to do in one of those cases.

## The project (without submodules, at the beginning)

It's a simple app that we have, we just want to start up a local server and show the
chances that it's gonna be awesome in that city.

```bash
npm install
node index.js
```

should fire up your server and have you ready to rock at `localhost:3000`

## add the submodule

Because the predictions are just a bit too rosy, let's add a transform function that was developed by somebody else.

Fork the repo at https://github.com/lpmi-13/submodule-transform to your github account, and then clone the submodule to a different local folder so we can work with it later:

```bash
git clone git@github.com:YOUR_USER_NAME/submodule-transform
```

now we add the remote repo as a submodule into this main repository, so we can track changes in it, like so:

```bash
git submodule add git@YOUR_USER_NAME/submodule-transform transform
```

you should see some terminal output like the following:

```
Cloning into 'transform'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 4 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), done.
Checking connectivity... done.
```

and when you try `git status`, you should see two new changes:

```
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   transform
```

And then let's add this new function into our `index.js`. Add

```js
const transform = require('./transform/transform');
```

to the top of the file, and change

```js
res.send(`the chances of awesomeness in ${city} are ${percentage}`);
```

to

```js
res.send(`the chances of awesomeness in ${city} are ${transform(percentage)}`);
```

Then run the server and see what the chances of awesomeness in a random city are:

```bash
curl localhost:3000/london
// the predicted awesomeness in london is 1.5%
```

you can also see what commit the submodule is pointing to by typing:

```bash
git submodule
```

the output should look something like:

```
 9399af58422259a04556c09395cc4efda7bab4bf transform (heads/master)
```

**(NOTE: I recommend that you add/commit the current changes to git now, since that makes it easier to update the submodules later. Essentially, updating will
be blocked if you have uncommitted submodule changes in there, and we just added
a submodule, so we definitely have submodule changes)**

## now let's change something!

The team responsible for the transform function has now hired a bunch of Americans, and they have decided there needs to be more awesome...so they make the predictions a thousand times as big!

Unfortunately, this change is made in the submodule, and then the reference to that submodule needs to be updated in our main repository. Not super straightforward, but let's walk through it and see what the process looks like.

First, change directories into the folder you cloned the submodule to, and update the code so that it multiplies the input by 1000.

change:

```js
const transform = inputNumber => inputNumber * 0.25;
```

to something like:

```js
const transform = inputNumber => inputNumber * 1000;
```

Now we just commit that change and push it back to your forked remote, so it will show as updated in our main repo:

```bash
git add transform.js
git commit -m 'make the percentage 1000 times bigger, for reasons!'
git push origin master
```

## update the submodule

back in your main repo, if you type

```bash
git submodule update --remote
```

you should see that your submodule has been updated, with terminal output like the following:

```
   9399af5..12f80cf  master     -> origin/master
Submodule path 'transform': checked out '12f80cfb707459bac790fd50d9982706a1913037'
```

and when you run the server locally, and try to find out the predicted awesomeness in a city, it should be some ridiculously too high number:

```bash
curl localhost:3000/chicago
// the predicted awesomeness in chicago is 3200%
```

the only last slight difference from normal git updates is that when you type

```bash
git status
```

you'll see something slightly different in the output, like so:

```
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   transform (new commits)
```

...rather than just the file name that got updated (`/transform/transform.js`) as you might expect. This is just the way that git shows updates in submodules,
and something to be aware of.

Now to finish off the exercise, add and commit that, and we're all done!

for the curious, your git history (with `git log -p`) should look like this:

```diff
diff --git a/transform b/transform
index 9399af5..12f80cf 160000
--- a/transform
+++ b/transform
@@ -1 +1 @@
-Subproject commit 9399af58422259a04556c09395cc4efda7bab4bf
+Subproject commit 12f80cfb707459bac790fd50d9982706a1913037
```

to show that we've updated the commit of the submodule that our repo is using.

...and that's it! You have successfully used and updated a submodule in a project.
