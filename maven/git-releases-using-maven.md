# Git Releases Using Maven

Deploying a new version of a Maven application typically includes three tasks:
	
1. Versioning and tagging in the Git source code repository (git or other)
2. Building and publishing the package or image to an official registry
3. Deploying from the registry into the server(s) where the app runs

This article covers the first task only. That is, only the Git changes.

## Versions: Release or Snapshot?

First things first: what is a snapshot version and what is a release version?

A *Release* version represents an official version of an application that is or is going to be
published in a package or image registry for QA purposes, for deployment to production, or any other official usage.
According to the [Semantic Versioning Standard](https://semver.org/) an official release 
version number could take the form:

    2.56.3

On the other hand a *Snapshot* version represents unfinished work that aims to become a version in the future.
For example, if we are currently working on the code that we expect to become the release
4.3.15 then the current snapshot version would be:

    4.3.15-SNAPSHOT

The standard indicates to always append "-SNAPSHOT" (all upper case) to the future, not-yet-existent 
release version. When a snapshot version is released (maybe every week, every night, or even every hour)
it represents the latest unstable version of an app. It's meant to be rapidly available to members of
the team, but it's not meant for official QA or demos.

## Versioning Using the Maven Release Plugin

The Maven Release Plugin automatically performs Git Releases. That is, it automatically upgrades versions
and tags the source code repository for us. All in one go in a controlled manner. To do this we'll need:

1. Make sure git command line is installed and the SSH keys to the git repository in question are also installed
2. Use a snapshot version in the `pom.xml` file
3. Tell Maven where Git repository is
4. Make sure all changes are committed to the Git repository

These items are explained below.

### 1. Git Can be Used from the Command Line

You'll need to make sure the "git" command is installed, and that the SSH keys for the project(s) in question
are also installed in the `<home>/.ssh` folder. Both subjects are vastly explained in many articles online.
Once that is ready you can test it by going to a folder where a git project was cloned to and type:

```bash
git pull
```

You should see something like `Already up to date` message or other message but not an authentication error. 

### 2. Use a Snapshot Version

make sure our pom.xml file will always use a snapshot version. For example, our pom.xml file could look like:

```xml
<project ...>
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.myorg</groupId>
  <artifactId>myapp</artifactId>
  <version>4.3.15-SNAPSHOT</version>
```

Use `4.3.15-SNAPSHOT`, not `4.3.15` here.

### 3. Tell Maven where Git Repository Is

Get the URL you used to clone the repository and add it to the pom.xml file, right after the `<properties>` tag, by adding:

```xml
  <scm>
    <developerConnection>scm:git:<your-clone-url-here></developerConnection>
    <tag>HEAD</tag>
  </scm>
```

**Note**: This is the developer git URL that has write permissions to the repository, not a read-only git URL.

### 4. Commit all changes to git

Use the standard commit/push commands to make sure you have committed all changes to the repository, including the pom.xml file.

## Versioning and Tagging in Git

Performing a Git Release is as simple as running:

```bash
mvn release:perform
```

This command will:

- Check there are no uncommitted local changes
- Validate that the version is a -SNAPSHOT one, and that the `<scm>` tag is ready
- Then, it will switch versions and tag the repo. That is:
    - a) Will remove the `-SNAPSHOT` part of the version in the pom.xml file and will commit and push this change to the repository
    - b) Will create a Git tag with in the form of `application-version` (as in "myapp-4.3.15"). No SNAPSHOT here.
    - c) Will increase the version number to the next snapshot version (as in "4.3.16-SNAPSHOT") and will commit this change to the git repository. The patch moved from 15 to 16 now.

That's it.

After the release is performed in Git the source code will be using the next snapshot version, and the git repository will have a commit for the
release version that will be tagged appropriately.

At this point any automated build tool can now go to the Git repository search for the tagged release, retrieve the source code, build and publish it in
a registry.

Again, this functionality does not publish to a registry, it only manages the version and tags in Git.



















