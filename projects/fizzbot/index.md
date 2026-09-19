---
title: "Fizzbot"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "Fine-tuning a chat model on my cohort's Discord server and wiring it up as a bot."
thumbnail: "media/thumbnail.png"
date: 2026-01-04
status: "complete"
featured: false
tags: ["ml"]
media:
  - "media/example-text.png"
  - "media/discord-profile.png"
---

# Fizzbot

![Fizzbot](media/thumbnail.png)

I wanted to see if a model could sound like one particular Discord server
instead of a generic chatbot. That meant I needed the whole path: exporting the
chat history, preserving who said what, training the model, decoding its output,
and finally putting it back into Discord.

## Turning Discord history into training data

The input is a pile of Discrub channel exports. My generator normalizes every
message down to a username, content, and timestamp, sorts them within their
channel, throws out the empty, URL-only, and mention-only ones, and writes
context/target pairs as JSONL.

Speaker identity has to survive that. Usernames become tokens like `<S0>` and
`<S1>`, every message ends with `<EOT>`, and a separate speaker map turns those
tokens back into `username: message` at inference time. Context never crosses
channels.

## Training and inference

The main config fine-tunes Mistral-7B v0.1 with 4-bit QLoRA instead of training
anything from scratch. LoRA adapters keep the runs small enough to iterate on,
and there is a tiny CPU config for checking the pipeline still works. The
inference CLI can load the latest run or a specific checkpoint, generate a few
turns, and decode the speaker tokens back into chat.

![Example generated text](media/example-text.png)

## Putting it in Discord

The bot wrapper is written in Rust with Serenity. It starts the model command as
a child process, maps Discord users to the saved speaker tokens, and exchanges
prompts and responses over standard input and output. It replies when mentioned,
removes the triggering mention from the prompt, and sanitizes its response so
generated text can't ping people.

![Fizzbot Discord profile](media/discord-profile.png)

The repository has Make and Docker commands for each step, from generating the
dataset to launching the bot. Training the model was only one piece. Most of the
work was making sure the same speaker mapping survived every step from the raw
Discord export to the bot's reply.

## Repository

[github.com/georgesleen/fizzbot-2](https://github.com/georgesleen/fizzbot-2)
