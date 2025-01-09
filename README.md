# Robotics and Autonomous System Reference

[![Documentation Status](https://readthedocs.org/projects/robotics-and-autonomous-systems/badge/?version=latest)](https://robotics-and-autonomous-systems.readthedocs.io/en/latest/?badge=latest)

Read the web-friendly version [here](https://robotics-and-autonomous-systems.readthedocs.io).

## Purpose of this Reference

1. Serve as a quick reference for checking buzzwords, understanding topics, revising for interviews, or delving deeply into subjects.
2. Act as the first stop for finding resources on any topic, including courses, blogs, papers, video tutorials, and code.
3. Potentially be used as a resource for AI agents.

## Contributing

- Please use appropriate template from templates folder and try to cover all subheading in the template if possible.
- Looking forword to your PRs!

## Installation

To install the requirements:

```bash
git clone https://github.com/siddas27/Robotics-and-Autonomous-Systems.git 
git fetch origin --force --prune --prune-tags --depth 50 refs/heads/main:refs/remotes/origin/main 
asdf global python 3.10.15 
python -mvirtualenv $READTHEDOCS_VIRTUALENV_PATH 
python -m pip install --upgrade --no-cache-dir pip setuptools 
python -m pip install --upgrade --no-cache-dir sphinx 
python -m pip install --exists-action=w --no-cache-dir -r docs/requirements.txt 
```
Build html pages
```bash
python -m sphinx -T -b html -d _build/doctrees -D language=en . $READTHEDOCS_OUTPUT/html
```