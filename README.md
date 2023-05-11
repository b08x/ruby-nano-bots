# Nano Bots 💎 🤖

A Ruby implementation of the [Nano Bots](https://github.com/icebaker/nano-bots) specification.

- [Setup](#setup)
- [Usage](#usage)
  - [Command Line](#command-line)
  - [Library](#library)
- [Cartridges](#cartridges)
- [Development](#development)
  - [Publish to RubyGems](#publish-to-rubygems)

## Setup

For a system usage:

```sh
gem install nano-bots -v 0.0.1
```

To use it in a project, add it to your `Gemfile`:

```ruby
gem 'nano-bots', '~> 0.0.1'
```

```sh
bundle install
```

For credentials and configurations, relevant environment variables can be set in your `.bashrc`, `.zshrc`, or equivalent files, as well as in your Docker Container or System Environment. Example:

```sh
OPENAI_API_ADDRESS=https://api.openai.com
OPENAI_API_ACCESS_TOKEN=your-token
OPENAI_API_USER_IDENTIFIER=your-user
NANO_BOTS_STATE_DIRECTORY=/home/your-user/.local/share/.nano-bots
```

Alternatively, if your current directory has a `.env` file with the environment variables, they will be automatically loaded.

## Usage

### Command Line

After installing the gem, the `rnb` binary command will be available for your project or system.

Examples of usage:

```bash
rnb to-en-us-translator.yml - eval "Salut, comment ça va?"
# => Hello, how are you doing?

rnb midjourney.yml - eval "happy and friendly cyberpunk robot"
# => The robot exploring a bustling city, surrounded by neon lights
#    and high-rise buildings. The prompt should include colorful
#    lighting and a sense of excitement in the facial expression.

rnb lisp.yml - eval "(+ 1 2)"
# => 3

cat article.txt |
  rnb to-en-us-translator.yml - eval |
  rnb summarizer.yml - eval
# -> LLM stands for Large Language Model, which refers to an
#    artificial intelligence algorithm capable of processing
#    and understanding vast amounts of natural language data,
#    allowing it to generate human-like responses and perform
#    a range of language-related tasks.
```

```bash
rnb assistant.yml - repl
```

All of the commands above are stateless. If you want to preserve the history of your interactions, replace the `-` with a state key. You can use a simple key, such as your username, or a randomly generated one:

```ruby
require 'securerandom'

SecureRandom.hex # => 6ea6c43c42a1c076b1e3c36fa349ac2c
```

```bash
rnb assistant.yml your-user eval "Salut, comment ça va?"
rnb assistant.yml your-user repl

rnb assistant.yml 6ea6c43c42a1c076b1e3c36fa349ac2c eval "Salut, comment ça va?"
rnb assistant.yml 6ea6c43c42a1c076b1e3c36fa349ac2c repl
```

### Library

To use it as a library:

```ruby
require 'nano-bots/cli' # Equivalent to the `rnb` command.
```

```ruby
require 'nano-bots'

NanoBot.cli # Equivalent to the `rnb` command.

NanoBot.repl(cartridge: 'cartridge.yml') # Starts a new REPL.

bot = NanoBot.new(cartridge: 'cartridge.yml')

bot.eval('Hello')

bot.repl # Starts a new REPL.

NanoBot.repl(cartridge: 'cartridge.yml', state: '6ea6c43c42a1c076b1e3c36fa349ac2c')

bot = NanoBot.new(cartridge: 'cartridge.yml', state: '6ea6c43c42a1c076b1e3c36fa349ac2c')
```

## Cartridges

Here's what a Nano Bot Cartridge looks like:

```yaml
---
name: Assistant
version: 0.0.1

behaviors:
  interaction:
    directive: You are a helpful assistant.

interfaces:
  repl:
    prompt:
      - text: '🤖'
      - text: '> '
        color: blue

provider:
  name: openai
  settings:
    model: gpt-3.5-turbo
    credentials:
      address: ENV/OPENAI_API_ADDRESS
      access-token: ENV/OPENAI_API_ACCESS_TOKEN
      user-identifier: ENV/OPENAI_API_USER_IDENTIFIER
```

Check the Nano Bots specification to learn more about [how to build cartridges](https://icebaker.github.io/nano-bots/#/README?id=cartridges).

## Development

```bash
bundle
rubocop -A
rspec
```

### Publish to RubyGems

```bash
gem build nano-bots.gemspec

gem signin

gem push nano-bots-0.0.1.gem
```