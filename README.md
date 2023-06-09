# `botsh`

[![PyPI version](https://badge.fury.io/py/botsh.svg)](https://badge.fury.io/py/botsh)

`botsh` attaches a chat agent to a Docker container running Ubuntu.

This effectively gives the agent access to the entire [APT](https://wiki.debian.org/Apt)
universe, while limiting its blast radius to the current directory (which gets mounted into the container).

## Demo

This demo uses the prompt `create a sequence of 100 200x200 images which each contain a different number, and turn them into a 30fps video with ffmpeg`.

`botsh`:

- tries to run `imagemagick` and fails (because it is a fresh Ubuntu install)
- installs `imagemagick`
- attempts to run `ffmpeg` and fails
- installs `ffmpeg`
- attempts to turn frames into a video and fails (because it has not created them)
- writes a bash for-loop to generate 100 frames with `imagemagick`
- uses `ffmpeg` to convert them into a video

https://user-images.githubusercontent.com/46173/230953506-c8545345-c0a1-46b1-b937-458191fc2456.mp4

## Setup

Install with:

    pip install botsh

`botsh` expects an OpenAI API key to be provided as the `OPENAI_API_KEY`
environment variable.

`botsh` also requires Docker to be running on the system.

## Examples

    botsh "convert cat.jpg into a png file"

    botsh "use a remote service to find my public ip and base64 encode it"

    botsh "run pylint on the codebase in src/"

## Additional details

`botsh` will create a bare Ubuntu Docker container associated with
the current directory, or create one if one does not exist. botsh
will then attach the OpenAI API to a shell running in the container
to attempt to complete the given task.

The AI is explicitly told that it is allowed to install software,
and will typically install programs as needed to complete its task.
Installed software remains confined to the container.

## Observations

These observations relate to the default model, `text-davinci-003`. Using GPT-4 may improve things.

- It works best if you explicitly specify the files/paths you want to work with (use relative references).
  It is not good at figuring out what you mean.
- It often gets stuck in loops if it can't complete a task rather than giving up, despite the prompt
  telling it not to.
- It sometimes needs subtle encouragement to break a task down into multiple parts, instead of chaining together
  a long shell command, particularly when the command it wants to run has a bug. For example, instead of
  saying “convert foo.png to a gif and compute its md5 sum”, try “convert foo.png into a gif, and then compute
  its md5 sum”

## Container re-use

When `botsh` is invoked, the current working directory is mounted
into the container and can be modified by programs the agent runs.
The filesystem outside of the current working directory is sealed
off from the container.

Each directory that you run `botsh` in will get its own container,
which is reused for future invocations of `botsh` in that container.

You can pass `--wipe` to discard the existing container and start a
new one before running your task. You can also pass `--rm` to remove
a container at the end of your task.

Containers are also removed if you purge containers in Docker with
`docker container prune`

## Usage

```
usage: botsh [-h] [--max-rounds MAX_ROUNDS] [--model MODEL] [--image IMAGE] [--shell-command SHELL_COMMAND] [--save-transcript] [--wipe] prompt

Task runner powered by OpenAI and Docker. Invoke botsh by providing a task as a command line argument. botsh will create a bare Ubuntu Docker container associated with the current directory, or create one if one does not exist. botsh will then attach the OpenAI API to a shell running in the container to attempt to complete the given task.

positional arguments:
  prompt                Prompt to execute.

options:
  -h, --help            show this help message and exit
  --max-rounds MAX_ROUNDS
  --model MODEL         OpenAI text completion model to use.
  --image IMAGE         Docker image to use.The current hard-coded prompt works for Debian-derived distributions.
  --shell-command SHELL_COMMAND
                        Shell to invoke within the container.
  --save-transcript     Save transcript to file
  --wipe                Start with a fresh container even if one exists for this directory.
```
