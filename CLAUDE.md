# Cerebras Wafer-Scale-Engine Clusters Systems for Artificial Intelligence

## Use local cerebras-pytorch in gigaGPT

gigaGPT currently depends on Python package
    "cerebras_pytorch==2.5.0; sys_platform == 'linux'  and platform_machine == 'x86_64'",
But I can only run gigaGPT in Mac OSX on ARM64, so I unpacked the Python package wheel of the above platform in ../cerebras-pytorch-src folder,

My goal is to change the dependency on cerebras_pytorch package in gigaGPT to the local copy in ../cerebras-pytorch-src so gigaGPT could be run on Mac OSX on ARM64.

Not sure how to achieve this, potential tasks are:

- Build cerebras_pytorch package in ../cerebras-pytorch-src, publish to local package repository if needed and feasible;
- Change the dependency on cerebras_pytorch package in gigaGPT to the local copy in ../cerebras-pytorch-src;
- Run gigaGPT

Save your thinking process and summary of change, timing of your work in a Markdown file.
