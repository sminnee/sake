`sake` - Sapphire Make
======================

**This document is currently a speculative discussion of how we would like the sake to behave.  For information on the current `sake` tool, see [the SilverStripe documentation wiki](http://doc.silverstripe.org/doku.php?id=sake)**.

`sake` is a tool designed to a perform a number development tasks on Sapphire and SilverStripe CMS projects.  It can help with the following:

 * Creating new Sapphire projects, making use of configuration templates
 * Managing modules in your project, using both `git and `svn`
 * Executing post-deployment build scripts
 * Executing Sapphire maintenance scripts such as `dev/build` and `dev/tests/all`

Note that for the purposes of this discussion, a site made on SilverStripe CMS is just a special kind of Sapphire project.

Installing sake
---------------

`sake` is designed to be installed onto your machine by executing a bootstrap script:

	curl http://www.example.org/somehwere/sake-bootstrap | sudo php
	
**This is speculative, the installation doesn't work yet!**

Creating projects
-----------------

`sake` can create a skeletal Sapphire project in a directory on your site:

	sake create ~/Sites/myproject
	
It can automatically put your project into a `git` or `svn` repository:

	sake create ~/Sites/myproject --git
	sake create ~/Sites/myproject --git=git@github.com:me/myproject.git 
	sake create ~/Sites/myproject --svn=http://svn.example.com/myproject/trunk

It can also make use of "skeleton" definitions that give more information about how to set the site up.  In particular, they say where to download ap

	sake create ~/Sites/myproject --git --skeleton=cms_github_master --modules=cmsworkflow

Sake currently comes with a few predefined skeletons:

 * **`cms_stable`:** The latest stable release of SilverStripe CMS, with the blackcandy theme
 * **`cms_trunk`:** The development trunk of SilverStripe CMS, with the blackcandy theme
 * **`cms_github_master`:** The master branch of SilverStripe CMS from GitHub, with the blackcandy theme.

The skeleton will also specify the location of the module repository.

### What does a skeleton contain?

Essentially, a skeleton contains a number of configuration values that override the defaults given in the configuration.  The order of precedence for the configuration options is as follows:

 * Configuration from the command-line arguments
 * Configuration from the skeleton
 * Global configuration in ~/.sake

This means that you overload settings that the skeleton provides you.  For instance, here we are replacing the default `blackcandy` theme with the `artica` theme.

	sake create ~/Sites/myproject --git --skeleton=cms_github_master --modules=cmsworkflow --theme=artica

### What configuration options are available?

In addition to `--git`, `--svn`, and `--skeleton`, the following configuration options are available.  These are the options that can be overwritten by the skeletons.

 * **`--theme`**: Select a theme to use:
  * `--theme=artica`: Use the `artica` theme
  * `--theme=basic->mytheme`: Use the `basic` theme, renaming it to `mytheme` within the project.  This is useful when you're wanting to create your own theme based on a starting point.
 * **`--theme-repos`**: _The format of theme repositories is unclear._
 * **`--modules`**: Select modules to use. 
  * `--modules=cmsworkflow,ecommerce/stable`: Install the stable releases of cmsworkflow and ecommerce modules.
  * `--modules=cmsworkflow/trunk,ecommerce/0.5.2`: Install the development trunk of cmsworkflow and version 0.5.2 of ecommerce.
 * **`--module-repos`**: _The format of module repositories is unclear._
 * **`--database`**: Set a database name; defaults to `SS_(sitedirectory)`

Managing modules
----------------

`sake` can be used to add and remove modules from your site:

	cd ~/Sites/myproject
	sake modules/add ecommerce
	sake modules/add cmsworkflow/stable
	sake modules/remove forum
	
After each such command, the post-deployment build scripts will be executed.
	
Post-deployment build scripts
-----------------------------

`sake` can be used to define build and deployment actions such as post-deploy.

	sake post-deploy
	
What this will actually do is execute the `.sake/post-deploy.cmd.php` script.  Here is an example of what that might contain:

	<?php
	// Update modules as necessary
	svnModule('sapphire', 'http://svn.silverstripe.org/open/modules/sapphire/trunk');
	svnModule('cms', 'http://svn.silverstripe.org/open/modules/cms/trunk');
	svnModule('jsparty', 'http://svn.silverstripe.org/open/modules/jsparty/trunk');
	
	// Update the database
	sake('dev/build');
	
	// Update the static cache
	sake('dev/buildcache');

Sake will create an initial `post-deploy` script for you when creating a project.  It is designed to be called after checking the site code out into a new location to get the site to an executable state.  This can be hooked into a git `post-update` or `post-receive` hook.  It is also called by the module manager.
	
Executing maintenance commands
------------------------------

`sake` can be used to call any URL in your Sapphire application.  Although this is of limited usefulness for your general URLs, there are a number of useful URLs starting with `dev/`.  For example:

	sake dev           # See a list of dev/ actions
	sake dev/build     # Upgrade the database schema as necessary
	sake dev/tests/all # Run all unit tests
	
**Fun fact:** The `Director::is_cli()` will return true if you are calling a URL from sake, and can be used to tailor the input to the CLI text-based interface.