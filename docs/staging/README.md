# Kite SDK Guide

Kite is a set of libraries, tools, examples, and documentation focused on
making it easier to build applications on top of the Hadoop ecosystem.

This repository contains the Kite SDK Guide.

## Building

### Setup

If you haven't already done so, [install Jekyll](http://jekyllrb.com/docs/installation/) on your system. You may also
need to install Coderay for syntax highlighting:

```
sudo gem install coderay
```

Then, you can build the current version of the documentation by executing:

```
jekyll build
```

This will compile the documentation into HTML in a directory called \_site.

### Maven content

Some pages depend on documentation generated by maven. To include the content for these pages, generate the maven site from the root of a [kite](http://github.com/kite-sdk/kite) working copy. The command to run the maven site plugin is:

```
export MAVEN_OPTS="$MAVEN_OPTS -Xmx1g"
mvn post-site site:stage -DstagingDirectory=/tmp/kite-stage -DtopSiteURL=http://kite-sdk.org
```

Next, link the created docs into the HTML includes folder:

```
ln -s /tmp/kite-stage/docs/<version> _includes/current
```

Pages will be built using content from the staged site when the site is generated.

## Debugging

You can view a live version of the documentation by executing:

```
jekyll serve --baseurl ''
```

and then viewing the site at [localhost:4000](http://localhost:4000).

## Releasing

### Prepare the content

1. Ensure the the version section of [`_config.yml`](_config.yml) is correct for the current Kite version.
2. Ensure the `_includes/current` link points to the maven site built for the current Kite version.
3. Update the `baseurl` for the site you will deploy. For example, `/docs/staging` or `/docs/<version>`
4. Build the documentation site: `jekyll build`

### Deploy a staging site

1. Check out the `gh-pages` branch of the [kite repository](https://github.com/kite-sdk/kite)
2. Delete the `docs/staging/` directory if it exists.
```
cd docs
rm -rf staging/
```
3. Copy the `_site` directory to `docs/staging/`
```
cp -r ../../kite-docs/_site/ staging
```
4. Copy `_includes/current/apidocs/` into `docs/staging/`
```
cd staging
cp -r ../../../kite-docs/_includes/current/apidocs/ apidocs
```
5. Copy `_includes/current/jdiff/` into `docs/staging/`
```
cp -r ../../../kite-docs/_includes/current/jdiff/ jdiff
```
6. Commit and push the staging site.
```
git add .
git commit -m "<RELEASE ISSUE>: Deploying staging site"
```

### Deploying live site

1. Stage the current master and ensure it is correct
2. Update `_config.yml` to the correct `baseurl` for the current Kite version, `/docs/<version>`.
3. Builde the documentation site with `jekyll build`
4. Delete the `docs/staging/` directory if it exists.
```
cd docs
rm -rf <version>/
```
5. Copy the `_site` directory to `docs/<version>/`
```
cp -r ../../kite-docs/_site/ staging
```
6. Copy `_includes/current/apidocs/` into `docs/staging/`
```
cd staging
cp -r ../../../kite-docs/_includes/current/apidocs/ apidocs
```
7. Copy `_includes/current/jdiff/` into `docs/staging/`
```
cp -r ../../../kite-docs/_includes/current/jdiff/ jdiff
```
8. Commit and push the staging site.
```
git add .
git commit -m "<RELEASE ISSUE>: Deploying staging site"
```

### Release script

```bash
#!/bin/bash

REMOTE=origin
GH_PAGES=/path/to/kite-gh-pages
KITE_DOCS=/path/to/kite-docs
VERSION=staging
RELEASE_ISSUE=<release-issue-id>

# remove the target if it exists
cd $GH_PAGES/docs
rm -rf $VERSION

# copy the site content
cp -r $KITE_DOCS/_site $VERSION
cd $VERSION
cp -r $KITE_DOCS/_includes/current/apidocs/ apidocs
cp -r $KITE_DOCS/_includes/current/jdiff/ jdiff

# prepare a commit with the changes
git add .
git commit -m "$RELEASE_ISSUE: Deploying $VERSION site"
git push $REMOTE master
```