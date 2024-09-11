<p align="center">
  <a href="https://github.com/efugier/smartcat/discussions">
    <img src="https://img.shields.io/badge/commmunity-discussion-blue?style=flat-square" alt="community discussion">
  </a>
  <a href="https://github.com/efugier/smartcat/actions/workflows/ci.yml">
      <img src="https://github.com/efugier/smartcat/actions/workflows/ci.yml/badge.svg?branch=main" alt="Github Actions CI Build Status">
  </a>
  <a href="https://crates.io/crates/smartcat">
      <img src="https://img.shields.io/crates/v/smartcat.svg?style=flat-square" alt="crates.io">
  </a>
  <br>
</p>

<p align="center">
  <img src="assets/sc_logo.png" width="200">
</p>

# smartcat (sc)

Puts a brain behind `cat`! CLI interface to bring language models in the Unix ecosystem and allow terminal power users to make the most out of llms while keeping full control.

<p align="center">
  <img src="assets/workflow.gif" />
</p>


What makes it special:

- made for power users, tailor the config to reduce overhead on your most frequent tasks;
- minimalist, built according the unix philosophy with terminal and editor integration in mind;
- good io handling to insert user input in prompts and use the result in cli-based workflows;
- built-in partial prompt to make the model play nice as a cli tool;
- full configurability on which API, LLM version and temperature you use;
- write and save your own prompt templates for faster reccuring tasks (simplify, optimize, tests, etc);
- conversation support;
- glob expressions to include context files.

Currently supports the following APIs:

