# Release Process For Java Components

**NOTES:**

* This process covers major and minor releases only. Bug-fix releases, which increment the third digit, are performed on a A.B.X branch and not on master, but otherwise are similar.
* Some of these operations can be performed either on the Command-Line or in your IDE, whatever you prefer.

## Preparation

### Check for Code Completeness
* Confirm that all __temporary__ branches are checked into master and/or deleted, both local and remote.
* Confirm any new bug fixes have corresponding tests
* At major version releases, search for deprecated code and remove at __Major Versions__ only.
    * `find . -name "*.java" -type f -print | xargs grep -i -n -s -A0 "deprecated"`
    * **Note:** When first marking a segment of code deprecated, please add the current version number. This will make it easier to know when to remove the deprecated code.

### Check Maven Plugin, Dependency, Property Versions of the POM:
* `mvn versions:display-plugin-updates`
* `mvn versions:display-dependency-updates`
* `mvn versions:display-property-updates`

### Visual Checks for Correctness
* LICENSE
* NOTICE -- check for copyright dates
* README.md
* .asf.yaml
* .travis.yml (if used)
* .gitattributes -- used to exclude files from release zip, assumes .gitignore
* .github/workflows
* .gitignore -- used to exclude files from origin
* pom.xml / apache-rat-plugin config -- checks for license headers, assumes .gitignore
* pom.xml

### Run Maven Tests
* `mvn apache-rat:check`
* `mvn clean test`
* `mvn clean test -P strict`
* `mvn clean test -P check-cpp-files`
* `mvn clean javadoc:javadoc`
* `mvn clean install -DskipTests=true`
* Check that the /target/ directory has 5 jars: (may need to refresh)
    * datasketches-\<component\>-SNAPSHOT-javadoc.jar
    * datasketches-\<component\>-SNAPSHOT-sources.jar
    * datasketches-\<component\>-SNAPSHOT-test-sources.jar
    * datasketches-\<component\>-SNAPSHOT-tests.jar
    * datasketches-\<component\>-SNAPSHOT.jar
* Check your local Maven local repository
    * _~/.m2/repository/org/apache/datasketches/datasketches-\<component\>/A.B.0-SNAPSHOT/_ 
    * It should have 5 new jars and a .pom file. 

### Run IDE Checks
* Run Code Coverage > 90%
* SpotBugs checks (is it properly configured?)
* Checkstyle (is it properly configured?)

## Create Permanent Release Branch & Update POM Version

### Make sure GPG-Agent is running and GitHub is clean
* From Command Line at Component root:
    * `eval $(gpg-agent --daemon)`
        * if it is not running it will start it.
        * if it is already running you will see something like:
        * `gpg-agent: a gpg-agent is already running - not starting a new one`  

    * Confirm GitHub repository is current and git status is clean:
        * `git status` # should return:
        * "On branch master, your branch is up to date with 'origin/master', nothing to commit, working tree clean."
     
### Create New Release Branch
* Assume current master POM version = A.B.0-SNAPSHOT
* Create new __Permanent Branch__: "A.B.X"
    * Note: This assumes a normal progression of release numbers. However, when moving to a new major release the current A.B.0-SNAPSHOT will be followed by a new __Permanent Branch__: A'.0.X, where A' = A + 1.

