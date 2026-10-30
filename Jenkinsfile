pipeline {
   agent any

   stages {
      stage('Pull code from github') {
         steps {
            git branch: 'master', url: 'https://github.com/premanand-dalvi/jenkins-project.git'
         }
      }

      stage('Stop Existing App') {
         steps {
            sh '''
               PID=$(pgrep -f "student-registration-backend" || true)
               if [ -n "$PID" ]; then
                  echo "Stopping PID: $PID"
                  kill -9 $PID || true
               fi
            '''
         }
      }

      stage('Build Backend') {
         steps {
            dir('backend') {
               sh 'mvn clean package -DskipTests'
            }
         }
      }

      stage('Deploy Backend') {
         steps {
            sh '''
               nohup java -jar backend/target/student-registration-backend-0.0.1-SNAPSHOT.jar > backend.log 2>&1 &
               sleep 5
            '''
         }
      }
   }
}
