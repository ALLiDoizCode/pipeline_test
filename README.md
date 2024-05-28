[![GitHub stars](https://img.shields.io/github/stars/ALLiDoizCode/pipeline_test)](https://github.com/ALLiDoizCode/pipeline_test/stargazers)
![Last Commit](https://img.shields.io/github/last-commit/ALLiDoizCode/pipeline_test)
![Pull Requests](https://img.shields.io/github/issues-pr-raw/ALLiDoizCode/pipeline_test)

---

<div align="center">

# 🚀 Elevate Your Development.

## "CI/CD Solution For Building Dapps on The Internet Computer Protocol"

</div>

---


# pipeline_test

This Repo is a working example of a ci/cd pipeline for Dapps using motoko built on the Internet Computer

## Convert your identity into a base64

```bash
base64 -i identity.pem
```

### Add your base64 Identity to your repository secrets and name the secrets for each relevant env 
For example DEVELOPEMNT, STAGING and PROD would be the name of your repository secrets and the workflow would access them using secrets.DEVELOPEMNT, secrets.STAGING and secrets.PROD

# Continuous Integration with GitHub Actions

This repository uses GitHub Actions to automate testing and building of the project. The CI workflow is triggered on pull requests to the `main`, `master`, and `development` branches.

## Workflow Configuration

The GitHub Actions workflow is defined in the `.github/workflows/ci.yml` file. Below is an explanation of the workflow configuration:

### Trigger Events

The workflow is triggered on `pull_request` events targeting the following branches:
- `main`
- `master`
- `development`

### Jobs

The workflow defines a single job named `test` that uses a matrix strategy to run tests in parallel with different versions of Node.js, MOPS, and the Motoko compiler.

### Job Strategy

The job strategy specifies the following:
- **Max Parallel**: 2 (Runs a maximum of 2 jobs in parallel)
- **Matrix**:
  - `moc-version`: [latest]
  - `mops-version`: [ic-mops@latest]
  - `node-version`: [20]

### Job Steps

The job runs on an `ubuntu-latest` runner and includes the following steps:

1. **Checkout Repository**: 
   ```yaml
   - uses: actions/checkout@v4
    ```

2. **Setup Node.js**: 
    ```yaml
    - uses: actions/setup-node@v4
        with:
            node-version: ${{ matrix.node-version }}
            cache: 'npm'
    ```

3. **Install DFX**: 
    ```yaml
    - name: Install dfx
        uses: dfinity/setup-dfx@main
    ```

4. **Confirm DFX Installation**: 
    ```yaml
    - name: Confirm successful installation
        run: dfx --version
    ```

5. **Install DFX Cache**: 
    ```yaml
    - name: Installing dfx cache
        run: dfx cache install
    ```

6. **Cache MOPS Packages**: 
    ```yaml
    - name: Cache mops packages
        uses: actions/cache@v4
            with:
                key: mops-packages-${{ hashFiles('mops.toml') }}
                restore-keys: |
                mops-packages-${{ hashFiles('mops.toml') }}
                mops-packages-
                path: |
                ~/.cache/mops
    ```

7. **Install NPM Packages for CLI (Conditionally)**: 
    ```yaml
    - name: Install npm packages
        if: ${{ matrix.mops-version == './cli' }}
        run: cd cli && npm install
    ```

8. **Build Local CLI (Conditionally)**: 
    ```yaml
    - name: Build local cli
        if: ${{ matrix.mops-version == './cli' }}
        run: cd cli && npm run prepare
    ```

9. **Install MOPS**:
    ```yaml
    - name: Install mops
        run: npm i -g ${{ matrix.mops-version }}
    ```

10. **Install MOPS Packages**:
    ```yaml
    - name: Install mops packages
        run: mops install
    ```

11. **Select Moc Version**:
    ```yaml
    - name: Select moc version
        run: mops toolchain use moc ${{ matrix.moc-version }}
    ```

12. **Run Tests**:
    ```yaml
    - name: Run tests
        run: mops test    
    ```
### Usage Instructions

1. **Add Workflow File**:
   Save the provided YAML content into a file named `ci.yml` in the `.github/workflows/` directory of your repository.

2. **Push to Repository**:
   Commit and push the workflow file to your GitHub repository.

3. **Create Pull Requests**:
   Create pull requests targeting the `main`, `master`, or `development` branches to trigger the CI workflow.

# Continuous Deployment with GitHub Actions

This repository uses GitHub Actions to automate the deployment of the project. The CD workflow is triggered on closed pull requests to the `main`, `master`, `staging`, and `development` branches.

## Workflow Configuration

The GitHub Actions workflow is defined in the `.github/workflows/cd.yml` file. Below is an explanation of the workflow configuration:

### Trigger Events

The workflow is triggered on `pull_request` events with the `closed` type, targeting the following branches:
- `main`
- `master`
- `staging`
- `development`

### Jobs

The workflow defines a single job named `check-branch` that runs if the pull request is merged. It includes steps for setting up the environment, caching packages, installing dependencies, and deploying to the appropriate network.

### Job Steps

The job runs on an `ubuntu-latest` runner and includes the following steps:

1. **Checkout Repository**: 
   ```yaml
   - uses: actions/checkout@v4
    ```

2. **Setup Node.js**: 
   ```yaml
   - uses: actions/setup-node@v4
        with:
            node-version: 20
            cache: 'npm'
   ```
3. **Cache MOPS Packages**: 
   ```yaml
   - name: Cache mops packages
        uses: actions/cache@v4
        with:
            key: mops-packages-${{ hashFiles('mops.toml') }}
            restore-keys: |
            mops-packages-${{ hashFiles('mops.toml') }}
            mops-packages-
            path: |
            ~/.cache/mops
    ```

4. **Install DFX**: 
   ```yaml
   - name: Install dfx
        uses: dfinity/setup-dfx@main
   ```

5. **Confirm DFX Installation**: 
   ```yaml
   - name: Confirm successful installation
        run: dfx --version
   ```

6. **Install DFX Cache**: 
   ```yaml
   - name: Installing dfx cache
        run: dfx cache install
   ```

7. **Install MOPS**: 
   ```yaml
   - name: Install mops
        run: npm i -g ic-mops@latest
    ```
8. **Install MOPS Packages**: 
    ```yaml
    - name: Install mops packages
        run: mops install
    ```

9. **Deploy to Development**: 
   ```yaml
   - name: Deploy to development
    if: github.base_ref == 'development'
    run: |
        echo ${{ secrets.DEVELOPMENT }} | base64 --decode > development.pem
        dfx identity import --storage-mode=plaintext development development.pem
        dfx identity use development
        dfx deploy --network development -y

   ```

10. **Deploy to Staging**: 
    ```yaml
    - name: Deploy to staging
        if: github.base_ref == 'staging'
        run: |
            if [[ "${{ github.head_ref }}" != "development" ]]; then
                echo "Only changes from the development branch can be merged into staging."
                exit 1
            else
                echo ${{ secrets.STAGING }} | base64 --decode > staging.pem
                dfx identity import --storage-mode=plaintext staging staging.pem
                dfx identity use staging
                dfx deploy --network staging -y
            fi
    ```

11. **Check Source for Master**: 
    ```yaml
    - name: Check source for master
        if: github.base_ref == 'master'
        run: |
                if [[ "${{ github.head_ref }}" != "staging" ]]; then
                    echo "Only changes from the staging branch can be merged into master."
                    exit 1
                else
                    echo ${{ secrets.PROD }} | base64 --decode > prod.pem
                    dfx identity import --storage-mode=plaintext prod prod.pem
                    dfx identity use prod
                    dfx deploy --network ic -y
                fi
    ```

### Conclusion

By following these steps, you'll set up a robust CI pipeline that automatically tests your code using the specified matrix of Node.js, MOPS, and Motoko versions aswell as a robust CD pipeline that automatically deploys your code to different environments, ensuring higher reliability and consistency in your deployment process.