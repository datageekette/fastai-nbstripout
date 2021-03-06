# fastai-nbstripout

A much faster version of nbstripout with a slightly different set of features, and autotrust notebooks local git configuration

## About

This is a rewrite of [nbstripout](https://github.com/kynan/nbstripout). It's much faster because it doesn't load any of the very heavy `nbformat`, and operates on the json file directly.

It's used in all [fastai](https://github.com/fastai/) projects, and this repo was created to make it easy to re-use it in other projects, so all files are in one place.

## As is software - not identical to nbstripout's functionality

This tool implements only a sub-set of nbstripout (most of it) and makes no attempt to neither be identical nor try to keep it in sync with nbstripout. It implements the parts we needed for fastai needs.

This repository's purpose is not to maintain a different implementation of nbstripout, but to make it easy to integrate the existing functionality into other fastai projects, since it involves quite a few files. 

Therefore please submit no PRs or Issues unless you found a bug in the current implementation. 

If you'd like to create and maintain a faster version of nbstripout with all the features it provides, please feel free to fork this implementation and build upon it.

## Structure

1. The strip out tool itself is: `tools/fastai-nbstripout`

2. The helper tool `tools/trust-origin-git-config` autogenerates `.gitconfig`, which in turn configures the action of the git filters and tells git to trust this local config file. (`git config --local include.path ../.gitconfig`). Here is the usage instructions:

   ```
   tools/trust-origin-git-config -h
   usage: trust-origin-git-config [-h] [-e] [-d] [-t]

    optional arguments:
      -h, --help     show this help message and exit
      -e, --enable   Trust repo-wide .gitconfig (default action)
      -d, --disable  Distrust repo-wide .gitconfig
      -t, --test     Validate repo-wide .gitconfig config
   ```

   Therefore, to enable the setup, you run:
   ```
   tools/trust-origin-git-config -e
   ```
   and to disable:
   ```
   tools/trust-origin-git-config -d
   ```
   You will want to add `.gitconfig` to `.gitignores`, since it's autogenerated (the included in this repo `.gitignores` already does that).

3. Now, all that remains is to configure directories with jupyter notebooks to be stripped out. This is done via the `.gitattributes` file placed in the desired directories.

   Currently, `fastai-nbstripout` supports two stripout configurations:

   This `.gitattributes` will strip out all the unnecessary bits and keep the `output`s:
   ```
   *.ipynb filter=fastai-nbstripout-code
   *.ipynb diff=ipynb-code
   ```
   You can see this setup and its effects under `with-outputs` directory.

   This `.gitattributes` will strip out all the unnecessary bits, including the `output`s:

   ```
   *.ipynb filter=fastai-nbstripout-docs
   *.ipynb diff=ipynb-docs
   ```
   You can see this setup and its effects under `without-outputs` directory.

   These settings apply recursively to all sub-dirs.

You will need to `git add` all these files to your git repo before activating the setup.

## Bonus: Auto-trust the notebooks

Normally, modified outside of jupyter environment notebooks lose their "trusted" state, so you won't be able to run them automatically and will need to manually set them to be trusted. The following setup does it automatically for you on `git pull`.

1. Configure which directories you want to be "auto-trusted" by editing `tools/trust-doc-nbs`. E.g. in this sample project you will find:

   ```
   trust_nbs('without-outputs')
   trust_nbs('with-outputs')
   ```

2. `tools/trust-doc-nbs` is a hook that runs automatically on `git pull`, and `tools/trust-doc-nb-install-hooks` is the tool that configures that hook. So you need to run it once after `git clone`.

## One tool to rule them all

Who's going to remember all these tools to be run after `git clone`. That's why there is a wrapper `tools/run-after-git-clone` that runs all the other tools that configure git to do the right thing. And it's easy to remember. Of course, you don't have to use it and you can run each tool separately.

## Why can't the configuration process be automated.

Every user, that needs to have these configured, has to run the configuration scripts manually once upon the first clone of the repository. Which sucks, because some users will forget and commit unstripped out notebooks.

Unfortunately, due to the way git security is set up, there is no other way to go about it. The only way to catch unstripped out notebook committed is to have server-side git hooks, but github doesn't allow those.

Change the instructions for your projects to include the local git setup. For example, for this project it'd become:

   ```
   git clone https://github.com/fastai/fastai-nbstripout
   cd fastai-nbstripout
   tools/run-after-git-clone
   ```

The way I personally deal with me forgetting to run the local git setup is to always run `git diff` before `git commit`, it takes a few seconds and I catch my forgetfulness this way. If the output contains `execution_count` that is not `null`:

   ```
      {
       "cell_type": "code",
   -   "execution_count": null,
   +   "execution_count": 1,
   ```
That means I don't have it configured.
