pipeline {
  agent any
  stages {
    stage('Preprocessing') {
      parallel {
        stage('Data Pull') {
          agent any
          steps {
            sh 'sudo cp /home/k8user/Akhil/mlops/mlflow/ChurnPrediction/data/external/Churn_Prediction.csv ./data/external/'
            sh '/home/k8user/anaconda3/bin/dvc repro raw_dataset_creation'
          }
        }

        stage('Dependencies') {
          steps {
            sh 'python3 -m pip install -r requirements.txt'
            sh 'python3 -m pip install joblib'
          }
        }

        stage('Preprocess') {
          steps {
            sh 'sudo cp /home/k8user/Akhil/mlops/mlflow/ChurnPrediction/data/external/Churn_Prediction.csv ./data/external/'
            sh '/home/k8user/anaconda3/bin/dvc repro preprocess'
          }
        }

      }
    }

    stage('Split') {
      steps {
        sh '/home/k8user/anaconda3/bin/dvc repro split_data'
      }
    }

    stage('Optimize Hyperparametre') {
      steps {
        sh '/home/k8user/anaconda3/bin/dvc repro optimize'
      }
    }

    stage('Train/Test') {
      steps {
        sh '/home/k8user/anaconda3/bin/dvc repro model_train'
      }
    }

    stage('Register Model') {
      steps {
        sh '/home/k8user/anaconda3/bin/dvc repro log_production_model'
      }
    }

    stage('Build Image') {
      steps {
        sh 'sudo DOCKER_BUILDKIT=1 docker build -t churn:latest . --build-arg http_proxy=http://172.30.10.43:3128 --build-arg https_proxy=http://172.30.10.43:3128'
      }
    }

    stage('Push Image') {
      steps {
        sh 'sudo -S docker tag churn:latest theakhilb/mlflow_churn:latest'
        sh 'sudo -S docker push theakhilb/mlflow_churn:latest'
      }
    }

    stage('Deploy in Kubernetes') {
      steps {
        withKubeConfig(credentialsId: 'kube', serverUrl: 'https://172.27.35.85:6443') {
          sh 'kubectl apply -f ./deployment/deployment.yaml'
          sh 'kubectl apply -f ./deployment/service.yaml'
        }

      }
    }

    stage('End Points') {
      parallel {
        stage('Monitor Cluster') {
          steps {
            echo 'Check the Cluster and Deployment health at http://172.27.35.85:3000'
          }
        }

        stage('Monitor Data Drift') {
          steps {
            sh '/home/k8user/anaconda3/bin/dvc repro monitor_model'
          }
        }

      }
    }

    stage('Data Report') {
      steps {
        sh 'sudo -S docker build -t churn_monitor:latest ./reports/ --build-arg http_proxy=http://172.30.10.43:3128 --build-arg https_proxy=http://172.30.10.43:3128'
        sh 'sudo -S docker start churn_monitor'
        sh 'sudo -S docker cp ./reports/templates/data_and_target_drift_dashboard.html churn_monitor:/root/ChurnPrediction/templates/'
        echo 'Check Data Drift at http://172.27.35.85:3600/monitor_data'
      }
    }

  }
}