---
layout: post
title:  LinkChecker - Jekyll flavor
tags:   [Jekyll, Jenkins, Linkchecker, Warnings plugin]
author: Lars Kruse
---

__This blog is about how to setup link and resource verification on your website. We're using [LinkChecker](https://wummel.github.io/linkchecker/) which we've wrapped up in a docker image and we're utilizing Jenkins CI especially the [Warnings plugin](https://wiki.jenkins-ci.org/display/JENKINS/Warnings+Plugin) to do the test.__

The Warnigns plugin comes with a lot of build in  warnign parsers, but LinkChecker is not one of them. Lucikly the Warnigns plugin supports that you can write you own regular expression and a groovy script to parse it.

On your Jenkins CI web you simply go:

_Jenkins -> Manage Jenkins -> Configures System_ and then you scroll down to the _Compiler warnings parsers_ and create your own.

Our parser is designed to parse the CSV output from LinkChecker. It assumes that the source of the web, that is being parsed is available in the `_site` folder of the workspace. `_site` is the Jekyll default but it can be overridden.

Having the files available in the workspace enables the Warnings plugin to actually open the file and highlight the line where an observation has been made. However, a lot of URLs don't list the file that being browsed. Like `www.praqma.com` implicitly points to `www.praqma.com/index.html`.

In order to have the Warnings plugin work properly we have added the feature intor our parser, that implicit file names are med explicit - assuming the the default is `index.html`, but that settign can also be overridden.

##The user scenario

Our user scenario is optimized for pages served on GitHub Pages. Which implies that you are building locally using Jekyll and you source code is in a Git repository on GitHub. If your that lucky then setting it up is really easy once you've extended the Warnings plugin with our parser.

###Example
We're hosting `www.josra.org` from the master branch at GitHub so to set up that job includes:

 * Use Git plugin to pull master from https://github.com/josra/josra.github.io.git
 * In the build step simply run `jekyll build` (Jekyll is availabel on several Docker images)
 * Run `linkchecker -o text -Fcsv/report/linkchecker.report.csv  http://www.josr.org`
 * Confige the Warnings plugin to scan the generated report/linkchecker.report.csv using our parser.

 ###Locally hosted web
 If you want to test a website before actually publishing it, you can run it as part of your Jenkins job using a docker container with Jekyll.

## Creating the LinkChecker CSV (Jekyll flavor) parser

![Warnigns plugin](/images/linkchecker-jekyll-setup.png){: .img-lrg}

### Name
LinkChecker CSV (Jekyll flavor)

### Link name
Jekyll Linkchecker

### Trend report Name
Jekyll Linkchecker Trend

### Regular expression

    ^(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(\d+);(\d+);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?)\n

The CSV format of LinkChecker gives 17 output parameters in this format:

`urlname; parentname; baseref; result; warningstring; infostring; valid; url; line; column; name; dltime; dlsize; checktime; cached; level; modified`

This header is actually also printed in the output, so in order to not pick this one up as a warning we're specifically looking for digits `\d+` in match 9 and 10 (line and column).

### Mapping script

    // www.praqma.com/blogs/linkckecker-jekyll-flavor.html

    import hudson.plugins.warnings.parser.Warning
    import hudson.plugins.analysis.util.model.Priority  

    String jekylltarget = "_site"
    String defaultindex = "index.html"

    String urlname       = matcher.group(1)
    String parentname    = matcher.group(2)
    String baseref       = matcher.group(3)
    String result        = matcher.group(4)
    String warningstring = matcher.group(5)
    String infostring    = matcher.group(6)
    String valid         = matcher.group(7)
    String url           = matcher.group(8)
    String line          = matcher.group(9)
    String column        = matcher.group(10)
    String name          = matcher.group(11)
    String dltime        = matcher.group(12)
    String dlsize        = matcher.group(13)
    String checktime     = matcher.group(14)
    String cached        = matcher.group(15)
    String level         = matcher.group(16)
    String modified      = matcher.group(17)

    String modparentname = parentname

    if (modparentname ==~ /https?:\/\/[^\s\.].[^\s\/]*$/) {
      modparentname = ${modparentname}${"/"}
    }

    if (modparentname ==~ /.*\/$/) {
      modparentname = ${modparentname}${defaultindex}
    }

    String localfile = modparentname.replaceAll(/https?:\/\/[^\s\.].[^\s\/]*/,
      jekylltarget)

    message = "In " + modparentname + " the reference to " + urlname +
      " @ line " + line + ":" + column + " says: " + result + " " +
       warningstring + " " + infostring

    def prio

    switch ( valid ){
      case ~/True/ :
        prio = Priority.LOW
        break

      case ~/False/ :
        prio = Priority.NORMAL
        break
    }

    if ( result ==~ /.*?404 Not Found.*?/){
      prio = Priority.HIGH
    }

    return new Warning( localfile, Integer.parseInt( line ), "Level:" + level,
      result, message, prio)

The import of `hudson.plugins.analysis.util.model.Priority` is necessary since we want to get access to the `Warning` constructor that takes five arguments - the last being the priority.

The `modparentname`is where we make the implicit filenames explicit. `localfile` is used for mapping the files locally to the workspace of the Jenkins job.

We set the priority based on the `valid = True|False` setting and then we raise it to HIGH of we're dealing with a `404 Not Found` error.

##An example

![LinkChecker and Warnigns plugin example](/images/linkchecker-sample.png){: .img-lrg}

The picture above is from the virgin run of the [www.josra.org](http://www.josra.org) web. Boy - we have some cleaning up to do!

You can easily configure the Warnings plugin to set the Jenkins jobs build status based on the number of warnings. Just go to the advanced settings of the Warngins pluign at the individual job and set it there.

![Priorites of the Warnings plugin](/images/priorities.warnings-plugin.png){: .img-lrg}
In the setting we run with, any `HIGH` proprity warning will make the job fail. Three `NORMAL` priority warnings will mark it unstable while 6 will make it fail too. `LOW` priority findings aren't even considered.
