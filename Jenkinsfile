pipeline {
  agent any
  stages {
    stage('Preprocessing') {
      steps {
        sh '''dvc repro raw_dataset_creation

'''
        sh 'dvc repro preprocess'
      }
    }

    stage('Split') {
      steps {
        sh 'dvc repro split_data'
      }
    }

    stage('Optimize Hyperparametre') {
      steps {
        sh 'dvc repro   optimize'
      }
    }

    stage('Train/Test') {
      steps {
        sh 'dvc repro model_train'
      }
    }

    stage('Log Model') {
      steps {
        sh 'dvc repro log_production_model'
      }
    }

  }
}