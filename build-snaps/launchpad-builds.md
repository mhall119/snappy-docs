---
title: Build on Launchpad
---

Here are the steps needed to start using Launchpad to build Snap packages when your snapcraft.yaml lives in a separate git repo from your main codebase.

In this document we will use `MyApp` to refer to the main codebase, and `MyApp-pkg` to refer to the package config repo.

# Setup your package branch

In order for Launchpad to find your snapcraft.yaml file, it needs to be in one of the following locations under your branch root:
* snap/snapcraft.yaml
* snapcraft.yaml
* .snapcraft.yaml

# Push MyApp-pkg to Launchpad

You'll need to create a git remote for the code's location on Launchpad. This can be either a personal repo or one associated with a Launchpad project.

### For personal:

`git remote add launchpad git+ssh://://OWNER@git.launchpad.net/~OWNER/+git/REPOSITORY`

### For project:
`git remote add launchpad git+ssh://://OWNER@git.launchpad.net/~OWNER/PROJECT/+git/REPOSITORY`

Where OWNER is your Launchpad username, PROJECT is your Launchpad project, and REPOSITORY is any name you'd like (but probably `myapp-pkg`).

You'll need to have your SSH public key used for this added to your Launchpad profile so it can do the authentication that way.

For more info about Git on Launchpad: https://help.launchpad.net/Code/Git

# Configure a Snap build

Once you've pushed the code to the Git repo on Launchpad, you can create a build config for it. On the git branch (not just repo) page you'll see a **Create snap package** link that will take you to a form where you can define your snap builder.

Here it's best to stick with "Ubuntu Xenial, for Ubuntu Core 16" as your build series, unless you specifically need something only available in a later release of Ubuntu. Then choose what build architectures you want to target (you will only be able to start builds for architectures you've chosen here).

If your snapcraft.yaml was in the same branch as your application's code, you could select **Automatically build when branch changes** here to start builds on ever commit. But because the `MyApp-pkg` branch isn't going to change when the main `MyApp` code gets updated, you'll need to trigger it externally instead.

Next, select **Automatically upload to store**. Below that choose `Edge` for your channel, so that these automated builds are only available to people who intentionally subscribe to this bleeding-edge channel. You'll be shown how to promote releases into the `Stable` channel below.

Finally click **Create snap package** to save your build config and kick it off.

# Trigger Snap builds from CI

Once you've done the above you can manually trigger new builds, and any successful one will be published into the `edge` channel of the store, and be installable with `snap install --edge myapp`.

But what you're really after is making this automatic. Since the packaging config lives separate from the main `MyApp` code, you'll need some way of kicking builds off when the main code repo is updated.

There's a simple command-line tool called `lp-build-snap` that will let you do just that. It will find the build config you created in the previous step, and tell launchpad to start a new build for it.

To get it, `snap install lp-build-snap` and then `lp-build-snap --help` to see how to use it.

You can run this from your existing CI process whenever a new commit lands in master. The first time you run the command it will make you do an authorization step in a webbrowser to get access credentials. These credentials will store in `~/snap/lp-build-snap/current/snap-builds/credentials`, and you should safeguard them if you don't have strict control over your CI environment. After you've authorized the tool it can be called by your CI scripts to kick off snap builds whenever a new commit lands.

# Publish and promote builds

Once your snap is built and in the store, you can manage what channels
it's published to from any computer with the `snapcraft` tool installed.

To see what revisions of the snap package are in each of the channels:
`snapcraft status myapp`

To promote a revision to the publicly visible "stable" channel:
`snapcraft release myapp <revision> stable`

There's a video showing what else you can do with the store's channels
here: https://www.youtube.com/watch?v=-3b9qkl9Z_k


