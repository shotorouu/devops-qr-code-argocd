# AWS credentials, run locally after writing your credentials in the api/.env file
kubectl create secret generic aws-creds --from-env-file=api/.env -n qrcode
