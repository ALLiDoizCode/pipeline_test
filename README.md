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

2. **Setup Node.js**: 
    ```yaml
    - uses: actions/setup-node@v4
        with:
            node-version: ${{ matrix.node-version }}
            cache: 'npm'

3. **Install DFX**: 
    ```yaml
    - name: Install dfx
        uses: dfinity/setup-dfx@main

4. **Confirm DFX Installation**: 
    ```yaml
    - name: Confirm successful installation
        run: dfx --version

5. **Install DFX Cache**: 
    ```yaml
    - name: Installing dfx cache
        run: dfx cache install

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


7. **Install NPM Packages for CLI (Conditionally)**: 
    ```yaml
    - name: Install npm packages
        if: ${{ matrix.mops-version == './cli' }}
        run: cd cli && npm install


8. **Build Local CLI (Conditionally)**: 
    ```yaml
    - name: Build local cli
        if: ${{ matrix.mops-version == './cli' }}
        run: cd cli && npm run prepare


9. **Install MOPS**:
    ```yaml
    - name: Install mops
        run: npm i -g ${{ matrix.mops-version }}


10. **Install MOPS Packages**:
    ```yaml
    - name: Install mops packages
        run: mops install


11. **Select Moc Version**:
    ```yaml
    - name: Select moc version
        run: mops toolchain use moc ${{ matrix.moc-version }}


12. **Run Tests**:
    ```yaml
    - name: Run tests
        run: mops test    

### Usage Instructions

1. **Add Workflow File**:
   Save the provided YAML content into a file named `ci.yml` in the `.github/workflows/` directory of your repository.

2. **Push to Repository**:
   Commit and push the workflow file to your GitHub repository.

3. **Create Pull Requests**:
   Create pull requests targeting the `main`, `master`, or `development` branches to trigger the CI workflow.

### Conclusion

By following these steps, you'll set up a robust CI pipeline that automatically tests your code using the specified matrix of Node.js, MOPS, and Motoko versions, ensuring higher code quality and reliability.

