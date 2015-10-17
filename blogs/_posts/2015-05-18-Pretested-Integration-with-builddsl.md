---
layout: post
title: Pretested integration with buildDSL
tags: [Jenkins, Pretested, buildDSL]
author: Mike Long
---

The pretested integration plugin does not currently support jobDSL.  However,
we can still utilize a [configure block](https://github.com/jenkinsci/job-dsl-plugin/wiki/The-Configure-Block)
that will generate the requirece xml to setup the plugin.  Here is an example
from Novelda:


~~~ groovy

job('xtMain_build_pretested') {
    label('windows')
  configure { project ->
      project / buildWrappers << 'org.jenkinsci.plugins.pretestedintegration.PretestedIntegrationBuildWrapper' {
        scmBridge(class: 'org.jenkinsci.plugins.pretestedintegration.scm.git.GitBridge') {
          branch master
          integrationStrategy(class: 'org.jenkinsci.plugins.pretestedintegration.scm.git.AccumulatedCommitStrategy')
          repoName xtmain
        }
        rollbackEnabled false
      }
    }
    multiscm {
      git{
        remote {
        	name('xtradaraccess')
        	url('https://repo1.novelda.no/scm/xtsw/xtradaraccess.git')
        	credentials('builduser')
          pruneBranches(true)
        }
        browser { // since 1.26
        	stash('https://repo1.novelda.no/projects/XTSW/repos/xtradaraccess/')
    	  }
        branch('master')
        relativeTargetDir('xtRadarAccess')
        //<hudson.plugins.git.extensions.impl.IgnoreNotifyCommit/>
        ignoreNotifyCommit() //coming in jobdsl 1.33
      }
      git{
        remote {
        	name('xtmain')
        	url('https://repo1.novelda.no/scm/xtsw/xtmain.git')
        	credentials('builduser')
          pruneBranches(true)

        }
        browser { // since 1.26
        	stash('https://repo1.novelda.no/projects/XTSW/repos/xtmain/')
    	  }
        branch('xtmain/pretested/**')
        relativeTargetDir('xtMain')
        //<hudson.plugins.git.extensions.impl.IgnoreNotifyCommit/>
        ignoreNotifyCommit()
      }
    }
    triggers {
        scm(''){
          ignorePostCommitHooks()
        }
    }
    steps {
        environmentVariables(GRADLE_TASK: 'publishToMavenLocal')
        batchFile("""call xtMain\\buildsrv\\setup62.bat
                  |cd xtRadarAccess\\xtRadarHAL\\Release
                  |make all""".stripMargin())
        batchFile("""cd xtMain\\buildsrv
                  |build_test_all.bat""".stripMargin())
    }
    publishers{
      //downstream('xtMain_cppcheck_main')
    }
    wrappers{
      timestamps()
    }

}

~~~
