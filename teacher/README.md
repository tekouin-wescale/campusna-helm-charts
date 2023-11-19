# Teacher Test Deployment

helm upgrade --cleanup-on-fail \
  --install jupyter jupyterhub/jupyterhub \
  --namespace jupyterhub \
  --create-namespace \
  --version=3.1.0	 \
  --values values.yaml
## Files Structure
```bash
├── README.md
├── deployment.yaml # Contains the deployment configuration
├── service.yaml # Contains the service configuration to expose the deployment
|── secrets.yaml # Contains the secrets configuration to be used by the deployment 
├── docker_secret.yaml # Contains the docker secret configuration to be used by the deployment  to pull image from Docker Hub
├── config.yaml # Contains the configmap configuration to be used by the deployment
└── namespace.yaml # Contains the namespace configuration to be used by the deployment
```

## Available Secrets & Configurations

Please make sure to create the following secrets and configmap before deploying the service

| Secret Name | Description |
| ----------- | ----------- |
| MYSQL_PASSWORD | User Password |
| AWS_SECRET_ACCESS_KEY | AWS Secret key |
| AWS_ACCESS_KEY_ID | AWS access key  |
| MYSQL_USER | The database user |
| MYSQL_HOST | The database host (AWS RDS)|
| AWS_MASTER_ACCOUNT_ID | Organization Managment account id|

| Config Name | Description |
| ----------- | ----------- |
| MYSQL_DATABASE | The database name |
| MOODLE_BASE_URL | The base url for the moodle service |
| TEACHER_BASE_URL | The base url for the teacher service |
| GITHUB_WORKFLOW | The github workflow name |
| MYSQL_TEACHER_DATABASE | The teacher database name |
| MYSQL_MOODLE_DATABASE | The moodle database name |
| ALIAS_PREFIX | The alias prefix for the resources |
| FILE_FOR_RESOURCES_TYPES | The file path for the resources types |
| NUKE_CONFIG_FILE_DIRECTORY | The directory path for the nuke config file |
| NUKE_OUTPUT_FILE_DIRECTORY | The directory path for the nuke output file |
| NUKE_ERROR_FILE_DIRECTORY | The directory path for the nuke error file |
| AWS_PROFILE | The AWS profile to be used by the nuke |
| AWS_REGION | The AWS region to be used by the nuke |
| AWS_OUTPUT | The AWS output to be used by the nuke |
| AWS_DEFAULT_REGION | The AWS default region to be used by the terraform test |
| AWS_DEFAULT_OUTPUT | The AWS default output to be used by the terraform test |
| AWS_DEFAULT_PROFILE | The AWS default profile to be used by the terraform test |
| AWS_SDK_LOAD_CONFIG | The AWS SDK load config to be used by the nuke |
| AWS_SDK_LOAD_CONFIG_OPTIONS | The AWS SDK load config options to be used by the nuke |

* Note: The secrets and configmap values are base64 encoded
* Docker Secret is on this format: `{{ "{\"auths\": {\"docker.io\": {\"username\": \"<username>\", \"password\": \"<password>\"}}}" | b64encode }}`

## Procedure

1. Make sure you have a running EKS cluster and kubectl configured to use it
2. Secret Creation Example:

```bash
echo -n 'admin' | base64
YWRtaW4=
```

2. Create the namespace

```bash
kubectl apply -f namespace.yaml
```

3. Create the secrets

```bash
kubectl apply -f secrets.yaml
```

4. Create the docker secret

```bash
kubectl apply -f docker_secret.yaml
```

5. Create the configmap

```bash
kubectl apply -f config.yaml
```

6. Create the deployment

```bash
kubectl apply -f deployment.yaml
```

7. Create the service

```bash
kubectl apply -f service.yaml
```
