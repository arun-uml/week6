podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:6.3-jdk14
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt        
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') 
{
  node(POD_LABEL) {
    stage('Build a gradle project') {
		git 'https://github.com/arun-uml/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
		container('gradle') {			
			stage('Build a gradle project') {
			  sh '''
			  pwd
			  cd Chapter08/sample1
			  chmod +x gradlew
			  ./gradlew build
			  mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
			  '''
			}
		}
	}
	stage('Build Java Image') {
		if (env.BRANCH_NAME != 'playground')
		{
			echo "This ${env.BRANCH_NAME} branch"
			container('kaniko') {
				stage('Build a Java Image Kaniko') {
					calc_file = 'main-calculator:1.0'
					if (env.BRANCH_NAME == 'feature')
						calc_file = 'feature-calculator:1.0'
							
					sh '''
					echo 'FROM openjdk:8-jre' > Dockerfile
					echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
					echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
					mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
					/kaniko/executor --context `pwd` --destination arunuml/''' + calc_file
				}
			}
		}
	  
	}
	
	stage("Unit test") {
	  echo "I am the ${env.BRANCH_NAME} branch"
	  if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'feature') {
		echo "Running Unit test on ${env.BRANCH_NAME} branch"	  
		try {		  
		  sh '''
		  pwd
		  cd Chapter08/sample1
		  chmod +x gradlew
		  ./gradlew test
		  '''
		} 
		catch (Exception E) {
		  echo 'Failure detected'
		}				
	  }
	}
		
	stage("Code coverage") {
	  echo "I am the ${env.BRANCH_NAME} branch"
	  if (env.BRANCH_NAME == 'main') {
		echo "Running Code coverage on ${env.BRANCH_NAME} branch"	  
		try {		  
		  sh '''
		  pwd
		  cd Chapter08/sample1
		  chmod +x gradlew
		  ./gradlew jacocoTestCoverageVerification
		  '''
		} 
		catch (Exception E) {
		  echo 'Failure detected'
		}				
	  }
	}
		
	stage("Code StyleCheck") {
	  echo "I am the ${env.BRANCH_NAME} branch"
	  if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'feature') {	
		echo "Running Code Style-check on ${env.BRANCH_NAME} branch"	  
		try {
		  sh '''
		  pwd
		  cd Chapter08/sample1
		  chmod +x gradlew
		  ./gradlew checkstyleMain
		  '''
		} 
		catch (Exception E) {
		  echo 'Failure detected'
		}				
	  }
	}	
  }
}

