# Quick Start Guide for LLM Trading Agent
## Prerequisite
The entire project is tested only in following environment
- Operating System: Ubuntu 22.04.03 LTS
- Python: Version 3.10.12

## Environment Setup
```sh
# Create a virtual environment for Python packages
python -m venv .venv
source .venv/bin/activate

# Install required Python packages
pip3 install -r requirements.txt
```

## Configuration Files
- You need an NVIDIA_API_KEY in your .env file to use the NIM API.
- Generate your API key at [Try NVIDIA NIM APIs](https://build.nvidia.com/explore/discover).
- Example of .env content:

```sh
NVIDIA_API_KEY=nvapi-{...}
```

## Running the Agent workflow
To start the recursive Agent workflow, use the following command:
```sh
python -m analyst_group
```
If you desire a fresh start without using the cache.db, simply delete it or comment out the related code in `analyst_group/__init__.py`

### Human Interactions
There are three stages where human interaction is required in Agent workflow
1. If debugging loop exceeds `MAX_DEBUGGING_COUNT`, defined in `coder.py`, human supervisor should run `python -m backtesting --qa` to figure out the error and determine whether to fix it by human or give Agent another `MAX_DEBUGGING_COUNT` chances for debugging
2. When error occurred during parameter tuning, human supervision will take charge on the first error. Human supervisor should run `python -m backtesting --opt` to check the error and fix related code. The possible errors in tuning stage could be
    1. LLM generated parameters don't exist in `MyStrategy` class
    2. Runtime error wasn't caught in QA stage, since the problematic code under if statements never ran in QA stage
3. After every round of entire Agent workflow, human supervisor should check if the annual return is satisfactory enough to terminate. For another, human interaction here is to provide more human control of the recursive workflow.
    1. Entering 'y' will terminate the workflow immediately
    2. Any other human response will continue the next around of Agent workflow


## Backtesting Module
`backtesting` module is not only the one executed by Agent, it is also a good start point to take a close look at the strategies previously generated by the Agent. It provides three runnable options as below. 
- `python -m backtesting --qa` will simply trigger the backtesting with given definition of parameter in `MyStrategy` class
- `python -m backtesting --opt` will output the best trading result by tuning and optimizing the parameters defined in `params.py`
- `python -m backtesting --avg` will show the average trading performance among other tech giants


# Additional Information
## Strategy Report 
After each round of strategy analysis, a report markdown will be generated in `report` folder, so that human can have a clear summary of
- strategy description
- strategy code
- parameter permutations used for tuning
- best backtesting result with corresponding values of parameters used
- analysis of entire strategy and its output

## Reproduction of LLM response
Since the `end_date` variable used in `get_stock_data` function in `backtest.py` is set to current date, the backtesting result will differ every day, which means the cached LLM response from the second round of strategy implementation can't be reused.
</br>If you want a 100% reuse of cache.db regardless of the date change, fix the `end_date` variable to a desired date.

