---
title: "N8N - Telegram Bot for ZH-DE Translation With Anki"
date: 2025-08-24T14:22:07+08:00
draft: false
---

## Background
I'm coming to Germany in a few days to seek for a job. I want to have a tool to help me speak German better. I tried lots of dictionary apps, but no one meet my needs. An ideal app should accept Chinese voice input and translate it to German, and then store the translation in Anki. In that way, I can efficiently memorize the translation and practice speaking German. Therefore, I decided to build one by myself. 

## Workflow
The workflow shown as below, which is too long to fit the screen properly. 

The nodes are grouped into 4 parts:
1. Input: Get voice from Telegram,
2. Processing: Transcribe audio to text, translate text to German, generate explanation in Chinese,
3. Output: Send messages to Telegram, and
4. Storage: Save to Anki.




0. Reference
I was refering to this video when setting up this workflow. I highly recommend you to check this video before starting. This post will skip the basics and focus on the hard and different part of this specific needs.

## Infrastructure
I selfhost all the services in Coolify, including
1. n8n community version,
2. Anki's desktop application in docker,
3. and Traefik as the reverse proxy service (which is part of the Coolify service).



## Detail Implementations
### Replace If Node with Switch Node
The nodes before the Agent Node is almost identical, except that I'm using a Switch Node instead of an If Node because the If Node does't pass the value to next step correctly with the following message:
```
No fields - node executed, but no items were sent on this branch
```

The same issue sames happening to others as well, according to this post: https://community.n8n.io/t/switch-problem-no-fields-node-executed-but-no-items-were-sent-on-this-branch/134137

As suggested by its comments, I use a Switch Node to replace the If Node.

### Add a FFMPEG Node between Get-File Node and Transcribing-Audio Node
I have an issue when using OpenAI-compatible service. If you are using OpenAI's official service, you can skip this part. Below are the details about the problem and solutuon.

Telegram uses OGG or OGA to encode audio recordings sent by user. These formats are supported by OpenAI but may not be supported by third-party providers.

According to [this post](https://community.n8n.io/t/open-ai-transcribe-a-recording-telegram/149591), the file format error may rise if you don't have enough credits. However, this is not the case for me. I identified that its my service provider's issue by testing its endpoints in Postman with mp3 and ogg formats. The result is that mp3 works but not ogg. Therefore, I have to add a ffmpeg node to covert ogg to mp3. Below are the steps:
1. Build  n8n image with FFMPEG installed,
2. add the FFMPEG node to your n8n instance,
3. and configure the n8n node.

#### Build image
I prefer to build the image on-the-go and use `dockerfile_inline` inside a docker-compose file which keeps my code clean and simple. The following content is the docker-compose file I'm using on Coolify.
```
services:
  n8n:
    build:
      context: .
      dockerfile_inline: |
        FROM n8nio/n8n:latest
        USER root
        RUN apk add --update ffmpeg
        USER node
    # image: docker.n8n.io/n8nio/n8n
    environment:
      - SERVICE_FQDN_N8N_5678
      - 'N8N_EDITOR_BASE_URL=${SERVICE_FQDN_N8N}'
      - 'WEBHOOK_URL=${SERVICE_FQDN_N8N}'
      - 'N8N_HOST=${SERVICE_URL_N8N}'
      - 'GENERIC_TIMEZONE=${GENERIC_TIMEZONE:-Europe/Berlin}'
      - 'TZ=${TZ:-Europe/Berlin}'
    volumes:
      - 'n8n-data:/home/node/.n8n'
    healthcheck:
      test:
        - CMD-SHELL
        - 'wget -qO- http://127.0.0.1:5678/'
      interval: 5s
      timeout: 20s
      retries: 10

```

#### Add ffmpeg node
After building the image, you need to add the FFMPEG node to your n8n instance. You can find it in the marketplace. I'm using [n8n-nodes-ffmpeg](https://www.npmjs.com/package/n8n-nodes-ffmpeg) with its `Custom Command` operation. The command defined in this node is 
```
-i "{input}" -vf "transpose=1" "{output}"
```
where `transpose=1` is used to convert the audio to mono.


### Agent Setup

I have used XML format in prompts to generate structured format to me, and it works fantastically. So I use the same trick here. You can find my prompt in this link. It was completed in two steps:
1. At first, I just use nature language to init a prompt, and testing it in ChatGPT.
2. Then, I ask ChatGPT to help me format it to XML format. After serveral iterations, it becomes ideal with greate enhancements from ChatGPT's suggestions.


Next to the Agent Node, I add a XML Node because n8n uses JSON as the standard data format. 

### Telegram Setup for Sending Messages
Nodes after the Agent Node are built application-specific functions. In my case:
    1. Telegram needs to send German translations and Chinese explainations to user.
    2. In addition, Telegram also need to send a audio file of the German translation, generated by a TTS service.
    3. Once the messages are sent, they should also be saved to Anki via API.

The first two steps are straight forward:
1. Translation and Explaination are sent in two messages one by one, for better visibility.
2. A "Recording" status is sent before starting TTS service to hint user that they will also receive a audio version of the translation. 

The last step involves Anki setup in Coolify. I prefer to put it in an independent section as it's not an n8n guide, but rather an infrascture setup.

### Anki API Setup
To have running Anki API service, you need to do the two steps:
1. Install and run the Anki desktop version in docker. Anki's GUI client can be accessed via VNC.
2. Install the Anki-connect plugin which provide API endpoints to manipulations to data and GUI of the desktop client.
3. Protect the Anki services with basic auth.

#### Hardware Requirement
Running a desktop environment docker requires a lot resource. From `docker stats`, I can see my service uses 517MB memory. To be on the safe side, you should have at least 756 free memory. 
```
CONTAINER ID   NAME                                    CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS 
11e345a80f64   anki-desktop-e4k004csggkkwwo48oo4g8ow   0.48%     517.6MiB / 3.823GiB   13.22%    1.96MB / 1.15MB   196MB / 7.76MB    113 
```

#### Anki in Docker
I'm using the following docker-compose file to run the Anki service. Because I'm running in on Coolify, the label configurations for Traefik is significantly simplified. Basic auth is added via the `labels` section. You only need to define it and coolify will automatically add it to the middlewares of the service. 

```
services:
  anki-desktop:
    image: 'mlcivilengineer/anki-desktop-docker:main'
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - './anki_data:/config'
    expose:
      - '3000'
      - '8765'
    labels:
      # === add basic auth
      - 'traefik.http.middlewares.anki.basicauth.users=${AUTH_USER}:${HT_AUTH_PWD}'
      # === done add basic auth
```

If you are not using Coolify, you may find an reference docker-compose here: Pending sumplementry file


In this doker, there are two ports: `3000` for VNC and `8765` for Anki-connect. To configure two DNS records for the ports in Coolify, you should map the ports and donmains in Coolify like that:
```
https://anki.harrylearns.com:3000,https://api-anki.harrylearns.com:8765
```  
![anki-fqdn-setup](/20250824-n8n-telegram-bot/image.png)
Coolify will parse this configuration and forward traffic of `https://anki.harrylearns.com` to the `3000` port of this container.

To finish your setup Anki, you need to 
1. install Anki-connect plugin,
2. login to Anki Web account so you can use Anki Web's sync function,
3. and restart your container to make Anki-connect working.

