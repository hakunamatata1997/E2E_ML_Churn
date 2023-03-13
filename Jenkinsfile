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
        kubeconfig(serverUrl: 'http://172.27.35.85:6443', credentialsId: 'kubesec', caCertificate: '-----BEGIN CERTIFICATE----- MIIC/jCCAeagAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl cm5ldGVzMB4XDTIzMDIxMzEwMTgzMloXDTMzMDIxMDEwMTgzMlowFTETMBEGA1UE AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM+e SjhshmzHAj67vXScrD2sdSkHGJOqWN4S0IiUIYpjDWGBXAuaHGyi008Q0hwNs4Op QpxO/oQNc6sxwsFMXZptVgBsrl9a/4BiS10xCmw2xfBUh9sA+9FRuqsy6YSv7FXF JBynIpC1H0gn9/sZ8i1jMqHRbLmXDmR7l7I3GWPCAFklMv6TE3txMMtvOERAkHGO vAMPq4og9EbnqIG7tWOrLScyxPd23F32IkEdJUREgQTsFJuTDcQqF9ink6N01OIP 5iim9dQbDmcVkFv2f4gNUQpu+ZwBpEeYbupMmsPng7Rw8mQgVgN5dyX6aoYbermh L4IJr1eu86x1jpR0TLcCAwEAAaNZMFcwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB /wQFMAMBAf8wHQYDVR0OBBYEFAZvVV2er0urMzRQHXnAEogcZSEZMBUGA1UdEQQO MAyCCmt1YmVybmV0ZXMwDQYJKoZIhvcNAQELBQADggEBAG8uV+ai06GqZcewxLJ8 pNldv4UDNvBnnTpenSOefFffYZCafPPLpe4mpFjyruT4SNKDF86lYPehKutKf7K+ YcV+xXZvMBiXMX7pt7wLgmrGKLnDjYf8QISHfbKD6f0IDRcG0OirEYuHkdQ4XpSF hfsrhARvlk/MI5XlemjgK4tZk0VjZGyjBS5XBZ0xzV+7qHH0M6IEzSrFArvgCb/W VgIw/UNglsfs30rglFbUNaOgRtowyFeeVN6fJFSIiKq9DUvxmWhF4T2AVfFEgzMT qHyq9LJlg1T5jYQCAgL1KAMVb0q3TxmovcOA1UdyHZCng07l1Vc7z2nV+0UMjKDk tgI= -----END CERTIFICATE-----')
        sh 'kubectl apply -f ./deployment/deployment.yaml --context kubernetes-admin@kubernetes'
        sh 'kubectl apply -f ./deployment/service.yaml --context kubernetes-admin@kubernetes'
      }
    }

    stage('End Points') {
      steps {
        sh 'echo "http://172.27.35.85:3000"'
        sh 'kubectl apply -f ./deployment/service.yaml'
      }
    }

    stage('Data Report') {
      steps {
        sh 'python3 ./reports/monitor.py'
      }
    }

  }
}