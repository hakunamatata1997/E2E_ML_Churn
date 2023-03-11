pipeline {
  agent any
  stages {
    stage('Preprocessing') {
      parallel {
        stage('Data Pull') {
          agent any
          steps {
            sh '''sudo cp /home/k8user/Akhil/mlops/mlflow/ChurnPrediction/data/external/Churn_Prediction.csv ./data/external/
'''
            sh '/home/k8user/anaconda3/bin/dvc repro raw_dataset_creation'
          }
        }

        stage('Preprocess') {
          steps {
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
        sh '/home/k8user/anaconda3/bin/dvc repro   optimize'
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

  }
}