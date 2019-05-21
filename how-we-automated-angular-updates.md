# How we automated our Angular updates

Upgrading your Angular applications is quite easy with the Angular CLI. We have been faithfully upgrading to major releases
usually within a week of release, without hesitation or issues since Angular 4.

This process has been delightful and very low effort. You get a compile error with some breaking changes and maybe there's some 
manual work to be done. But other than that, it's been quite effortless and easy to maintain.

So much that we have automated our Angular upgrade process using our CI solution.

### How we did it

We have 5 steps in our CI pipeline. For our example we used Jenkins, but there's no reason this wouldn't work with any other CI pipeline.

Our 5 steps consist of:
- Git checkout, workspace cleanup and creating a new branch.
- Using the Angular CLI to run our updates.
- If the updates failed, or there were no updates available; we send a message to our communication platform of choice. 
- If the updates succeeded, we commit and push our new branch to our git repository.
- We then create a Pull Request using the API of our git management platform and post the resulting URL to our communication platform.

#### Git checkout
We clean our workspace, checkout the `master` branch and make sure we create a new separate branch. Because we use something resembling `git-flow` and `semantic commit messages`; our generated branches might look like this: `chore-ngupdate/autoupdater-2019-05-21`. 

#### Angular CLI
The Angular CLI has built in update functionality that we leveraged. Running `ng update --all` is sufficient with most solutions. In some cases where you run a private repository like Nexus, we found that the `--all` flag ran into some issues. But you can script around this using some `grep` and `awk` commands to get the results you need to run your manual `ng update @angular/core rxjs` commands.

In this step, we check and determine if there are no updates (`No outdated dependencies!`), if the update is not compatible with automatic upgrading, or that it indeed succeeded and there are changes to files on our filesystem. Depending on the above outcome, we might `return non-zero` and send a message to our communication platform to alert a developer to potential manual steps.

#### Github/Gitlab API
When the update has indeed succeeded and we have determined that there are changes to the filesystem, we could run the default pipeline that contains `ng test` and `ng e2e`. However, our environment runs CI jobs automatically on `pull requests`, so we just create the pull request and let CI take it from there. 

For us, this means that we have to `commit` to our new branch and `push` our branch to the repository. 
In order to get this to work, you may need to allow your CI's `SSH key` so it has the rights to create and push new branches.

After that, we use the provided API's from our `Git management platform` like Github or Gitlab to create a new `Pull Request`. For us this consisted of sending a `HTTP POST` to a URL with a required `payload` containing our `branch name`, the `target branch` and a `title` reflecting the changes. 

After this, the `pull request` is made and ran by CI automatically to check if the changes break the pipeline.

#### Communication channel
Optionally you could include your communication platform of choice; a simple `Webhook` call in the form of `HTTP POST` to Slack (or comparable alternatives) makes it possible to notify a channel of a new (automatic) pull request, or a failed attempt. 
We decided to also send a message when the update ran, but no dependency changes were detected; to remind us to keep track. 

For us this easily integrated in our steps, where the final result would contain a link to the new `pull request`, and a failed attempt with incompatible changes, or no changes at all would send different messages. Keeping our developers actively in the loop of any changes outside our development.

### When/how do we run this pipeline?

Our CI allows for automatically scheduled builds, solutions like Jenkins and Gitlab offer these out of the box. We have it set up so that it runs once a week.

This way we can come in fresh on a Monday, and hopefully be greeted by a fresh `pull request` with the latest Angular updates to start off your week.

In case your CI doesn't support this, there's enough creative ways to achieve something similar. A `cronjob` that triggers are rebuild on your pipeline might already do the job.

### In conclusion

We automated these steps for two reasons: 

- They take up some valuable time from our developers; but there's no reason a bot couldn't do the basic steps for us. 
- This actively alerts our developers to keep track of updates. When the bot reports a failure, we know there's manual steps to be taken to keep us up to date and our codebase happy.

We would highly recommend you explore these possibilities for your own project, and save yourself some valuable time!

