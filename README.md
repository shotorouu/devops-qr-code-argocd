# AWS credentials AND Bucket name, run locally after writing your credentials in the api/.env file
kubectl create secret generic aws-creds --from-env-file=api/.env -n qrcode

# An example of the api/.env file
AWS_ACCESS_KEY="your-key"
AWS_SECRET_KEY="your-secret-key"
AWS_BUCKET_NAME="bucket-name"
