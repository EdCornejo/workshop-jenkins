# Jenkins project

- Create a gradle empty project
- `cd jenkins`
- `touch Dockerfile`
- `touch plugins.txt`
- `echo $'authorize-project:1.3.0\njob-dsl:1.77\nworkflow-aggregator:2.6\ngit:4.2.2' > plugins.txt`

```
authorize-project:1.3.0
job-dsl:1.77
workflow-aggregator:2.6
git:4.2.2
```

- `echo $'FROM jenkins/jenkins:2.244\n\nCOPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt\n\nCOPY seedJob.xml /usr/share/jenkins/ref/jobs/seed-job/config.xml\n\nENV JAVA_OPTS -Djenkins.install.runSetupWizard=false' > Dockerfile`

```
FROM jenkins/jenkins:2.244

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt

COPY seedJob.xml /usr/share/jenkins/ref/jobs/seed-job/config.xml

ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false
```

- `vim build.gradle`

```
plugins {
    id 'base'
    id 'com.palantir.docker' version '0.22.1'
    id 'com.palantir.docker-run' version '0.22.1'
    id 'pl.allegro.tech.build.axion-release' version '1.10.1'
}

project.version = scmVersion.version

docker {
    name "${project.name}:${project.version}"
    files "plugins.txt", "seedJob.xml"
}

dockerRun {
    name "${project.name}"
    image "${project.name}:${project.version}"
    ports '8080:8080'
    clean true
    daemonize false
}
```

- `vim seedJob.xml`

```
<?xml version='1.1' encoding='UTF-8'?>
<project>
  <description></description>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <scm class="hudson.plugins.git.GitSCM" plugin="git@4.2.2">
    <configVersion>2</configVersion>
    <userRemoteConfigs>
      <hudson.plugins.git.UserRemoteConfig>
        <url>https://github.com/EdCornejo/workshop-jenkins</url>
      </hudson.plugins.git.UserRemoteConfig>
    </userRemoteConfigs>
    <branches>
      <hudson.plugins.git.BranchSpec>
        <name>*/master</name>
      </hudson.plugins.git.BranchSpec>
    </branches>
    <doGenerateSubmoduleConfigurations>false</doGenerateSubmoduleConfigurations>
    <submoduleCfg class="list"/>
    <extensions/>
  </scm>
  <canRoam>true</canRoam>
  <disabled>false</disabled>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <triggers/>
  <concurrentBuild>false</concurrentBuild>
  <builders>
    <javaposse.jobdsl.plugin.ExecuteDslScripts plugin="job-dsl@1.77">
      <targets>createJobs.groovy</targets>
      <usingScriptText>false</usingScriptText>
      <sandbox>false</sandbox>
      <ignoreExisting>false</ignoreExisting>
      <ignoreMissingFiles>false</ignoreMissingFiles>
      <failOnMissingPlugin>false</failOnMissingPlugin>
      <failOnSeedCollision>false</failOnSeedCollision>
      <unstableOnDeprecation>false</unstableOnDeprecation>
      <removedJobAction>IGNORE</removedJobAction>
      <removedViewAction>IGNORE</removedViewAction>
      <removedConfigFilesAction>IGNORE</removedConfigFilesAction>
      <lookupStrategy>JENKINS_ROOT</lookupStrategy>
    </javaposse.jobdsl.plugin.ExecuteDslScripts>
  </builders>
  <publishers/>
  <buildWrappers/>
</project>
```

- `./gradlew docker`
- `./gradlew dockerRun`
- Enter in `localhost:8080`
- Enter in `seed-job`
    - Configure > Look on Filesystem > DSL Scripts > add `createJobs.groovy`
    - Add Source Code Management > Git > Repositories > Set `Repository URL`
- `vim createJobs.groovy`

```
pipelineJob('pipelineJob') {
    definition {
        cps {
            script(readFileFromWorkspace('pipelineJob.groovy'))
            sandbox()
        }
    }
}
```

- `vim pipelineJob.groovy`
```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Build'
            }
        }
        stage('Test'){
            steps {
                echo 'Test'
            }
        }
    }
}
```

- `git add .`
- `git commit -m "add tasks"`
- `git push origin master`
- `./gradlew docker`
- `./gradlew dockerRun`
- Enter in the localhost:8080 and review if the seed-job has SUCCESS status
- Review pipeline job

# References
- [$'' strings use ANSI C Quoting](https://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html)