---
layout: post
title:  LinkChecker - Jekyll flavor
tags:   [Jekyll, Jenkins, Linkchecker, Warnings plugin]
author: Lars Kruse
---

This parser is designed to parse the CSV output from LinkChecker
It assumes that the source of the web, that is being parsed is
available in the `_site` folder of the workspace. `_site` is the Jekyll
default, but it can be overridden.


__Example:__
Say that your hosting http://blogs.praqma.com from the master branch
Set up a job that

 * pull master
 * jekyll build
 * linkchecker -o text -Fcsv/report/linkchecker.report.csv  http://blogs.praqma.com

Let the Warnings plugin scan the generated report/linkchecker.report.csv
using this parser.

## Settings for the Warnings plugin

### Name

LinkChecker CSV (Jekyll flavor)

### Link name

Jekyll Linkchecker

### Trend report Name

Jekyll Linkchecker Trend

### Regular expression

    (.*?);(.*?);(.*?);(404 Not Found|.*?Errno 111.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?);(.*?)\n

### Mapping script

The import of `hudson.plugins.analysis.util.model.Priority` is necessary if you want to get access to the `Warning`constructor that takes five arguments - the last being the priority.

    import hudson.plugins.warnings.parser.Warning
    import hudson.plugins.analysis.util.model.Priority  

    String baseurl = "http://blogs.praqma.com"
    String jekylltarget = "_site"
    String defaultindex = "index.html"

    // As defined by LinkChecker:

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


    String modparentname = parentname.replaceAll(/\/$/,"/"+defaultindex)
    modparentname =     modparentname.replaceAll(/http:\/\/blogs.praqma.com$/,baseurl+"/"+defaultindex)
    String localfile = modparentname.replaceAll(baseurl+"/", jekylltarget+"/")

    switch ( result ) {
        case "404 Not Found":
            message = "In " + modparentname + " at line:" +line + " column:" + column + " the link "+name+ " to " + urlname + " is broken"
            return new Warning(localfile, Integer.parseInt(line), "Level:" +level, result, message, Priority.HIGH)

        case ~/.*?Errno 111.*?/:
            message = "In " + modparentname + " at line:" +line + " column:" + column + " the link "+name+ " can not connect to " + urlname
            return new Warning(localfile, Integer.parseInt(line), "Level:" +level, result, message, Priority.NORMAL)

        default:
            break
    }

    switch ( warningstring ) {

        case ~/HTTP 301.*?/:
            message = "In " + modparentname + " at line:" +line + " column:" + column + " the link "+name+ " to " + urlname + " is permanently     redirected"
            return new Warning(localfile, Integer.parseInt(line), "Level:" +level, "Permanent redirect", message, Priority.LOW)

        default:
            break
    }
