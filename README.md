# pipeline_test

This Repo is a working example of a ci/cd pipeline for Dapps using motoko built on the Internet Computer

## Convert your identity into a base64

```bash
base64 -i identity.pem
```

### Add your base64 Identity to your repository secrets and name the secrets for each relevant env 
For example DEVELOPEMNT, STAGING and PROD would be the name of your repository secrets and the workflow would access them using secrets.DEVELOPEMNT, secrets.STAGING and secrets.PROD
