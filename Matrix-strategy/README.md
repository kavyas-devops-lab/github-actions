# GitHub Stratergy

GitHub Actions matrix is a powerful feature that allows you to run jobs across multiple configurations simultaneously.

# Basic Matrix Concept

A matrix strategy creates multiple job runs from a single job definition by varying environment variables, operating systems, language versions, or any parameter you define

.github/workflows/01-basic-matrix-startergy.yaml
```
name: Basic Matrix Example

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14, 16, 18, 20]
    
    steps:
    - uses: actions/checkout@v4
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
    - name: Install dependencies
      run: npm install
    - name: Run tests
      run: npm test
```
<img width="1875" height="712" alt="image" src="https://github.com/user-attachments/assets/55581785-d2f0-44a3-8162-e37285da5bd7" />

This basic example creates 4 parallel jobs, each testing against a different Node.js version.

# Multi-Dimensional Matrix
.github/workflows/multi-dimensional-matrix.yaml
```
name: Multi-Dimensional Matrix

on: [push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16, 18, 20]
        include:
          # Add specific combinations
          - os: ubuntu-latest
            node-version: 14
            experimental: true
        exclude:
          # Remove specific combinations
          - os: macos-latest
            node-version: 16
    
    steps:
    - uses: actions/checkout@v4
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
    - name: Install dependencies
      run: npm install
    - name: Run tests
      run: npm test
    - name: Extra step for experimental builds
      if: matrix.experimental
      run: npm run test:experimental
```
<img width="1898" height="720" alt="image" src="https://github.com/user-attachments/assets/d59e594a-e2c5-4cb9-9957-494c859bdc49" />

# Advanced Matrix Features
.github/workflows/03-advanced-matrix-features.yaml
Failure Handling and Concurrency Control
```
name: Advanced Matrix Configuration

on: [push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16, 18, 20]
        include:
          # Add specific combinations with extra properties
          - os: ubuntu-latest
            node-version: 14
            experimental: true
            label: "legacy"
          - os: ubuntu-latest
            node-version: 21
            experimental: true
            label: "cutting-edge"
        exclude:
          # Remove specific combinations
          - os: macos-latest
            node-version: 16
      
      # Control failure behavior
      fail-fast: false  # Don't cancel other jobs if one fails
      max-parallel: 3   # Limit concurrent job
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
    
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ matrix.node-version }}-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-${{ matrix.node-version }}-
    
    - name: Install dependencies
      run: npm install
    
    - name: Run tests
      run: npm test
      continue-on-error: ${{ matrix.experimental }}
    
    - name: Extra step for experimental builds
      if: matrix.experimental
      run: |
        echo "Running experimental tests for ${{ matrix.label }}"
        npm run test:experimental || true
    
    - name: Platform-specific commands
      run: |
        if [ "${{ matrix.os }}" = "windows-latest" ]; then
          echo "Windows-specific setup"
        elif [ "${{ matrix.os }}" = "macos-latest" ]; then
          echo "macOS-specific setup"
        else
          echo "Linux-specific setup"
        fi
      shell: bash
```

<img width="1886" height="708" alt="image" src="https://github.com/user-attachments/assets/dbfebd1a-7d69-4711-add7-d73ddbb96250" />


