# Nano Bots 💎 🤖

A Ruby implementation of the [Nano Bots](https://github.com/icebaker/nano-bots) specification.

![Ruby Nano Bots](https://user-images.githubusercontent.com/113217272/237839690-7880915a-b287-4484-a75e-0b96284b8a32.png)
_Image artificially created by Midjourney through a prompt generated by a Nano Bot specialized in Midjourney._

https://user-images.githubusercontent.com/113217272/238141567-c58a240c-7b67-4b3b-864a-0f49bbf6e22f.mp4

- [Setup](#setup)
  - [Docker](#docker)
- [Usage](#usage)
  - [Command Line](#command-line)
  - [Library](#library)
- [Cartridges](#cartridges)
  - [Marketplace](#marketplace)
- [Security and Privacy](#security-and-privacy)
  - [Cryptography](#cryptography)
  - [End-user IDs](#end-user-ids)
  - [Decrypting](#decrypting)
- [Providers](#providers)
- [Debugging](#debugging)
- [Development](#development)
  - [Publish to RubyGems](#publish-to-rubygems)

## Setup

For a system usage:

```sh
gem install nano-bots -v 0.0.10
```

To use it in a project, add it to your `Gemfile`:

```ruby
gem 'nano-bots', '~> 0.0.10'
```

```sh
bundle install
```

For credentials and configurations, relevant environment variables can be set in your `.bashrc`, `.zshrc`, or equivalent files, as well as in your Docker Container or System Environment. Example:

```sh
export OPENAI_API_ADDRESS=https://api.openai.com
export OPENAI_API_KEY=your-access-token

export NANO_BOTS_ENCRYPTION_PASSWORD=UNSAFE
export NANO_BOTS_END_USER=your-user

# export NANO_BOTS_STATE_DIRECTORY=/home/user/.local/state/nano-bots
# export NANO_BOTS_CARTRIDGES_DIRECTORY=/home/user/.local/share/nano-bots/cartridges
```

Alternatively, if your current directory has a `.env` file with the environment variables, they will be automatically loaded:

```sh
OPENAI_API_ADDRESS=https://api.openai.com
OPENAI_API_KEY=your-access-token

NANO_BOTS_ENCRYPTION_PASSWORD=UNSAFE
NANO_BOTS_END_USER=your-user

# NANO_BOTS_STATE_DIRECTORY=/home/user/.local/state/nano-bots
# NANO_BOTS_CARTRIDGES_DIRECTORY=/home/user/.local/share/nano-bots/cartridges
```

## Docker

Clone the repository and copy the Docker Compose template:

```
git clone git@github.com:icebaker/ruby-nano-bots.git
cd ruby-nano-bots
cp docker-compose.example.yml docker-compose.yml
```

Set your provider credentials and choose your desired directory for the cartridges files:

```yaml
version: '3.7'

services:
  nano-bots:
    image: ruby:3.2.2-slim-bullseye
    command: sh -c "gem install nano-bots -v 0.0.10 && bash"
    environment:
      OPENAI_API_ADDRESS: https://api.openai.com
      OPENAI_API_KEY: your-access-token
      NANO_BOTS_ENCRYPTION_PASSWORD: UNSAFE
      NANO_BOTS_END_USER: your-user
    volumes:
      - ./your-cartridges:/.local/share/nano-bots/cartridges
      - ./your-state:/.local/state/nano-bots
```

Enter the container:
```sh
docker compose run nano-bots
```

Start playing:
```sh
nb - - eval "hello"
nb - - repl

nb assistant.yml - eval "hello"
nb assistant.yml - repl
```

## Usage

### Command Line

After installing the gem, the `nb` binary command will be available for your project or system.

Examples of usage:

```bash
nb - - eval "hello"
# => Hello! How may I assist you today?

nb to-en-us-translator.yml - eval "Salut, comment ça va?"
# => Hello, how are you doing?

nb midjourney.yml - eval "happy cyberpunk robot"
# => A cheerful and fun-loving robot is dancing wildly amidst a
#    futuristic and lively cityscape. Holographic advertisements
#    and vibrant neon colors can be seen in the background.

nb lisp.yml - eval "(+ 1 2)"
# => 3

cat article.txt |
  nb to-en-us-translator.yml - eval |
  nb summarizer.yml - eval
# -> LLM stands for Large Language Model, which refers to an
#    artificial intelligence algorithm capable of processing
#    and understanding vast amounts of natural language data,
#    allowing it to generate human-like responses and perform
#    a range of language-related tasks.
```

```bash
nb - - repl

nb assistant.yml - repl
```

```text
🤖> Hi, how are you doing?

As an AI language model, I do not experience emotions but I am functioning
well. How can I assist you?

🤖> |
```

All of the commands above are stateless. If you want to preserve the history of your interactions, replace the `-` with a state key:

```bash
nb assistant.yml your-user eval "Salut, comment ça va?"
nb assistant.yml your-user repl

nb assistant.yml 6ea6c43c42a1c076b1e3c36fa349ac2c eval "Salut, comment ça va?"
nb assistant.yml 6ea6c43c42a1c076b1e3c36fa349ac2c repl
```

You can use a simple key, such as your username, or a randomly generated one:

```ruby
require 'securerandom'

SecureRandom.hex # => 6ea6c43c42a1c076b1e3c36fa349ac2c
```

### Debugging

```sh
nb - - cartridge
nb cartridge.yml - cartridge

nb - STATE-KEY state
nb cartridge.yml STATE-KEY state
```

### Library

To use it as a library:

```ruby
require 'nano-bots/cli' # Equivalent to the `nb` command.
```

```ruby
require 'nano-bots'

NanoBot.cli # Equivalent to the `nb` command.

NanoBot.repl(cartridge: 'cartridge.yml') # Starts a new REPL.

bot = NanoBot.new(cartridge: 'cartridge.yml')

bot = NanoBot.new(
  cartridge: YAML.safe_load(File.read('cartridge.yml'), permitted_classes: [Symbol])
)

bot = NanoBot.new(
  cartridge: { ... } # Parsed Cartridge Hash
)

bot.eval('Hello')

bot.eval('Hello', as: 'eval')
bot.eval('Hello', as: 'repl')

# When stream is enabled and available:
bot.eval('Hi!') do |content, fragment, finished|
  print fragment unless fragment.nil?
end

bot.repl # Starts a new REPL.

NanoBot.repl(cartridge: 'cartridge.yml', state: '6ea6c43c42a1c076b1e3c36fa349ac2c')

bot = NanoBot.new(cartridge: 'cartridge.yml', state: '6ea6c43c42a1c076b1e3c36fa349ac2c')

bot.prompt # => "🤖\u001b[34m> \u001b[0m"

bot.boot

bot.boot(as: 'eval')
bot.boot(as: 'repl')

bot.boot do |content, fragment, finished|
  print fragment unless fragment.nil?
end
```

## Cartridges

Here's what a Nano Bot Cartridge looks like:

```yaml
---
meta:
  symbol: 🤖
  name: Nano Bot Name
  author: Your Name
  version: 1.0.0
  license: CC0-1.0
  description: A helpful assistant.

behaviors:
  interaction:
    directive: You are a helpful assistant.

provider:
  id: openai
  credentials:
    address: ENV/OPENAI_API_ADDRESS
    access-token: ENV/OPENAI_API_KEY
  settings:
    user: ENV/NANO_BOTS_END_USER
    model: gpt-3.5-turbo
```

Check the Nano Bots specification to learn more about [how to build cartridges](https://spec.nbots.io/#/README?id=cartridges).

Try the [Nano Bots Clinic (Live Editor)](https://clinic.nbots.io) to learn about creating Cartridges.

### Marketplace

You can explore the Nano Bots [Marketplace](https://nbots.io) to discover new Cartridges that can help you.

## Security and Privacy

Each provider will have its own security and privacy policies (e.g. [OpenAI Policy](https://openai.com/policies/api-data-usage-policies)), so you must consult them to understand their implications.

### Cryptography

By default, all states stored in your local disk are encrypted.

To ensure that the encryption is secure, you need to define a password through the `NANO_BOTS_ENCRYPTION_PASSWORD` environment variable. Otherwise, although the content will be encrypted, anyone would be able to decrypt it without a password.

It's important to note that the content shared with providers, despite being transmitted over secure connections (e.g., [HTTPS](https://en.wikipedia.org/wiki/HTTPS)), will be readable by the provider. This is because providers need to operate on the data, which would not be possible if the content was encrypted beyond HTTPS. So, the data stored locally on your system is encrypted, which does not mean that what you share with providers will not be readable by them.

To ensure that your encryption and password are configured properly, you can run the following command:
```sh
nb security
```

Which should return:
```text
✅ Encryption is enabled and properly working.
     This means that your data is stored in an encrypted format on your disk.

✅ A password is being used for the encrypted content.
     This means that only those who possess the password can decrypt your data.
```

Alternatively, you can check it at runtime with:
```ruby
require 'nano-bots'

NanoBot.security.check
# => { encryption: true, password: true }
```

#### End-user IDs

A common strategy for deploying Nano Bots to multiple users through APIs or automations is to assign a unique [end-user ID](https://platform.openai.com/docs/guides/safety-best-practices/end-user-ids) for each user. This can be useful if any of your users violate the provider's policy due to abusive behavior. By providing the end-user ID, you can unravel that even though the activity originated from your API Key, the actions taken were not your own.

You can define custom end-user identifiers in the following way:

```ruby
NanoBot.new(environment: { NANO_BOTS_END_USER: 'custom-user-a' })
NanoBot.new(environment: { NANO_BOTS_END_USER: 'custom-user-b' })
```

Consider that you have the following end-user identifier in your environment:
```sh
NANO_BOTS_END_USER=your-name
```

Or a configuration in your Cartridge:
```yml
---
provider:
  id: openai
  settings:
    user: your-name
```

The requests will be performed as follows:

```ruby
NanoBot.new(cartridge: '-')
# { user: 'your-name' }

NanoBot.new(cartridge: '-', environment: { NANO_BOTS_END_USER: 'custom-user-a' })
# { user: 'custom-user-a' }

NanoBot.new(cartridge: '-', environment: { NANO_BOTS_END_USER: 'custom-user-b' })
# { user: 'custom-user-b' }
```

Actually, to enhance privacy, neither your user nor your users' identifiers will be shared in this way. Instead, they will be encrypted before being shared with the provider:

```ruby
'your-name'
# _O7OjYUESagb46YSeUeSfSMzoO1Yg0BZqpsAkPg4j62SeNYlgwq3kn51Ob2wmIehoA==

'custom-user-a'
# _O7OjYUESagb46YSeUeSfSMzoO1Yg0BZJgIXHCBHyADW-rn4IQr-s2RvP7vym8u5tnzYMIs=

'custom-user-b'
# _O7OjYUESagb46YSeUeSfSMzoO1Yg0BZkjUwCcsh9sVppKvYMhd2qGRvP7vym8u5tnzYMIg=
```

In this manner, you possess identifiers if required, however, their actual content can only be decrypted by you via your secure password (`NANO_BOTS_ENCRYPTION_PASSWORD`).

## Decrypting

To decrypt your encrypted data, once you have properly configured your password, you can simply run:

```ruby
require 'nano-bots'

NanoBot.security.decrypt('_O7OjYUESagb46YSeUeSfSMzoO1Yg0BZqpsAkPg4j62SeNYlgwq3kn51Ob2wmIehoA==')
# your-name

NanoBot.security.decrypt('_O7OjYUESagb46YSeUeSfSMzoO1Yg0BZJgIXHCBHyADW-rn4IQr-s2RvP7vym8u5tnzYMIs=')
# custom-user-a

NanoBot.security.decrypt('_O7OjYUESagb46YSeUeSfSMzoO1Yg0BZkjUwCcsh9sVppKvYMhd2qGRvP7vym8u5tnzYMIg=')
# custom-user-b
```

If you lose your password, you lose your data. It is not possible to recover it at all. For real.

## Providers

Currently supported providers:

- [x] [FastChat (Vicuna)](https://github.com/lm-sys/FastChat)
- [x] [Open AI](https://platform.openai.com/docs/api-reference)
- [ ] [Google PaLM](https://developers.generativeai.google/)
- [ ] [Alpaca](https://github.com/tatsu-lab/stanford_alpaca)
- [ ] [LLaMA](https://github.com/facebookresearch/llama)

Although only OpenAI has been officially tested, some of the open-source providers offer APIs that are compatible with OpenAI, such as [FastChat](https://github.com/lm-sys/FastChat#openai-compatible-restful-apis--sdk). Therefore, it is highly probable that they will work just fine.

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

gem push nano-bots-0.0.10.gem
```
