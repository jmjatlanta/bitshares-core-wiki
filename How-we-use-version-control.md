
NOTE:  We're still working on getting ready to transition to this policy.  It isn't in effect quite yet!

Our new (as of October 2015) Git branching model is heavily influenced by [Git Flow](http://nvie.com/posts/a-successful-git-branching-model/),
and [this blog post](http://www.draconianoverlord.com/2013/09/07/no-cherry-picking.html).

The core development repository is [Graphene](https://github.com/cryptonomex/graphene).  Code always flows
downward from Graphene to chain repositories.  Porting code from a chain repository to Graphene requires
`cherry-pick`, and is to be avoided.

### List of production blockchains

- https://github.com/bitshares/bitshares-2
- https://github.com/peertracksinc/muse

### Chain-specific branches

- `$CHAIN/master`  : Most recent common ancestor of `$CHAIN` and `graphene/master`
- `$CHAIN/$CHAIN`  : Production branch.  Currently active nodes are advised to run this.
- `$CHAIN/staging` : Staging branch.  Should be helping Advanced users may run this to help us test new features.
- `$CHAIN/hftest`  : Hardfork testing.  Hardfork with earlier inclusion date, thus, it is different from the main chain and will be thrown away when testing is finished.  Such branches will not be created until further infrastructure improvements have been made, i.e.:  GUI improvements to show when you're on a non-production chain, hardforking the reported chain ID, accepting transactions with multiple chain ID to allow us to import the tx flow from the main chain.

### Master repo branches

- `graphene/master`  : New chains are advised to start with this commit, then configure `config.hpp`.
- `graphene/develop` : Staging branch for new features not yet ready for `graphene/master`.
- `graphene/stable`  : Most recent common ancestor of `$CHAIN/master` across all chains.  Preferred basis for patches, since patches written against this should apply to any chain.

### Individual patch branches:

- `graphene/fork-123`    : HF Commits which will change consensus go in here. 
- `graphene/api-123`     : Non-HF Commits which add API calls go here.
- `graphene/wallet-123`  : Non-HF Commits which add wallet features go here.
- `graphene/opt-123`     : Non-HF Commits which optimize existing code go here.  Resulting code should be semantically equivalent but faster.
- `graphene/bugfix-123`  : Non-HF Commits which fix a specific local bug.
- `graphene/cleanup-123` : Non-HF Commits which refactor, neaten, or fix architectural issues.  May be semantically equivalent.
- `graphene/feature-123` : Possibly-HF Commits which don't fit well in the above categories.  The issue should describe exactly what it does.

## Policies

### Force-push policy

- `$CHAIN/master` should never be force-pushed.
- `$CHAIN/$CHAIN` should never be force-pushed.
- Ideally, `graphene/master` should never be force-pushed; exceptions to this policy may be made on a case-by-case basis.
- `graphene/stable` is always maintained as the MRCA of all `$CHAIN/master`.  Since we have a policy to never force-push `$CHAIN/master`, this effectively means `graphene/stable` will never be force-pushed.
- `graphene/develop` should rarely be force-pushed, but it may be force-pushed to kill patches that are prematurely merged.
- Individual patch branches may be force-pushed at any time, at the discretion of the developer or team working on the patch.

### Tagging policy

- A tag is made of `$CHAIN/$CHAIN` when a new "production" release is created.
- A corresponding tag should be made on `graphene/master` based on the MRCA of `$CHAIN/$CHAIN` and `graphene/master`.

### Pull request policy

- All pull requests should be against `graphene/stable`, unless they contain code specific to some chain.

### Advancement of chain-specific master branch

- Each chain-specific `master` branch should always be an ancestor of `graphene/master`.
- A chain-specific `master` branch should not include any commit that has not been merged into its production branch.
- If we have a situation where a commit from `graphene/master` can *never* be ported to some chain, the above policies imply that chain's `master` will become stuck.  In this case, the commit should be merged to the production branch and immediately reverted.  This action documents our conscious decision not to put this particular patch on this particular chain, and should hopefully keep the patch from being inadvertently merged later.

### Code review policy

- Two developers *must* review *every* consensus-breaking change before it moves into `graphene/master`.
- Two developers *should* review *every* patch before it moves into `graphene/master`.
- One of the reviewers may be the author of the change.
- This policy is designed to encourage you to take off your "writer hat" and put on your "critic/reviewer hat."  If this was a patch from an unfamiliar community contributor, would you accept it?  Can you understand what the patch does and check its correctness based only on its commit message and diff?  Does it break any existing tests, or need new tests to be written?  Is it stylistically sloppy -- trailing whitespace, multiple unrelated changes in a single patch, mixing bugfixes and features, or overly verbose debug logging?
- Having multiple people look at a patch reduces the probability it will contain bugs.

## Scenarios

Each of these scenarios starts out with "The Backstory" specifying what situation you're in and what you're trying to do,
and then a concise explanation and list of commands telling you how to get to where you need to go.

### Setting up remotes

The Backstory:  I want to work on `bitshares` and `graphene` in the same local repository.

- This is why git remotes were invented.  A git remote is simply an alias to a URL where a repository lives.  Create git remotes as follows:

    git remote add graphene git@github.com:cryptonomex/graphene
    git fetch graphene
    git remote add bts2 git@github.com:bitshares/bitshares-2
    git fetch bts2
    git remote add muse git@github.com:peertracksinc/muse
    git fetch muse

- Git tip:  The `git fetch` command locally mirrors all the branches on the specified remote into *remote-tracking branches* such as `bts2/bitshares`.  These remote-tracking branches are automatically updated by future `git fetch` operations, so at any point in time the `bts2/` namespace collectively represents the state of the `bts2` node the last time you did a `git fetch bts2`.

- Git tip:  If you created your repository with `git clone`, then the `origin` remote is set up as the remote you originally specified.  Anywhere you specify `origin` in your normal `git` workflow, you can specify an arbitrary remote name.

- Git tip:  The URL for a remote can be changed with `git remote set-url`.  This may be useful e.g. when moving from a privately hosted repository on a local filesystem or Gitlab, to a public repository on Github.

- Git tip:  Remotes may also be `http` or `https` URL's, paths on the local filesystem, or `git+ssh` to run over ssh.

### Creating a new feature

The Backstory:  I want to add an API call.

- Create an issue in the [Graphene repo](https://github.com/cryptonomex/graphene).  This assumes the issue you created was issue number 123.

    git fetch graphene
    git checkout graphene/stable -b api-123
    notepad libraries/app/api.cpp
    git add libraries/app/api.cpp
    git commit -m "The API is better now"
    git push graphene HEAD:api-123

- Git tip:  The `-b` command creates a branch.  A branch is simply a named pointer to a commit, which automatically advances when you create a new commit with `git commit`.  Creating local branches for specific tasks and deleting them when you're done is often a useful workflow.

### Deploying a new feature

The Backstory:  I want to add the API feature I created in Graphene (see previous item) to BitShares.

- First, self-review.  Take off your "patch author hat" and put on your "critic/reviewer hat".  If this was a patch from an unfamiliar community contributor, would you accept it?  Can you understand what the patch does and check its correctness based only on its commit message and diff?  Does it break any existing tests, or need new tests to be written?  Is it stylistically sloppy -- trailing whitespace, multiple unrelated changes in a single patch, mixing bugfixes and features, or overly verbose debug logging?

- Merge it into `graphene/develop`:

    git fetch graphene
    git checkout graphene/develop -b develop
    git merge graphene/api-123
    git push graphene HEAD:develop

- Merging your feature into `graphene/develop` is a signal that you've completed your self-review and it's as ready for production as you can make it on your own.  It's time for other developers to take a look at it.

- When it passes code review, it's ready to be merged to `graphene/master`:

    git fetch graphene
    git checkout graphene/master -b master
    git merge graphene/api-123
    git push graphene HEAD:master

- Anything in `graphene/master` can be merged into chain branches:

    git fetch bts2
    git checkout bts2/master -b master
    git merge graphene/api-123
    git push graphene HEAD:master

### When to update a chain's `master`

The Backstory:  I'm not sure what I should put in `$CHAIN/master`.

This can actually be *mechanically determined*:

    git merge-base graphene/master bts2/bitshares

### When to update `graphene/stable`

The Backstory:  So what's the deal with `graphene/stable` then?

- It's the most recent common ancestor of `graphene/master` and all other chains, i.e.:

    git merge-base --octopus graphene/master bts2/bitshares muse/muse otherchain/otherchain ...

- If the current `stable` commit is different from the output of the above command, `stable` should be updated.

### Fixing an un-merged mistake

The Backstory:  OK, I screwed up.  Something bad happens like `api.cpp` does not compile, or running it formats the user's disk.
I already made a commit.  What do I do?

- *If you have not merged your commit to master* you can rewrite the commit.

    git commit --amend
    git push -f origin HEAD:api-123

### Fixing a multiple-commit un-merged mistake

The Backstory:  OK, I screwed up and made two commits.  What do I do?

- *If you have not merged your commits to master* you can rewrite the commits.  Let's say you created a new API call that doesn't compile.

Your history looks like this:

    commit 2c5d85c0ceaef022bdd783720503cfe11a493782
    Author: me <me@localhost>

        Fix the API call

    commit 5b457a9f305f585bda6c5b2dcd40c12235b8f2bc
    Author: me <me@localhost>

        Create a new API call that doesn't compile

    commit a4382631cb3f3343e18c5894dcc818e7f3497b06
    Author: coredev <coredev@coredevbox>

        This is somebody else's code

The middle commit is useless because it does not compile.
What you really want to do is combine the last two commits into a single patch.
The two commits you want to combine need not be adjacent in your history (although
they are adjacent in this example).
You can combine commits using the interactive mode of `git rebase`:

    git rebase -i a43826

This will provide you with an editor containing a list of commits since the listed commit, like this:

    pick 5b457a9 Create a new API call that doesn't compile
    pick 2c5d85c Fix the API call

If the commits you are combining are not adjacent in the history, you will need to
reorder your commits so that they are adjacent; simply cut and paste the second commit immediately below
the first commit.

Once your two commits are adjacent, you can change the "pick" for the second commit into "squash", like this:

    pick 5b457a9 Create a new API call that doesn't compile
    squash 2c5d85c Fix the API call

Then save and quit the editor.  A new editor prompt will appear asking you to create a message for the new commit,
already containing the messages of squashed commits.  (If you would like the combined commit to have the same message
as the first commit, you can use the "fixup" command instead of "squash".)  Interactive rebase is very powerful.

Rebase works by throwing away your commits and replacing them with different commits.
That means you need to force-push your commits:

    git push -f origin HEAD:api-123

### Fixing a merged mistake

If you want to fix a mistake that has been merged, you just need to create a new commit.

    git checkout bad-commit
    notepad libraries/app/api.cpp
    git add libraries/app/api.cpp
    git commit -m "api.cpp now compiles"
    git push origin HEAD:api-123

### Has a commit been merged?

First, find the merge base:

    git merge-base HEAD origin/master

Any commit on or before the `merge-base` has been merged and should not be replaced.

### Fixing whitespace

Extra whitespace at the ends of lines should not go into any commit.

    git rebase --whitespace=fix origin/stable

If you want to keep yourself from writing such commits in the first place, `git` comes with
a handy pre-commit hook which will yell if you attempt to write such a commit.  Try this command:

    cp .git/hooks/pre-commit.sample .git/hooks/pre-commit

### Writing a feature that depends on other features

The Backstory:  I want to write a patch `api-456` which depends on
`bugfix-123` and `wallet-345` which haven't made it into `stable` yet.

    git checkout graphene/stable -b api-456
    git merge graphene/bugfix-123
    git merge graphene/wallet-345
    # write your patch...
    git add libraries/api/api.cpp
    git commit -m "Now the API is even better"

### Putting a commit on its own branch

The Backstory:  I did an API improvement, then I fixed a bug.  These are two unrelated things and should be in two different branches.  However my history looks like this:

    commit 2c5d85c0ceaef022bdd783720503cfe11a493782
    Author: me <me@localhost>

        Fix a bug

    commit 5b457a9f305f585bda6c5b2dcd40c12235b8f2bc
    Author: me <me@localhost>

        Create a new API call

    commit a4382631cb3f3343e18c5894dcc818e7f3497b06
    Author: coredev <coredev@coredevbox>

        This is graphene/stable

- First we create a new `bugfix` branch with just the bugfix commit based on `graphene/stable` and cherry-picking the commit:

    git checkout graphene/stable -b bugfix-456
    git cherry-pick 2c5d85
    git push -f graphene HEAD:bugfix-456

- Then we fix the API branch by backing up to `5b457`:

    git checkout api-123
    git reset --hard 5b457
    git push -f graphene HEAD:api-123

- You shouldn't do this if your code has not been merged to `master`!  If your code has been merged, well, now the bugfix spuriously depends on the API call.  Living with the spurious dependency is easier than cherry-picking into master.

- If you're worried that this violates the "avoid cherry-picking" guideline of [this blog post](http://www.draconianoverlord.com/2013/09/07/no-cherry-picking.html), you should read the post more carefully.  In particular, the title states that "If you cherry pick, your branch model is wrong."  In this case, you broke the branching model when you made the bugfix commit without creating a new branch from stable; now you need to cherry-pick. The need to cherry-pick to get back to a more policy-compliant state is an indication that you did something wrong (namely, breaking the branching model).

- This should only be a very minor problem.  Project policy states that the `api-` branches and `develop` can be reset at any time, so even if you've pushed commits like this, they can still be rewound.

- If it has been merged to `master` it is a slightly less minor problem, but still not very significant in the scheme of things.  We should still be able to avoid many cases of this happening, as a branch should not pass review if it creates this kind of dependency across multiple unrelated commits.
