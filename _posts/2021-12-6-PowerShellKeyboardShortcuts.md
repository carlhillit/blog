---
layout: post
title: 7 Keyboard Shortcuts for PowerShell Speed and Efficiency
published: true
tags: [PowerShell]
---

7 of my favorite keyboard shortcuts to make using PowerShell faster and more efficient.

[![Watch the video](https://img.youtube.com/vi/cenJHwdz3Mk/hqdefault.jpg)](https://youtu.be/cenJHwdz3Mk)

# TL;DR

1. **UP arrow** key to cycle through previously entered commands
2. **Tab** to auto-complete
3. **HOME** and **END** keys to move the cursor to the beginning and end of a line
4. **Ctrl+HOME** and **Ctrl+END** to remove everything to the right or left of the cursor
5. **Ctrl+ARROW** keys to move one word block at a time
6. **Ctrl+DEL** and **Ctrl+BACKSPACE** to delete whole word blocks at a time
7. **Ctrl+SPACE** to bring up a menu of parameters

## Top 7 Keyboard Shortcuts

1. **UP arrow** key to cycle through previously entered commands

   The first keyboard shortcut that I'm going to show you is to use the UP arrow key to cycle through the previously entered commands. Here, we can see that I've previously entered the `Get-NetIPAddress` command. If I don't want to type that in again, I can hit the UP arrow key and it will bring that command back to the line.

   You can continue to press the UP arrow to cycle through your command history. If you've passed your desired command, use the **DOWN arrow key** to go backwards.

   ![_config.yml]({{ site.baseurl }}/images/keyboardshortcuts/updownarrow.gif)

2. **Tab** to auto-complete

   The next shortcut is probably the most time-saving shortcut on the list: Press the TAB key to auto-complete the current word. If I begin to type `Get-NetIP`, and hit TAB, PowerShell will auto-complete the command to `Get-NetIPAddress`.

   It works the same with parameters. Begin typing `-Inter`, press the TAB key, and it will auto-complete the parameter `-InterfaceIndex`. With `-InterfaceIndex`, there are a potentially large number of inputs, but only two for `-AddressFamily`: IPv4 and IPv6. Using TAB auto-complete will then show the first available input (IPv4). I can then continue to press the TAB key to cycle through the available parameter inputs.

   If you've already passed your desired selection, you can hold the SHIFT key while pressing tab (**Shift+Tab**) to cycle backwards.

   *(The more you type, the fewer options to cycle though and the more accurate the tab completion will be)*

   ![_config.yml]({{ site.baseurl }}/images/keyboardshortcuts/tab.gif)

3. **HOME** and **END** keys to move the cursor to the beginning and end of a line

   The next shortcut is to use the HOME key to move the cursor to beginning of a line.
   Use the END key to move the cursor to the end of a line.

   This helpful for a command like `Get-NetIPAddress` where I can move my cursor to the beginning of a line, press DELETE and replace the "G" with an "S" to use the `Set-NetIPAddress` command with the previously used parameters.

   ![_config.yml]({{ site.baseurl }}/images/keyboardshortcuts/home.gif)

4. **Ctrl+HOME** and **Ctrl+END** to remove everything to the right or left of the cursor

   Along the same lines, if I wanted to clear everything to the left or right of the cursor, I can hold the Ctrl key while pressing the HOME and END keys.
   To clear the entire line, press the END key to move the cursor to the end of a line, then Ctrl+HOME to clear.

    ![_config.yml]({{ site.baseurl }}/images/keyboardshortcuts/ctrlhomectrlend.gif)

5. **Ctrl+ARROW** keys to move one word block at a time

   Holding Control and using the left and right arrows, moves the cursor to the beginning of the next word.
   With `Get-NetIPAddress` the words are separated with a dash/hyphen ( `-` ).

   ![_config.yml]({{ site.baseurl }}/images/keyboardshortcuts/ctrlarrow.gif)

6. **Ctrl+DEL** and **Ctrl+BACKSPACE** to delete whole word blocks at a time

   Similarly, if you wish to remove entire word blocks, hold Ctrl and use BACKSPACE to clear words to the left, and hold Ctrl and press DEL to remove word blocks to the right of the cursor.

   ![_config.yml]({{ site.baseurl }}/images/keyboardshortcuts/ctrlbackspace.gif)

7. **Ctrl+SPACE** to bring up a menu of parameters

   Last, but not least, is my personal favorite. That is the Ctrl+SPACE combination to bring up a menu of available options. These are all the parameters that I have available for the `Get-NetIPAddress` command. It even shows you the type of parameter in the bottom left corner. In this case, `-IPAddress` takes an array of strings. To navigate, use the UP/DOWN/LEFT/RIGHT arrows to highlight the desired selection, and hit ENTER to make the selection. To exit the menu without making a selection, simply press the BACKSPACE key.

   ![_config.yml]({{ site.baseurl }}/images/keyboardshortcuts/ctrlspace.gif)
