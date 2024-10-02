Running in devcontainer
    Use Python 3.12

I get an error:
```
vscode ➜ /workspaces/AutoGPT/classic $ ./run setup
Defaulting to user installation because normal site-packages is not writeable
Collecting click
  Downloading click-8.1.7-py3-none-any.whl.metadata (3.0 kB)
Downloading click-8.1.7-py3-none-any.whl (97 kB)
Installing collected packages: click
Successfully installed click-8.1.7
Traceback (most recent call last):
  File "/workspaces/AutoGPT/classic/cli.py", line 8, in <module>
    import click
ModuleNotFoundError: No module named 'click'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/workspaces/AutoGPT/classic/cli.py", line 13, in <module>
    import click
ModuleNotFoundError: No module named 'click'
```

Everything seemed fine:
```
vscode ➜ /workspaces/AutoGPT/classic $ pip --version
pip 24.2 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
vscode ➜ /workspaces/AutoGPT/classic $ pip install click
Defaulting to user installation because normal site-packages is not writeable
Requirement already satisfied: click in /home/vscode/.local/lib/python3.12/site-packages (8.1.7)
vscode ➜ /workspaces/AutoGPT/classic $ python3.12 -m pip install click  # If you're using Python 3.12
Defaulting to user installation because normal site-packages is not writeable
Requirement already satisfied: click in /home/vscode/.local/lib/python3.12/site-packages (8.1.7)
vscode ➜ /workspaces/AutoGPT/classic $ python -m site
sys.path = [
    '/workspaces/AutoGPT/classic',
    '/usr/local/lib/python312.zip',
    '/usr/local/lib/python3.12',
    '/usr/local/lib/python3.12/lib-dynload',
    '/home/vscode/.local/lib/python3.12/site-packages',
    '/usr/local/lib/python3.12/site-packages',
]
USER_BASE: '/home/vscode/.local' (exists)
USER_SITE: '/home/vscode/.local/lib/python3.12/site-packages' (exists)
ENABLE_USER_SITE: True
```


Solution that worked
```
vscode ➜ /workspaces/AutoGPT/classic $ pip uninstall click
Found existing installation: click 8.1.7
Uninstalling click-8.1.7:
  Would remove:
    /home/vscode/.local/lib/python3.12/site-packages/click-8.1.7.dist-info/*
    /home/vscode/.local/lib/python3.12/site-packages/click/*
Proceed (Y/n)? Y
  Successfully uninstalled click-8.1.7

vscode ➜ /workspaces/AutoGPT/classic $ pip install click
Defaulting to user installation because normal site-packages is not writeable
Collecting click
  Using cached click-8.1.7-py3-none-any.whl.metadata (3.0 kB)
Using cached click-8.1.7-py3-none-any.whl (97 kB)
Installing collected packages: click
Successfully installed click-8.1.7
```

Running `./run setup` installs Poetry

Create new agent
```
vscode ➜ /workspaces/AutoGPT/classic $ ./run agent create TEST_AGENT_1
🎉 New agent 'TEST_AGENT_1' created. The code for your new agent is in: agents/TEST_AGENT_1
```

Received error when starting agent:
```
vscode ➜ /workspaces/AutoGPT/classic (master) $ ./run agent start TEST_AGENT_1
⌛ Running setup for agent 'TEST_AGENT_1'...
Path /workspaces/AutoGPT/classic/agents/benchmark for agbenchmark does not exist
Removed the poetry environment at /home/vscode/.cache/pypoetry/virtualenvs/autogpt-forge-3GLPcG1L-py3.12.
Path /workspaces/AutoGPT/classic/agents/benchmark for agbenchmark does not exist
Creating virtualenv autogpt-forge-3GLPcG1L-py3.12 in /home/vscode/.cache/pypoetry/virtualenvs
Installing dependencies from lock file
Path /workspaces/AutoGPT/classic/agents/benchmark for agbenchmark does not exist
...
Path /workspaces/AutoGPT/classic/agents/benchmark for agbenchmark does not exist

Path /workspaces/AutoGPT/classic/agents/benchmark for agbenchmark does not exist
Setup completed successfully.

⌛ (Re)starting agent 'TEST_AGENT_1'...
kill: usage: kill [-s sigspec | -n signum | -sigspec] pid | jobspec ... or kill -l [sigspec]
Path /workspaces/AutoGPT/classic/agents/benchmark for agbenchmark does not exist
Traceback (most recent call last):
  File "<frozen runpy>", line 198, in _run_module_as_main
  File "<frozen runpy>", line 88, in _run_code
  File "/workspaces/AutoGPT/classic/agents/TEST_AGENT_1/forge/__main__.py", line 4, in <module>
    import uvicorn
ModuleNotFoundError: No module named 'uvicorn'
```

Install `uvicorn` but it also seems that there's needs to be benchmark

```
vscode ➜ /workspaces/AutoGPT/classic (master) $ ./run benchmark start TEST_AGENT_1
🚀 Running benchmark for 'TEST_AGENT_1' with subprocess arguments: 
vscode ➜ /workspaces/AutoGPT/classic (master) $ Path /workspaces/AutoGPT/classic/agents/benchmark for agbenchmark does not exist
Command not found: agbenchmark
```

`pip install agbenchmark`

Now I get different error:

```
vscode ➜ /workspaces/AutoGPT/classic (master) $ ./run benchmark start TEST_AGENT_1
🚀 Running benchmark for 'TEST_AGENT_1' with subprocess arguments: 
vscode ➜ /workspaces/AutoGPT/classic (master) $ Path /workspaces/AutoGPT/classic/agents/benchmark for agbenchmark does not exist
Usage: agbenchmark [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  serve
  start
  version  Print the version of the benchmark tool.
^C
```

From discord:
 in pyproject.toml 
it's a relative path so if you have your agent in agents/{your_agent_name} you should change line 12 from
`agbenchmark = { path = "../benchmark", optional = true }`
to
`agbenchmark = { path = "../../benchmark", optional = true }`

After that, benchmark still gave the same issue but without the agbenchmark error

I ran start command, it complained `primp` can't be installed since I don't have cargo and rust. To install those:
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustc --version
cargo --version
```

Then I ran start again and now it complained about primp again for not having google logging module:  
So I ran `pip install google-cloud-logging`

I tried to install `primp` with `pip wheel --no-cache-dir --use-pep517 "primp==0.6.3"` from TEST_AGENT_1 directory but it's complaining again I don't have cmake:
```
failed to execute command: No such file or directory (os error 2)
is `cmake` not installed?
```
To install `cmake`:
```
sudo apt-get update
sudo apt-get install cmake
```
