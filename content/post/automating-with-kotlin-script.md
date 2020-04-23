---
title: "Automating development tasks with Kotlin Script"
date: 2020-04-23T07:46:28+02:00
draft: false
categories: ["Android", "Kotlin"]
---

Kotlin Script is the perfect glue for piecing together a variety of shell commands to automate your development tasks.
Syntax we know and love. More intuitive than Bash scripts.
Some autocomplete

You probably need kscript
[kscript](https://github.com/holgerbrandl/kscript)
Dependencies (but these should be minimized)
Import support for better script organization & reuse

How we used it to automate our regression testing process
1. Display a list of app versions from AppCenter using their CLI, prompt the user to choose one.
2. Download the selected app version using `curl`.
3. Install the selected app version on a connected device/emulator using `adb`.
4. Download the latest deep link config YAML from a separate GitHub repo using their CLI.
5. Convert the YAML to JSON and flatten it using `yq`.
6. Parse the JSON into objects in Kotlin Script using Moshi.
7. Build screen deep links from the config in Kotlin Script.
8. Ask the user whether they'd like to record the screen launches, start a screen recording using `adb` if so.
9. Launch each screen deep link in succession, prompt the user to press Enter to continue when waiting for device interaction.
10. Scroll the screen with `adb`.
11. Repeat Steps 9-10 for each screen.
12. If a screen recording was started in Step 8, end it and copy the MP4 file to the working directory of the local machine.