pipeline {
  agent {
    kubernetes {
      defaultContainer "python"
      yamlFile "builder.yaml"
    }
  }

  environment {
    AWS_ACCESS_KEY_ID     = credentials("aws_access_key_id")
    AWS_SECRET_ACCESS_KEY = credentials("aws_secret_access_key")
    AWS_DEFAULT_REGION    = "eu-central-1"
  }

  stages {
    stage("Prepare") {
      steps {
        sh "apk add git make"

        // install generatore
        git url: "git@github.com:picofish/generatore.git", credentialsId: "9529a133-68e2-4534-8484-904ba30530de"

        sh "pip3 install -r requirements.txt"
        sh "make"
        sh "make install"

        // cleanup
        sh "ls -A1 | xargs rm -rf"
      }
    }

    stage("Build") {
      steps {
        git url: "git@github.com:picofish/cata.dev.git", credentialsId: "9529a133-68e2-4534-8484-904ba30530de"

        sh "generatore --build"
      }
    }

    stage("Deploy") {
      steps {
        sh "pip3 install awscli"

        sh """
        export AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}
        export AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}
        export AWS_DEFAULT_REGION=${env.AWS_DEFAULT_REGION}
        """

        sh "aws s3 sync output/. s3://cata.dev --delete"
      }
    }
  }
}