### Edit Release Branch POM Version to Target Release Version 
* Switch to new Release Branch
* Edit pom.xml version to A.B.0 (remove -SNAPSHOT, do not change A or B) in case of normal progression, or A'.0.0 in the case of a new major release.
* Commit the change. __DO NOT PUSH!__
* Create Annotated TAG for this commit: A.B.0-RC1 (or RCn) or A'.0.0-RC1
* Write down the Git hash : example: 40c6f4f (you will need it later)
* __Now Push__ Branch  "A.B.X" with edited pom.xml to origin __NEVER MERGE THIS PERMANENT BRANCH INTO MASTER__
* Do explicit push of tags on new branch A.B.X (or A'.0.X) to origin:
    * `git push origin --tags`

### Confirm new Release Branch, Tag and Git hash
* From a web browser at origin web site: github.com/apache/datasketches-\<component\>
    * Select the A.B.X branch or A'.0.X
    * Confirm that the tag: A.B.0-RC1 (or A'.0.0-RC1) exists and that the tag is on the latest commit and with the correct Git hash.
    * __DO NOT CREATE PR OR MERGE THIS PERMANENT BRANCH INTO MASTER__
* From IDE or Command Line:
    * Confirm that the tag A.B.0-RC1 and the branch A.B.X, (or A'.0.0-RC1 and the branch A'.0.X) and HEAD coincide with the correct Git hash.
    * Confirm that there are no unstaged or staged changes.
    * Return to master branch

### Edit Master Branch with the SNAPSHOT Version of the Next Release
* Edit master pom.xml to A'.B'.0-SNAPSHOT where A' or B' will be incremented by 1. (Bug fix releases will change the 3rd digit)
* Commit and Push this change to origin/master with the comment "Release Process: Change pom version to A'.B'.0-SNAPSHOT."
    * This may require changing to a temparary branch and creating a PR to be approved if master branch is restricted. 
* Create a tag A'.B'.0-SNAPSHOT on master at the HEAD.
* Push the tag to origin: `git push origin --tags`
* Return to release branch A.B.X (or A'.0.X)
* You may minimize your IDE, pointing at the release branch.

## If Absent, Create Local *dist/dev* directories on your system
* On your system create the two directory structures that mirror the dist.apache.org/repos/ directories:
    * `mkdir dist/dev/datasketches/`
    * `mkdir dist/release/datasketches/`

## Checkout the *dist/dev* directory now 
* Open a terminal in the dist/dev/datasketches directory and do a checkout:
    * `svn co https://dist.apache.org/repos/dist/dev/datasketches/ .`      #Note the DOT
    * `svn status`    # make sure it is clean: does not list any (?) or (!) files

## Create the Candidate Apache Release Distribution on *dist/dev*
### Create Primary Zip Files & Signatures/Hashes
* You will need the following arguments:
  * Absolute path of target project.basedir on your system
  * Project.artifactId : datasketches-\<component> where component is e.g., java, pig, hive,...
  * GitHub Tag: A.B.0-RC1 (or RCn)
  * Have your GPG passphrase handy -- you may have only a few seconds to enter it!
* Start a new terminal in the above dist/dev/datasketches/scripts directory on your system:
  * Confirm *gpg-agent* is running:
      * `eval $(gpg-agent --daemon)`
          * if it is not running it will start it
          * if it is already running you will see something like:
          * `gpg-agent: a gpg-agent is already running - not starting a new one`  
  * Run something like (you need to edit this):
    * `./bashDeployToDist.sh /Users/\<name\>/dev/git/Apache/datasketches-\<component\> datasketches-\<component\> A.B.0-RC1`
    * Follow the instructions.
    * NOTE: if you get the error "gpg: signing failed: No pinentry":
      * open .gnupg/gpg-agent.conf
      * change to: pinentry-program _/usr/local/bin/pinentry-tty_
      * reload the gpg agent in the terminal: `gpg-connect-agent reloadagent /bye`
      * restart the _./bashDeployToDist_ script
    * Close the terminal

### Check Primary Zip Files & Signatures/Hashes
* Check this web URL ~ *https://dist.apache.org/repos/dist/dev/datasketches/\<component\>/A.B.0-RC1/*
    * There should be 3 files: \*-src.zip, \*-src.zip.asc, \*-src.zip.sha512
    * Copy the URL for later.

## Java Only: Push Jars to Nexus (Maven Central) Staging
* Return to original terminal at the project.basedir, still in the A.B.X branch.
* If starting new terminal make sure GPG is running:
  * Confirm *gpg-agent* is running:
      * `eval $(gpg-agent --daemon)`
          * if it is not running it will start it
          * if it is already running you will see something like:
          * `gpg-agent: a gpg-agent is already running - not starting a new one`  
* `git status` # make sure you are still on the release branch: A.B.X

### TRIAL-RUN:
* **Have your GPG passphrase handy -- you may have only a few seconds to enter it!**
* `mvn clean install -Pnexus-jars -DskipTests=true`
    * Check target/ that jars & pom have .asc signatures

### DEPLOY
* **Have your GPG passphrase handy -- you may have only a few seconds to enter it, but it may be automatic!**
* `mvn clean deploy -Pnexus-jars -DskipTests=true`

### DEPLOY-CHECK
* Login to [repository.apache.org](https://repository.apache.org/) / Staging Repositories for orgapachedatasketches-XXXX
* Click __Content__ and search to the end.  Each jar & pom should have .asc, .md5, .sha1 signatures

### CLOSE (Very Important)
* [CLOSE] the Staging Repository with a comment: "\<component\> A.B.0"

### CHECK CLOSE
* Confirm its existance under Repositories/Staging web-site URL (in the summary window)
* Grab its URL while there. You will need it for the Vote Letter.

### CHECK Local Maven Repo
* Check your local Maven repository
    * _~/.m2/repository/org/apache/datasketches/datasketches-\<component\>/A.B.0/_ 
    * It should have 5 new jars and a .pom file each with .asc, .md5, and .sha1 signatures

### Create Copy of External Artifact Distributions
#### JAVA ONLY
* Place copies of the artifact jars deployed to Nexus under a "maven" directory.  For example see <https://dist.apache.org/repos/dist/dev/datasketches/memory/1.3.0-RC1/>
* Note that the `jar` files with their `asc`, `md5` and `sha1` signature are all together in the .md2 archive 
* Add a `maven` directory under the `dist/dev/datasketches/\<component\>/A.B.0/`
* Bulk copy the `jar, asc, md5` and `sha1` files into the `maven` directory.
* `svn status` # check to see if it is ready to add
* `svn add . --force`
* `svn ci -m "add nexus jars to dist/dev/datasketches"`

#### NON-JAVA
* For external artifacts such as Python or Docker the subdirectory name should be relevant to the type.
* These must be signed with GPG (.asc) and SHA512 (.sha512)

### Prepare & Send \[VOTE] Letter to dev@

* See VoteTemplates directory for a recent example
* If vote is not successful, fix the problem and repeat above steps.
* After a successful vote return to **this point** and continue ...

### Prepare & Send \[VOTE-RESULT] Letter to dev@

* See VoteTemplates directory for a recent example
* Declare that the vote is closed.
* Summarize vote results

## If Successful, Finalize the Release

### Copy files from *dist/dev* to *dist/release*
* In local *dist/__dev__/datasketches/*
    * Open Terminal #1
        * Confirm you are in the `/dev/` directory: `pwd`
        * Perform SVN Checkout:
            * `svn co https://dist.apache.org/repos/dist/dev/datasketches/ .`  #note dot at end
            * `svn status` #make sure checkout is clean: does not list any (?) or (!) files
* In local *dist/__release__/datasketches/*
    * Open Terminal #2
        * Confirm you are in the `/release/` directory: `pwd`
        * Perform SVN Checkout:
            * `svn co https://dist.apache.org/repos/dist/release/datasketches/ .` #note dot at end
            * `svn status` #make sure checkout is clean: does not list any (?) or (!) files
        * Create new version directory under appropriate component directory:
            * `mkdir -p \<component\>/A.B.0`
    * Using local file system copy files 
        * From  ... /dist/dev/datasketches/\<component\>/version-RCnn/*
        * To    ... /dist/release/datasketches/\<component\>/version (no RCnn)/*
        * Make sure to move External Artifact Distributions *dist/dev* to *dist/release*
    * Using Terminal #2 at ... /dist/release/datasketches directory:
        * `svn add . --force`
        * `svn ci -m "Release A.B.0"`
        * Remove the prior release...
        * `svn remove \<component\>/X.Y.0`
        * `svn ci -m "Remove Prior release"`
        * `svn status` # should be empty
    * Using local file system
        * Delete the prior X.Y.0 directory if necessary.

### Java Only: Release Jars on Nexus Staging
* On Nexus [repository.apache.org](https://repository.apache.org/) click on Staging Repositories
* Select "orgapachedatasketches-XXXX" (If more than one make sure you select the right one!)
* At the top of the window, select "Release"
* Confirm that the attributes have moved to the "Releases" repository under "Repositories"
  * Browse to _Releases/org/apache/datasketches/..._

#### Java Only: Drop any previous Release Candidates that were not used.

### Java Only: Drop any previous Release Candidates that were not used.
* On Nexus [repository.apache.org](https://repository.apache.org/) click on Staging Repositories
* Select "orgapachedatasketches-XXXX" (If more than one make sure you select the right one!)
* At the top of the window, select "Drop"

#### If necessary, update branch _master_ from branch _A.B.X_

If you have gone through more than one Release Candidate, you may have changes that need to be reflected in the master. Use the **git cherry-pick** command for this.

### Finalize Release Documentation

#### Update Apache Reporter

* Because of the commit to the `dist/release` branch, you should get an automated email requesting you to update the Apache DataBase about the releaase. The email should point you to the [Apache Committee Report Helper](https://reporter.apache.org/addrelease.html?datasketches). You can choose to go there directly without waiting for the notice, there is only one box to fillout.
* Update the full name of the component release. For example: `Apache datasketches-memory-1.3.0`

#### Create & Document Release Tag on GitHub

* Open your IDE and switch to the recently created Release Branch A.B.X
* Find the recently created A.B.0-RCn tag in that branch
* At that same GitHub ID hash, create a new tag A.B.0 (without the RCn).
* From the Command Line: Push the new tag to origin:
  * `git push origin --tags`
* On the GitHub component site document the release

#### Update Website Downloads.md "Latest Source Zip Files" Table

* This script assumes that the remote _.../dist/release/datasketches/..._ directories are up-to-date with no old releases.
* Start a new terminal in the _../dist/dev/datasketches/scripts_ directory on your system:
* Make sure your local website directory is pointing to master and up-to-date.
* Run the following with the argument specifying the location of your local website directory:
  * `./createDownloadsInclude.sh /Users/\<name\>/ ... /datasketches-website`
* When this is done, be sure to commit the changes to the website.

### Update Website Documentation (if new functionality)

* ASF requests that you wait 24 hours to publish Announce letter to allow the propagation to mirrors.
* Use recent template
* Summarize vote results

### Update These Instructions

* If you have updated this file or any of the scripts, please update this file on the [website](https://datasketches.apache.org/docs/Community/ReleaseProcessForJavaComponents.html) and dist/dev/datasketches for the scripts.
