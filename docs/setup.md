There are several steps you will need to follow to setup Coordinate on your
own machine.

This setup guide should take roughly 30 minutes to 1 hour, even if you are not
acquainted with Discord.

## Setting up a server

The first step is to create a new server in Discord. This will be your test server. You should use
this server template, as it has all the necessary permissions for Coordinate to
work: [Coordinate Server Template](https://discord.new/QU2VzzDwHhfM).

You will need to enable community mode in your new server for question-and-answer
channels (forum channels) to appear! You can convert your server to a community
server by going to `'Server Settings' > 'Enable Community' on the left bar`.
Choose #rules as the rules channel, and #admin-chat as the community updates
channel.

Add any number of forum channels with names ending with "questions". In each of$
them, add the tags "Answered" and "Unanswered".

## Create and invite your new bot

First, you will need to create a Discord bot. This bot is the user that your
students (or you) will interact with in the Discord interface. You will also
create a test server that you will use to interact with your bot in a controlled
environment.

1. Add a bot account in the [Discord Developers Portal](https://discord.com/developers/applications/).
   Create a new application and add a bot in the Bot section. There, get your bot token.
2. Paste this link into your server, but replace `<APP_ID>` with the ID of your
   application shown in Discord.
```
https://discord.com/api/oauth2/authorize?client_id=<APP_ID>&permissions=9887014579026&scope=bot
```
3. Click on that link and click Accept to add your new application into your server!

## Install `pgvector` and setup Postgres

Before proceeding, ensure that Docker is installed. Afterwards, run the command
below to build a version of `pgvector` compatible with the bot and run it for the
bot to access.

> [!CAUTION]
> If you are deploying this bot on a public server, please change the default
username/password in the provided `docker-compose.yaml`, to prevent unauthorized
access to the bot's database.

```bash
$ cd ~
$ git clone https://github.com/pgvector/pgvector
$ cd pgvector
$ docker build -t pgvector:16 .
# navigate back to the project folder for your bot
$ cd your-bot-folder
$ docker-compose up # <-- Check that you see something like "database is ready to accept connections", then ctrl c to cancel
$ docker-compose up -d # <-- Same as last command, just running in the background
```

Log in to your new database using `psql mydb -h localhost -U <user>`
(default username is `abc` and default password is `def`) and run the following command:

```sql
CREATE EXTENSION vector;
```

This will create the `pgvector` extension in your new database! Great -- your
database should be ready to go!

## Environment variables

You will want to create a `.env` file in the `src/` directory -- this is where
you will store all credentials and configuration variables related to your running
bot instance.

Here is an example of what your `.env` file should look like:

```env
# The token for your Discord bot (from the Discord Developer Portal)
DISCORD_TOKEN=
# You definitely want this on!
DEV_MODE=TRUE
# The server ID of the server you want to use for testing
GUILD_ID=
# The URL of your PostgreSQL database (this is likely what the URL will be if you are using Docker Compose)
POSTGRES_URL="postgresql+asyncpg://abc:def@localhost/mydb"
# The URL of your Canvas instance
CANVAS_URL=
# Your Canvas API token
CANVAS_API_TOKEN=
# The URL of your Qualtrics instance
QUALTRICS_URL=
# Your Qualtrics API token
QUALTRICS_API_TOKEN=
# The datacenter of your Qualtrics instance
QUALTRICS_API_DATACENTER=
# The ID of the survey you want to use for Coordinate
QUALTRICS_SURVEY_ID=
# The ID of the filter (from your survey) you want to use for Coordinate
QUALTRICS_FILTER_ID=
# (optional) Your client ID for Codio
CODIO_CLIENT_ID=
# (optional) Your client secret for Codio
CODIO_CLIENT_SECRET=
# (optional) Your GitHub token
TOKEN_FOR_GH=
# (optional) Your NVIDIA NGC token (for LLM model)
NVIDIA_NGC_TOKEN=
# (optional) The name of the Canvas folder to use for sourcing LLM embeddings
LLAMA_SOURCE_FOLDER=
# (optional) The email of your Gradescope account
GRADESCOPE_EMAIL=
# (optional) The password of your Gradescope account
GRADESCOPE_PASSWORD=
# (optional) The course ID of the course you want to use for the bot
GRADESCOPE_COURSE_ID=
# (optional) The course ID of the course you want to use for pytest
GRADESCOPE_TEST_COURSE_ID=
```

### Qualtrics

The bot expects a certain survey format to run. Your Qualtrics account must be
able to access a survey with this format.

Here are the values to provide to those environment variables:

#### `QUALTRICS_API_DATACENTER (Field 2)`

![](../assets/user_qualtrics_table.png)

#### `QUALTRICS_SURVEY_ID (Field 3)`

![](../assets/qualtrics_ids_table.png)

#### `QUALTRICS_FILTER_ID (Field 4)`

![](../assets/qualtrics_filter.png)

## Run the Bot

Great, the setup should be complete! Take it for a spin!

```bash
# Installing dependencies listed in requirements.txt
python3.11 -m pip install -r requirements.txt
# Install pre-commit
pre-commit install --install-hooks
# Running the bot
python3.11 -m src.bot
```