- Local runs with **[Ollama](https://github.com/ollama/ollama/blob/main/docs/README.md)** or any server compliant with its format, see the [Ollama setup](#ollama-setup) section for the free and easiest way to get started!  
Answers might be slow depending on your setup, you may want to try the third party APIs for an optimal workflow.
- **[OpenAi](https://platform.openai.com/docs/models/overview)**, **[Mistral AI](https://docs.mistral.ai/getting-started/models/)**, **[Anthropic](https://docs.anthropic.com/claude/docs/models-overview)**, **[Groq](https://console.groq.com/docs/models)**.

# Table of Contents

- [Installation](#installation-)
- [Usage](#usage)
- [A few examples to get started 🐈‍⬛](#a-few-examples-to-get-started-)
  - [Integrating with editors](#integrating-with-editors)
    - [Example workflows](#example-workflows)
- [Configuration](#configuration) ← please read this carefully
    - [Ollama setup](#ollama-setup) ← easiest way to get running for free
- [Voice](#voice)
- [How to help?](./CONTRIBUTING.md)

## Installation

On the first run (`sc`), it will ask you to generate some default configuration files and give pointers on how to finalize the install (see the [configuration section](#Configuration)).

The minimum config requirement is a `default` prompt calling a setup API (either remote with api key or local with ollama).

Now on how to get it.

### With Cargo

With an **up to date** [rust and cargo](https://www.rust-lang.org/tools/install) setup (you might consider running `rustup update`):

```
cargo install smartcat
```

run this command again to update `smartcat`.

### By downloading the binary

Chose the one compiled for your platform on the [release page](https://github.com/efugier/smartcat/releases).

## Usage

```text
Usage: sc [OPTIONS] [INPUT_OR_TEMPLATE_REF] [INPUT_IF_TEMPLATE_REF]

Arguments:
  [INPUT_OR_TEMPLATE_REF]  ref to a prompt template from config or straight input (will use `default` prompt template if input)
  [INPUT_IF_TEMPLATE_REF]  if the first arg matches a config template, the second will be used as input

Options:
  -e, --extend-conversation        whether to extend the previous conversation or start a new one
  -r, --repeat-input               whether to repeat the input before the output, useful to extend instead of replacing
  -v, --voice                      whether to use voice for input
      --api <API>                  overrides which api to hit [possible values: another-api-for-tests, ollama, anthropic, groq, mistral, openai]
  -m, --model <MODEL>              overrides which model (of the api) to use
  -t, --temperature <TEMPERATURE>  temperature higher means answer further from the average
  -l, --char-limit <CHAR_LIMIT>    max number of chars to include, ask for user approval if more, 0 = no limit
  -c, --context <CONTEXT>...       glob patterns or list of files to use the content as context
                                   make sure it's the last arg.
  -h, --help                       Print help
  -V, --version                    Print version
```

You can use it to **accomplish tasks in the CLI** but **also in your editors** (if they are good unix citizens, i.e. work with shell commands and text streams) to complete, refactor, write tests... anything!

The key to make this work seamlessly is a good default prompt that tells the model to behave like a CLI tool an not write any unwanted text like markdown formatting or explanations.

## A few examples to get started 🐈‍⬛

```
sc "say hi"  # just ask (uses default prompt template)

sc -v  # use your voice to ask (then press <space> to stop the recording)

sc test                         # use templated prompts
sc test "and parametrize them"  # extend them on the fly

sc "explain how to use this program" -c **/*.md main.py  # use files as context

git diff | sc "summarize the changes"  # pipe data in

cat en.md | sc "translate in french" >> fr.md   # write data out
sc -e "use a more informal tone" -t 2 >> fr.md  # extend the conversation and raise the temprature
```

### Integrating with editors

The key for a good integration in editors is a good default prompt (or set of) combined with the `-p` flag for precising the task at hand.
The `-r` flag can be used to decide whether to replace or extend the selection.

#### Vim

Start by selecting some text, then press `:`. You can then pipe the selection content to `smartcat`.

```
:'<,'>!sc "replace the versions with wildcards"
```

```
:'<,'>!sc "fix this function"
```

will **overwrite** the current selection with the same text transformed by the language model.

```
:'<,'>!sc -r test
```

will **repeat** the input, effectively appending at the end of the current selection the result of the language model.

Add the following remap to your vimrc for easy access:

```vimrc
nnoremap <leader>sc :'<,'>!sc
```

#### Helix and Kakoune

Same concept, different shortcut, simply press the pipe key to redirect the selection to `smartcat`.

```
pipe:sc test -r
```
With some remapping you may have your most reccurrent action attached to few keystrokes e.g. `<leader>wt`!

#### Example Workflows

**For quick questions:**

```
sc "my quick question"
```

which will likely be **your fastest path to answer**: a shortcut to open your terminal (if you're not in it already), `sc` and you're set. No tab finding, no logins, no redirects etc.

**To help with coding:**

select a struct

```
:'<,'>!sc "implement the traits FromStr and ToString for this struct"
```

select the generated impl block

```
:'<,'>!sc -e "can you make it more concise?"
```

put the cursor at the bottom of the file and give example usage as input

```
:'<,'>!sc -e "now write tests for it knowing it's used like this" -c src/main.rs
```

...

**To have a full conversation with a llm from a markdown file:**

```
vim problem_solving.md

> write your question as comment in the markdown file then select your question
> and send it to smartcat using the aforementioned trick, use `-r` to repeat the input.

If you wan to continue the conversation, write your new question as a comment and repeat
the previous step with `-e -r`.

> This allows you to keep track of your questions and make a nice reusable document.
```

<p align="center">
  <img src="assets/qatohtml.gif" />
</p>


# Configuration

- by default lives at `$HOME/.config/smartcat`
- the directory can be set using the `SMARTCAT_CONFIG_PATH` environement variable
- use `#[<input>]` as the placeholder for input when writing prompts, if none is provided, it will be automatically added at the end of the last user message
- the default model is a local `phi3` ran with ollama but I recommend trying the latest ones and see which one works best for you;
- the prompt named `default` will be the one used by default.
- you can play with the temperature and set a default for each prompt depending on its use case;

Three files are used:

- `.api_configs.toml` stores your credentials, you need at least one provider with API with key or a local ollama setup;
- `prompts.toml` stores you prompt templates, you need at least the `default` prompt;
- `conversation.toml` stores the latest chat if you need to continue it, it's automanaged but you can make backups if you want.

`.api_configs.toml`

```toml
[ollama]  # local API, no key required
url = "http://localhost:11434/api/chat"
default_model = "phi3"
timeout_seconds = 180  # default timeout if not specified

[openai]  # each supported api has their own config section with api and url
api_key = "<your_api_key>"
default_model = "gpt-4-turbo-preview"
url = "https://api.openai.com/v1/chat/completions"

[mistral]
api_key_command = "pass mistral/api_key"  # you can use a command to grab the key
default_model = "mistral-medium"
url = "https://api.mistral.ai/v1/chat/completions"

[groq]
api_key_command = "pass groq/api_key"
default_model = "llama3-70b-8192"
url = "https://api.groq.com/openai/v1/chat/completions"

[anthropic]
api_key = "<yet_another_api_key>"
url = "https://api.anthropic.com/v1/messages"
default_model = "claude-3-opus-20240229"
version = "2023-06-01"
```

`prompts.toml`

```toml
[default]  # a prompt is a section
api = "ollama"  # must refer to an entry in the `.api_configs.toml` file
model = "phi3"  # each prompt may define its own model

[[default.messages]]  # then you can list messages
role = "system"
content = """\
You are an extremely skilled programmer with a keen eye for detail and an emphasis on readable code. \
You have been tasked with acting as a smart version of the cat unix program. You take text and a prompt in and write text out. \
For that reason, it is of crucial importance to just write the desired output. Do not under any circumstance write any comment or thought \
as you output will be piped into other programs. Do not write the markdown delimiters for code as well. \
Sometimes you will be asked to implement or extend some input code. Same thing goes here, write only what was asked because what you write will \
be directly added to the user's editor. \
Never ever write ``` around the code. \
"""

[empty]  # always nice to have an empty prompt available
api = "openai"
# not mentioning the model will use the default from the api config
messages = []

[test]
api = "anthropic"
temperature = 0.0

[[test.messages]]
role = "system"
content = """\
You are an extremely skilled programmer with a keen eye for detail and an emphasis on readable code. \
You have been tasked with acting as a smart version of the cat unix program. You take text and a prompt in and write text out. \
For that reason, it is of crucial importance to just write the desired output. Do not under any circumstance write any comment or thought \
as you output will be piped into other programs. Do not write the markdown delimiters for code as well. \
Sometimes you will be asked to implement or extend some input code. Same thing goes here, write only what was asked because what you write will \
be directly added to the user's editor. \
Never ever write ``` around the code. \
"""

[[test.messages]]
role = "user"
# the following placeholder string #[<input>] will be replaced by the input
# each message seeks it and replaces it
content ='''Write tests using pytest for the following code. Parametrize it if appropriate.

#[<input>]
'''
```

```toml
url = "https://api.openai.com/v1/audio/transcriptions"
# make sure this command fit you OS and works on its own
recording_command = "arecord -f S16_LE --quiet <audio_file_path_placeholder>"
model = "whisper-1"
api = "openai"
```

see [the config setup file](./src/config/mod.rs) for more details.

## Ollama setup

1. [Install Ollama](https://github.com/ollama/ollama#ollama)
2. Pull the model you plan on using `ollama pull phi3`
3. Test the model `ollama run phi3 "say hi"`
4. Make sure the serving is available `curl http://localhost:11434` which should say "Ollama is running", else you might need to run `ollama serve`
5. `smartcat` will now be able to reach your local ollama, enjoy!

⚠️ Answers might be slow depending on your setup, you may want to try the third party APIs for an optimal workflow. Timeout is configurable and set to 30s by default.

# Voice, deprecation in progress

⚠️ **Testing in progress** I only have a linux system and wasn't able to test the recording commands for other OS. The good news is you can make up your own that works and then plug it in the config.

Use the `-v` flag to ask for voice input then press space to end it. It will replace the prompt customization arg.

- uses openai whisper
- make sure your `recording_command` field works in your termimal command, it should create a wav file
- requires you to have an openai key in your `.api_keys.toml`
- you can still use any prompt template or text model to get your output

```
sc -v

sc test -v

sc test -v -c src/**/*
```

This could be a good accessiblity feature but I personnaly never use it and given its current state I am considering removing it.

## How does it work?

`smartcat` call an external program that handles the voice recording and instructs it to save the result in a wav file. It then listens to keyboard inputs and stops the recording when space is pressed.

The recording is then sent to a speech to text model, the resulting transcript is finally added to the prompt and sent to the text model to get an answer.

On linux: TODO
On Mac: TODO
On windows: TODO

To debug, you can check the `conversation.toml` file or listen to the `audio.wav` in the smart config home and see what the model heard and transcripted.

This feature shoud be offered as an extra down the road, totally optional on install. PRs are welcomed!


## How to help?

See [CONTRIBUTING.md](./CONTRIBUTING.md).
