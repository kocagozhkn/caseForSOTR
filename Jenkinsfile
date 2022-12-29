podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: maven
        image: maven:3.8.1-jdk-8
        command:
        - sleep
        args:
        - 99d
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
    node(POD_LABEL) {
        environment {
        DATE = new Date().format('yy.M')
        TAG = "${DATE}.${BUILD_NUMBER}"
    }
   stage('Get a Maven project') {
            git url: 'https://github.com/kocagozhkn/caseForSOTR.git', branch: 'main'
            container('maven') {
                stage('Build a Maven project') {
                    sh '''
           mvn clean package
          '''
                }
            }
        }

        stage('Build Java Image') {
            container('kaniko') {
                stage('Build a Maven project') {
                    sh '''
                    
            /kaniko/executor --context `pwd` --destination=kocagoz/hello-case:${BUILD_NUMBER}
          '''
                }
            }
        }
        stage('Update GIT') {
            git url: 'https://github.com/kocagozhkn/caseForSOTR.git', branch: 'main'
            catchError(buildResult:'SUCCESS', stageResult:'FAILURE') {
                withCredentials([usernamePassword(credentialsId:'github', passwordVariable:'GIT_PASSWORD', usernameVariable:'GIT_USERNAME')]) {
                    sh "git config user.email kocagoz@gmail.com"
                    sh "git config user.name Hakan"
                    
                    sh "cat ./k8s-deployment/deployment.yaml"
                    sh "sed -i.back '/image:/s/:[0-9].*/:${env.BUILD_NUMBER}/g' ./k8s-deployment/deployment.yaml"
                    sh "cat ./k8s-deployment/deployment.yaml"
                    sh "git add ."
                    sh "git commit -m 'Done by Jenkins Job'"
                    sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/caseForSOTR.git"
                }  
            }
        }
    }
}
