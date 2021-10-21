# ALL-Hardware's GitHub Actions for firmware remote deploy

CI/CD techniques are coming to the embedded world and recently All-Hardware service together with GitHub Actions enabled a remote access to the AlifSemiconductor [Ensemble DevKit-E7](https://alifsemi.com/products/) development board.

[GitHub Actions](https://github.com/features/actions) makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want. For more information check this [link](https://docs.github.com/en/actions).

## Setting up your workflow

[Here](https://github.com/all-hw/ci-demo) you can find demo project for AlifSemiconductor Ensemble DevKit-E7 development board with example workflow.

Now let's take a quick look at the workflow configuration file:

```yaml
name: ALL-HW ALIF CI

on:
  # Manually triggered workflow
  workflow_dispatch:
    inputs:
      # parameters description
      binary:
        description: 'Binary for the task'
        default: 'bin/HelloW.axf'
        required: true
      file:
        description: 'Input data for the task'
        default: 'test_data/uart_input.txt'
        required: true
      api_key:
        description: 'API key for the all-hw.com service'
        default: 'ba1e491b-331b-4e35-b799-f714b8505843'
        required: true
      timeout:
        description: 'Firmware task timeout'
        default: '10'
        required: true

jobs:
  build_and_run:
    runs-on: ubuntu-20.04
    steps:
    # checking out our repository
    - name: Check out repository
      uses: actions/checkout@v2
      with:
        submodules: recursive

    # using the parameters provided above we call
    # All-Hardware GitHub actions
    # to upload your firmware to the cloud hardware
    - name: Run firmware upload to All-Hardware
      id: task
      uses: all-hw/uart-task@main
      with:
        binary: ./${{ github.event.inputs.binary }}
        uart-in: ./${{ github.event.inputs.file }}
        api-key: ${{ github.event.inputs.api_key }}
        timeout: ${{ github.event.inputs.timeout }}
      timeout-minutes: 1

    - name: Getting the Status
      run: echo "${{ steps.task.outputs.status }}"

    - name: Getting the UART output
      run: echo "${{ steps.task.outputs.uart-out }}"

    - name: Getting the execution result code
      run: echo "${{ steps.task.outputs.result-code }}"
```


## Firmware requirements

Please note that currently the workflow process rely on the UART IO for data exchange.

The All-Hardware service is waiting for exiting _main_ function of your firmware to finish the workflow.
