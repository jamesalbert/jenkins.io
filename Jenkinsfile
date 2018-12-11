import groovy.text.SimpleTemplateEngine


adocTemplate = '''
---
layout: project
title: "Plugins"
section: projects
tags:
- plugins
---

== Plugins

<% plugins.each { %>
=== ${it.name}
${getDescription.call(it.full_name)}
<% } %>
'''

def renderTemplate(binding) {
  def engine = new SimpleTemplateEngine()
  def template = engine.createTemplate(adocTemplate).make(binding)
  def rendered = template.toString()
  engine = null
  template = null
  return rendered
}

def getPlugins() {
    def repoURL = "https://api.github.com/orgs/jenkinsci/repos"
    def cmd = "curl -sS $repoURL | jq -r '[.[] | select(.name|test(\"-plugin\$\"))]'"
    def json = sh(returnStdout: true, script: cmd)
    return readJSON(text: json)
}

@NonCPS
def getDescription() {
    return { String plugin ->
        def sout = new StringBuilder(), serr = new StringBuilder()
        def pomURL = "https://raw.githubusercontent.com/${plugin}/master/pom.xml"
        def proc = "curl -sS ${pomURL}".execute()
        proc.consumeProcessOutput(sout, serr)
        proc.waitForOrKill(1000)
        def pom = new XmlParser().parseText(sout.toString())
        return pom['description'].text()
    }
}

pipeline {
    agent any;

    triggers {
        upstream 'Infra/jenkins.io'
    }

    stages {
        stage ('Build Plugins Asciidoc') {
            steps {
                echo 'Building asciidoc...'
                script {
                    echo renderTemplate([
                        plugins: getPlugins(),
                        getDescription: getDescription()
                    ])
                }
                echo '...asciidoc built'
            }
        }
    }
}