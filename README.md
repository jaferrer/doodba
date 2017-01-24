# [Dockerized Odoo Base Image](https://hub.docker.com/r/tecnativa/odoo-base)

[![](https://images.microbadger.com/badges/version/tecnativa/odoo-base:latest.svg)](https://microbadger.com/images/tecnativa/odoo-base:latest "Get your own version badge on microbadger.com")
[![](https://images.microbadger.com/badges/image/tecnativa/odoo-base:latest.svg)](https://microbadger.com/images/tecnativa/odoo-base:latest "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/commit/tecnativa/odoo-base:latest.svg)](https://microbadger.com/images/tecnativa/odoo-base:latest "Get your own commit badge on microbadger.com")
[![](https://images.microbadger.com/badges/license/tecnativa/odoo-base.svg)](https://microbadger.com/images/tecnativa/odoo-base "Get your own license badge on microbadger.com")

[![](https://images.microbadger.com/badges/version/tecnativa/odoo-base:9.0.svg)](https://microbadger.com/images/tecnativa/odoo-base:9.0 "Get your own version badge on microbadger.com")
[![](https://images.microbadger.com/badges/image/tecnativa/odoo-base:9.0.svg)](https://microbadger.com/images/tecnativa/odoo-base:9.0 "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/commit/tecnativa/odoo-base:9.0.svg)](https://microbadger.com/images/tecnativa/odoo-base:9.0 "Get your own commit badge on microbadger.com")

[![](https://images.microbadger.com/badges/version/tecnativa/odoo-base:10.0.svg)](https://microbadger.com/images/tecnativa/odoo-base:10.0 "Get your own version badge on microbadger.com")
[![](https://images.microbadger.com/badges/image/tecnativa/odoo-base:10.0.svg)](https://microbadger.com/images/tecnativa/odoo-base:10.0 "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/commit/tecnativa/odoo-base:10.0.svg)](https://microbadger.com/images/tecnativa/odoo-base:10.0 "Get your own commit badge on microbadger.com")

Highly opinionated image ready to put [Odoo](https://www.odoo.com) inside it,
but **without Odoo**.

## What?

Yes, the purpose of this is to serve as a base for you to build your own Odoo
project, because most of them end up requiring a big amount of custom patches,
merges, repositories, etc. With this image, you have a collection of good
practices and tools to enable your team to have a standard Odoo project
structure.

BTW, we use [Alpine][]. I hope you like that.

  [Alpine]: https://alpinelinux.org/

## Why?

Because developing Odoo is hard. You need lots of customizations, dependencies,
and if you want to move from one version to another, it's a pain.

Also because nobody wants Odoo as it comes from upstream, you most likely will
need to add custom patches and addons, at least, so we need a way to put all
together and make it work anywhere quickly.

## How?

You can start working with this straight away with our [scaffolding][].

## Image usage

Basically, every directory you have to worry about is found inside `/opt/odoo`.
This is its structure:

    custom/
        entrypoint.d/
        build.d/
        conf.d/
        src/
            private/
            odoo/
            addons.yaml
            repos.yaml
    common/
        entrypoint.sh
        build.sh
        entrypoint.d/
        build.d/
        conf.d/
    auto
        addons/
        odoo.conf

Let's go one by one.

### `/opt/odoo/custom`: The important one

Here you will put everything related to your project.

#### `/opt/odoo/custom/entrypoint.d`

Any executables found here will be run when you launch your container, before
running the command you ask.

#### `/opt/odoo/custom/build.d`

Executables here will run just before those in `/opt/odoo/common/build.d`.

#### `/opt/odoo/custom/conf.d`

Files here will be environment-variable-expanded and concatenated in
`/opt/odoo/auto/odoo.conf` at build time.

#### `/opt/odoo/custom/src`

Here you will put the actual source code for your project.

When putting code here, you can either:

- Use [`repos.yaml`][], that will fill anything at build time.
- Directly copy all there.

Recommendation: use [`repos.yaml`][] for everything except for [`private`][],
and ignore in your `.gitignore` and `.dockerignore` files every folder here
except [`private`][], with rules like these:

    odoo/custom/src/*
    !odoo/custom/src/private
    !odoo/custom/src/*.*

##### `/opt/odoo/custom/src/odoo`

**REQUIRED.** The source code for your odoo project.

You can choose your Odoo version, and even merge PRs from many of them using
[`repos.yaml`][]. Some versions you might consider:

- [Original Odoo][], by [Odoo S.A.][].

- [OCB][] (Odoo Community Backports), by [OCA][].
  The original + some features - some stability strictness.

- [OpenUpgrade][], by [OCA][].
  The original, frozen at new version launch time + migration scripts.

##### `/opt/odoo/custom/src/private`

**REQUIRED.** Folder with private addons for the project.

##### `/opt/odoo/custom/src/repos.yaml`

A [git-aggregator](#git-aggregator) configuration file.

##### `/opt/odoo/custom/src/addons.yaml`

One entry per repo and addon you want to activate in your project. Like this:

```yaml
# Using `all` links all addons in a repository (not recommended)
server-tools: all

# List all addons you want per repository (recommended)
website:
    - website_legal_page
web:
    - web_responsive
```

You can bundle [several YAML documents][] if you want to logically group your
addons and some repos are repeated among groups, by separating each document
with `---`:

```yaml
# Spanish Localization
l10n-spain:
    - l10n_es
server-tools:
    - date_range
---
# SEO tools
website:
    - website_blog_excertp_img
server-tools: # Here we repeat server-tools, but no problem because it's a
              # different document
    - html_image_url_extractor
    - html_text
```

Important notes:

- Do not add repos for the required [`odoo`][] and [`private`][] directories;
  those are automatic.

- Only addons here are symlinked in [`/opt/odoo/auto/addons`][]. Addons from
  the required directories above are added directly in the [`odoo.conf`][]
  file.

- This means that if you have an addon with the same name in any of [`odoo`][],
  [`private`][] or [`/opt/odoo/auto/addons`][] directories, this will be the
  importance order in which they will be loaded (from most to least important):

  1. Addons in [`private`][].
  2. Custom addons listed in [`addons.yaml`][].
  3. Core Odoo addons from [`./odoo/addons`][`odoo`].

  Although it is better to simply have no name conflicts if possible.

- Any other addon not listed here will not be usable in Odoo (and will be
  removed by default, to keep the resulting image thin).

- If you list 2 addons with the same name, you'll get a build error.

### `/opt/odoo/common`: The useful one

This folder is full of magic. I'll document it some day. For now, just look at
the code.

Only some notes:

- Will compile your code with [`PYTHONOPTIMIZE=2`][] by default.

- Will remove all code not used from the image by default (not listed in
  `/opt/odoo/custom/src/addons.yaml`), to keep it thin.

### `/opt/odoo/auto`: The automatic one

This directory will have things that are automatically generated at build time.

#### `/opt/odoo/auto/addons`

It will be full of symlinks to the addons you selected in [`addons.yaml`][].

#### `/opt/odoo/auto/odoo.conf`

It will have the result of merging all configurations under
`/opt/odoo/{common,custom}/conf.d/`, in that order.

## The `Dockerfile`

I will document all build arguments and environment variables some day, but for
now keep this in mind:

- This is just a base image, full of tools. **You need to build your project
  subimage** from this one, even if your project's `Dockerfile` only contains
  these 2 lines:

      FROM tecnativa/odoo-base
      MAINTAINER Me <me@example.com>

- The above sentence becomes true because we have a lot of `ONBUILD` sentences
  here, so at least **your project must have a `./custom` folder** along with
  its `Dockerfile` for it to work.

- All should be magic if you adhere to our opinions here. Just put the code
  where it should go, and relax.

## Bundled tools

### `log`

Just a little shell script that you can use to add logs to your build or
entrypoint scripts:

    log INFO I'm informing

### `unittest`

Another little shell script, useful for debugging. Just run it like this and
Odoo will execute unit tests in its default database:

    unittest my_addon,my_other_addon

### [`psql`](https://www.postgresql.org/docs/9.5/static/app-psql.html)

Environment variables are there so that if you need to connect with the
database, you just need to execute:

    docker exec -it your_container psql

### [`wdb`](https://github.com/Kozea/wdb/)

In our opinion, this is the greatest Python debugger available, mostly for
Docker-based development, so here you have it preinstalled.

I told you, this image is opinionated. :wink:

**DO NOT USE IT IN PRODUCTION ENVIRONMENTS.** (I had to say it).

### [`git-aggregator`](https://pypi.python.org/pypi/git-aggregator)

We found this one to be the most useful tool for downlading code, merging it
and placing it somewhere.

We will use
[a development version](https://github.com/acsone/git-aggregator/tree/bdfdf05a7e903b06c201f163161053a924330bed)
until the stable one [allows you to choose a `--depth` when pulling
images](https://github.com/acsone/git-aggregator/pull/7), and [fetches only the
required remotes](https://github.com/acsone/git-aggregator/pull/6).

#### Example [`repos.yaml`][] file

This example merges [several sources][`odoo`]:

    ./odoo:
        defaults:
            # Shallow repositores are faster & thinner
            depth: 1000
        remotes:
            ocb: https://github.com/OCA/OCB.git
            odoo: https://github.com/odoo/odoo.git
        target:
            ocb 9.0
        merges:
            - ocb 9.0
            - odoo refs/pull/13635/head

### [`odoo`](https://www.odoo.com/documentation/10.0/reference/cmdline.html)

We set an `$ODOO_RC` environment variable pointing to [the autogenerated
configuration file](#optodooautoodooconf) so you don't have to worry about
it. Just execute `odoo` and it will work fine.

Note that version 9.0 has an `odoo` binary to provide forward compatibility
(but it has the `odoo.py` one too).

## Scaffolding

Get up and running quickly with the provided
[scaffolding](https://github.com/Tecnativa/docker-odoo-base/tree/scaffolding).

### Skip the boring parts

I will assume you know how to use Git, Docker and Docker Compose.

    git clone -b scaffolding https://github.com/Tecnativa/docker-odoo-base.git myproject
    cd myproject
    docker-compose -f setup-devel.yaml up
    docker-compose -f devel.yaml up

And if you don't want to have a chance to do a `git merge` and get possible
future scaffolding updates merged in your project's `git log`:

    rm -Rf .git
    git init

### Tell me the boring parts

The scaffolding provides you a boilerplate-ready project to start developing
Odoo in no time.

#### Environments

This scaffolding comes with some environment configurations, ready for you to
extend them. Each of them is a [Docker Compose
file](https://docs.docker.com/compose/compose-file/) almost ready to work out
of the box (or almost), but that will assume that you understand it and will
modify it.

After you clone the scaffolding, **search for `XXX` comments**, they will help
you on making it work.

##### Development

Set it up with:

    docker-compose -f setup-devel.yaml up

Once finished, you can start using Odoo with:

    docker-compose -f devel.yaml up --build

This is on purpose. It allows you to track only what Git needs to track and
provides faster Docker builds.

##### Production

This environment is just a template. **It is not production-ready**. You must
change many things inside it, it's just a guideline.

It includes pluggable `smtp` and `backup` services.

Once you fixed everything needed, run it with:

    docker-compose -f prod.yaml up --build

##### Testing

A good rule of thumb is test in testing before uploading to production, so this
environment tries to imitate the production one in everything, but *removing
possible pollution points*:

- It has no `smtp` service.

- It has no `backup` service.

Test it in your machine with:

    docker-compose -f test.yaml up --build

## FAQ

### Why my `99-whatever.sh` script in `/opt/odoo/*/*.d/` does not execute?

Files must be executable and have no `.` in their name.

### This project is too opinionated, but can I question any of those opinions?

Of course. There's no guarantee that we will like it, but please do it. :wink:

### What's this `hooks` folder here?

It runs triggers when doing the automatic build in the Docker Hub.
[Check this](https://hub.docker.com/r/thibaultdelor/testautobuildhooks/).

### Can I have my own [scaffolding][]?

You probably **should**, and rebase on our updates. However, if you are
planning on a general update to it that you find interesting for the
general-purpose one, please send us a pull request.

### How can I help?

Just [head to our project](https://github.com/Tecnativa/docker-odoo-base) and
open an issue or pull request.


[Original Odoo]: https://github.com/odoo/odoo
[Odoo S.A.]: https://www.odoo.com
[OCB]: https://github.com/OCA/OCB
[OCA]: https://odoo-community.org/
[OpenUpgrade]: https://github.com/OCA/OpenUpgrade/
[`PYTHONOPTIMIZE=2`]: https://docs.python.org/2/using/cmdline.html#envvar-PYTHONOPTIMIZE
[scaffolding]: #scaffolding
[`odoo`]: #optodoocustomsrcodoo
[`odoo.conf`]: #optodooautoodooconf
[`private`]: #optodoocustomsrcprivate
[`repos.yaml`]: #optodoocustomsrcreposyaml
[several YAML documents]: http://www.yaml.org/spec/1.2/spec.html#id2760395
[`addons.yaml`]: #optodoocustomsrcaddonstxt
[`/opt/odoo/auto/addons`]: #optodooautoaddons
