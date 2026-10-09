# Zip Bomb Generator (Educational & Defensive Testing Tool)

A lightweight security utility designed to generate decompression archives (zip bombs) for testing the resilience of antivirus software, web upload validation pipelines, and automated file parsers.

⚠️ **Disclaimer:** This tool is intended strictly for authorized security research, educational purposes, and defensive benchmarking. Do not use this tool to disrupt infrastructure or systems without explicit permission.

---

## 📌 Features

- **Adjustable Payload Sizes:** Generate archives that expand from a few kilobytes to gigabytes or terabytes of dummy data.
- **Layered Compression:** Supports nested configurations (ZIPs inside ZIPs) to test multi-level archive handling.
- **Flat Overlap Testing:** Implements modern, non-recursive overlap techniques to test single-layer parser limits.
- **Safety Thresholds:** Includes hardcoded generation limits to prevent accidental local system exhaustion.

---

## 🛠️ How It Works

A **zip bomb** (or decompression bomb) leverages the ZIP file format's compression algorithms (typically DEFLATE). By compressing highly repetitive data (such as a long stream of zeroes), the algorithm achieves an extreme compression ratio. 

When a vulnerable antivirus scanner, file parser, or web application attempts to extract the file to inspect it, the system expands the data exponentially. This exhausts the host's **disk space, RAM, or CPU**, resulting in a Denial of Service (DoS) condition.

---

## 🚀 Getting Started

### Prerequisites

- [Python 3.8+](https://python.org)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com
   cd zip-bomb-generator
   ```

2. Run the script to generate a payload:
   ```bash
   python generator.py --output payload.zip --size 10GB
   ```

---

## 🤖 CI/CD Integration (GitHub Actions)

You can integrate this generator into your GitHub Actions pipelines to automatically test your application's file upload or parsing logic against decompression attacks before code deployment.

### Example Workflow Configuration

Create a file named `.github/workflows/security-test.yml` in your repository and add the configuration below. This workflow will automatically generate a test payload and pass it to your application's parser.

```yaml
name: Decompression Bomb Security Test

on:
  push:
    branches: [ main, dev ]
  pull_request:
    branches: [ main ]

jobs:
  test-resilience:
    runs-on: ubuntu-latest
    timeout-minutes: 5 # Safety threshold to prevent infinite decompression loops

    steps:
    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.10'

    - name: Generate Test Zip Bomb
      run: |
        python generator.py --output test_bomb.zip --size 5GB

    - name: Execute Application Parser Test
      run: |
        # Replace 'run_parser.py' with your actual application test execution command.
        # Your application code should catch the bomb, log it, and exit cleanly (exit code 0).
        python tests/run_parser.py --file test_bomb.zip

    - name: Cleanup Payload
      if: always()
      run: rm -f test_bomb.zip
```

*Note: Standard GitHub-hosted Ubuntu runners come with **14 GB of available disk space**. Ensure your test script limits extraction or aborts early to prevent running out of disk space on the runner.*

---

## 🛡️ Mitigation & Defensive Engineering

If you are developing an application that accepts file uploads or parses archives, implement the following best practices to protect your systems:

1. **Inspect Metadata Before Decompression:** Check the uncompressed size field in the ZIP header before starting the extraction process.
2. **Implement an Expansion Ratio Limit:** Abort extraction immediately if the uncompressed data exceeds a specific ratio (e.g., if the uncompressed size is >200 times the compressed size).
3. **Set Absolute Thresholds:** Enforce a strict maximum uncompressed size limit (e.g., abort if uncompressed data exceeds 500 MB).
4. **Use Streaming Parsers:** Read the archive as a stream and track the bytes written to disk in real-time. If the limit is crossed, safely kill the process.

---

## ⚖️ License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
