---
title: "Retrieve hyperlinks via cli with shuttle"
date: 2022-07-30T12:49:42-04:00
draft: false
toc: false
images:
tags: 
  - dev
  - bash
---

>To infinity and beyond.
>
>-Buzz Lightyear

I spend a lot of time at the terminal, either in vscode or a zsh shell.

Two apps I use often are [tldr](https://github.com/tldr-pages/tldr) and [cht.sh](https://github.com/chubin/cheat.sh) because:
 1) I don't remember the bookmarks that I have.
 2) I have too many bookmarks.
   
 While tldr and cht.sh are handy for retrieving public info, I needed a similar utility for work.  Our tech stack has a lot of astronomy jargon e.g. galaxy, quadrant, planet, etc. and since we embrace a DevOps culture I wanted to build something that blends with our spatial theme and also fits nicely with multiple teams.

 ---

## Introducing [shuttle](https://github.com/codedinsugar/shuttle) - a simple cli tool to retrieve hyperlinks.

Usage is fairly simple, just run `./shuttle $arg` and it'll retrieve the contents of a respectively named json file.

## Example:
```
#microservice1.json is in the shuttle-files path

./shuttle microservice1
{
  "name": "microservice1",
  "wiki": "https://microservice1/wiki",
  "playbook": "https://microservice1/playbook",
  "diagram": "https://microservice1/diagram",
  "communication": "https://microservice1/comms"
}
```

Check it out [here](https://github.com/codedinsugar/shuttle)!