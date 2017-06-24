---
title: Trying to make Custom Hotkeys in Matlab
---

Next to changing the interface to look like something from a hacker film, a key feature that greatly improves workflow in any code editor is user-defined hotkeys.
As anyone who regularily uses more than one code editor knows, having a different set of hotkeys for each environment is just an annoying and unnecessary hassle.

I regularly use three different editors for university and personal projects: RStudio, Matlab, and Visual Studio Code. Both RStudio and VSC have a nice, large set of possible hotkeys, which can easily be bound to different key combinations. To be fair, Matlab does also allow reassigning key bindings ([This](http://hoeckerson.de/notes/2016/07/keyboard-shortcuts-in-matlab-2016a-windows-7-german/) is the most comprehensive list I've found outside the actual program). Unfortunately the options are extremely limited and sometimes just not very well thought out. For example why, dear Matlab, do I need seperate keybindings for commenting and uncommenting a line instead of one toggle switch? In what world would it be useul to comment out an already-a-comment?

<!--The hotkey that I was missing most of all was moving a line up or down (typically Ctrl+Shift+UpArrow/DownArrow).-->
The hotkey that I was missing most of all was moving a line up or down (typically <kbd>ctrl</kbd> + <kbd>shift</kbd> + <kbd>Up</kbd> Ctrl+Shift+UpArrow/DownArrow).

![Move line in Action](/resources/images/blog1/movelineinaction.gif "Move line")

Effectively, every time I return to Matlab to program an experiment I feel like someone has tied the laces of my shoes together. So recently I tried to research the problem a little bit more and was surprised that not more people were complaining about this. After some experimenting, I found two promising hacks.

## What Matlab Calls "Shortcuts"
Firstly, Matlab does have an native mechanism to perform custom actions at a keypress: their so-called shortcuts.
I very much get the impression that they are not designed to be used for something as banal as moving lines up and down. I assume they are meant as a tool to quickly access frequently used, self-defined functions, but you *can* use them to move a line. 

Matlab shortcuts live in the top right corner of the interface. Adding a shortccut makes a little icon appear which can be accessed by clicking or by pressing alt and then the number that appears next to the icon.
So, to be clear, you can make it do what you want but only use it as a hotkey with the alt+number binding. So, I wasn't entirely convinced by this, but heres how to do it:

Click "add shortcut" in the upper right corner of the interface.

![New Shortcut](/resources/images/blog1/s1m.jpg "New Shortcut"){:class="shadow"}

Put in a title and the script of what you want the shortcut to do. 

![New Shortcut](/resources/images/blog1/s2s.jpg "New Shortcut"){:class="shadow"}

In my case I wanted to move the lines up and down in the editor. I found a a script for that on [github](https://github.com/m-pilia/matlab-move-line/blob/master/matlab-move-line.m).
Disclaimer: this code only works in 2016b or later because Matlab has changed how it treats strings (This is awesome by the way, strings in Matlab were always annoying to deal with).

```matlab
% Move Line
% direction: +1 to move down, -1 to move up
d = -1;
% get text and cursor position
currentEditor = matlab.desktop.editor.getActive;
selection = currentEditor.Selection;
lines = splitlines(currentEditor.Text);
line = selection(1);
% boundary check
if line + d < 1 || line + d > length(lines)
    return;
end
% swap lines
tmp = lines(line);
lines(line) = lines(line + d);
lines(line + d) = tmp;
% update text and cursor position in the editor
currentEditor.Text = char(strjoin(lines, '\n'));
selection = selection + [d 0 d 0];
currentEditor.Selection = selection;
clear currentEditor selection;
```

So I added one shortcut for the line up version with `d=+1` and a second for the line down version with `d=-1`. 

While I was at it I also added a useful little shortcut to copy a line from the editor into the terminal and execute it. RStudio features this prominently and it can be useful when debugging and when showing other people your code line by line. 
I found this code on the [Matlab forum](https://de.mathworks.com/matlabcentral/answers/132119-keyboard-shortcut-to-evaluate-current-line)

```matlab
% Shortcut summary goes here
currentEditor = matlab.desktop.editor.getActive;
originalSelection = currentEditor.Selection;
assert(originalSelection(1)==originalSelection(3));%Check that multiple lines are not selected
currentEditor.Selection = [originalSelection(1) 1 originalSelection(1) Inf];%Select the whole line
eval(currentEditor.SelectedText);%Run the whole line
clear currentEditor originalSelection
```

I also went all-out and added little icons for each shortcut. I used the [Font Awesome](http://fontawesome.io/) symbols and just turned them into jpgs, which is the only format Matlab accepts. 

![Line Up](/resources/images/blog1/up2.jpg "Line Up"){:height="50px" class="inline"}
![Line Down](/resources/images/blog1/down2.jpg "Line Down"){:height="50px" class="inline"}
![Execute Line](/resources/images/blog1/magic2.jpg "Execute Line"){:height="50px" class="inline"}

So now the top corner of my Matlab looks a bit nicer.

![Nicer Icons](/resources/images/blog1/s3s.jpg "Nicer Icons"){:class="shadow"}

Pressing alt should now make little numbers show up next to the shortcuts. So <kbd>alt</kbd> + <kbd>1</kbd> will move a line up, <kbd>alt</kbd> + <kbd>2</kbd> down, and <kbd>alt</kbd> + <kbd>3</kbd> will execute the line.

![Nicer Icons](/resources/images/blog1/s4small.jpg "Nicer Icons"){:class="shadow"}

This is a nice-ish sort-of-solution, but it's still somewhat annoying that I can't just do this with any key combination I want.

## Autohotkey
This is the excuse I've been looking for to have a look at the Autohotkey scripting language for windows. I won't go into detail here about how it works as there are plenty of tutorials for autohotkey out there already and it's relatively simple to use. If you haven't heard of it: definitely check it out. The idea is that you can write a script that will move the mouse and execute clicks and keyboard input. Tell it exactly you would perform your action in terms of clicks and keys and it will do it, really fast and at the press of a key. It's made for setting up custom hotkeys. I thought this would work really well. Unfortunately Autohotkey is windows only, but I'm sure this kind of thing exists for other operating systems.

So, I tried writing the simplest possible move line in autohotkey. I mark the line the cursor is in, copy it, move the cursor up or down, and paste the line back. While this worked fine in things like the Windows Editor and Libre Office, in Matlab, it did not work. It turns out Matlab registers the Keypresses really slowly and sequentially.

The way to get it wo work was to insert a "Sleep" between every other command. Now if AHK sleeps for a second between every other keypress, the hotkey works, but it's slow. Partly for this reason I decided to map PgDown and PgUp to the hotkey, because I had the impression that less keypresses upset Matlab less. 
In the code, I've set the "Sleep"s to the smallest numer it still worked with. This may differ on other computers.

<script src="https://gist.github.com/lschwetlick/8199c5bb3648d4dfc73dc3395b44e4fe.js"></script>

I also tried to simply make Autohotkey send alt+number, triggering the Matlab Shortcut. This *kind of* worked, but I ended up going with the pure AHK version because it causes less visual noise (upon AHK sending alt sometimes the whole editor window refreshes - it just didn't look as seamless as the other version). 

## Conclusion
The solution I have settled on for my "move line" hotkey is the pure AHK script, even if it is a little bit slow and sometimes doesnt register an input. For the little "execute line" snippet, I use the matlab shortcut bound to Alt+Enter with Autohotkey.
I have also just loved using line up/down hotkey in other text editors with tasks like reordering a To Do list or when taking notes.
