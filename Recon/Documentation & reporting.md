
## Logging
When we are working in the terminal, we can save our work by using `tmux` with logging enabled, this way we can focus on exploitation and in the breaks we can view the logs and start writing our findings.

clone the [Tmux Plugin Manager](https://github.com/tmux-plugins/tpm)
```shell
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

create the config file
```shell
nvim ~/.tmux.conf
```

the file should contain the following
```shell
# List of plugins

set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-logging'

# Initialize TMUX plugin manager (keep at bottom)
run '~/.tmux/plugins/tpm/tpm'

# Adjust the number of lines in tmux buffer
set -g history-limit 50000
```

after that we can start `tmux`
```shell
tmux new -s setup
```

to install the plugin press `[CTRL] + [B]` then `[SHIFT] + [I]`.
to start logging the current tmux session `[Ctrl] + [B]` followed by `[Shift] + [P]`. Use the same combination to stop logging.

If we forget to enable Tmux logging and are deep into a project, we can perform retroactive logging by typing `[Ctrl] + [B]` and then hitting `[Alt] + [Shift] + [P]`

#### Screenshot
If we have multiple panes open, trying to copy output from one is messy, we can take a screenshot and output it to a file with `[CTRL] + [B]` then `[ALT] + [P]`. BE SURE THE CURSER IS ON THE PANE YOU WANT TO SCREENSHOT

### tmux plugins
- [tmux-sessionist](https://github.com/tmux-plugins/tmux-sessionist) - Gives us the ability to manipulate Tmux sessions from within a session: switching to another session, creating a new named session, killing a session without detaching Tmux, promote the current pane to a new session, and more.
- [tmux-pain-control](https://github.com/tmux-plugins/tmux-pain-control) - A plugin for controlling panes and providing more intuitive key bindings for moving around, resizing, and splitting panes.
- [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) - This extremely handy plugin allows us to restore our Tmux environment after our host restarts. Some features include restoring all sessions, windows, panes, and their order, restoring running programs in a pane, restoring Vim sessions, and more.

## Tracking Artifacts
If we create accounts or modify system settings, it should be evident that we need to track those things in case we cannot revert them once the assessment is complete. Some examples of this include:

- IP address of the host(s)/hostname(s) where the change was made
- Timestamp of the change
- Description of the change
- Location on the host(s) where the change was made
- Name of the application or service that was tampered with
- Name of the account (if you created one) and perhaps the password in case you are required to surrender it


## Tips for Reporting
- Aim to tell a story with your report. Why does it matter that you could perform Kerberoasting and crack a hash? What was the impact of default creds on X application?
- Write as you go. Don't leave reporting until the end. Your report does not need to be perfect as you test but documenting as much as you can as clearly as you can during testing will help you be as comprehensive as possible and not miss things or cut corners while rushing on the last day of the testing window.
- Stay organized. Keep things in chronological order, so working with your notes is easier. Make your notes clear and easy to navigate, so they provide value and don't cause you extra work.
- Show as much evidence as possible while not being overly verbose. Show enough screenshots/command output to clearly demonstrate and reproduce issues but do not add loads of extra screenshots or unnecessary command output that will clutter up the report.
- Clearly show what is being presented in screenshots. Use a tool such as [Greenshot](https://getgreenshot.org/) to add arrows/colored boxes to screenshots and add explanations under the screenshot if needed. A screenshot is useless if your audience has to guess what you're trying to show with it.
- Redact sensitive data wherever possible. This includes cleartext passwords, password hashes, other secrets, and any data that could be deemed sensitive to our clients. Reports may be sent around a company and even to third parties, so we want to ensure we've done our due diligence not to include any data in the report that could be misused. A tool such as `Greenshot` can be used to obfuscate parts of a screenshot (using solid shapes and not blurring!).
- Redact tool output wherever possible to remove elements that non-hackers may construe as unprofessional (i.e., `(Pwn3d!)` from CrackMapExec output). In CME's case, you can change that value in your config file to print something else to the screen, so you don't have to change it in your report every time. Other tools may have similar customization.
- Check your Hashcat output to ensure that none of the candidate passwords is anything crude. Many wordlists will have words that can be considered crude/offensive, and if any of these are present in the Hashcat output, change them to something innocuous. You may be thinking, "they said never to alter command output." The two examples above are some of the few times it is OK. Generally, if we are modifying something that can be construed as offensive or unprofessional but not changing the overall representation of the finding evidence, then we are OK, but take this on a case-by-case basis and raise issues like this to a manager or team lead if in doubt.
- Check grammar, spelling, and formatting, ensure font and font sizes are consistent and spell out acronyms the first time you use them in a report.
- Make sure screenshots are clear and do not capture extra parts of the screen that bloat their size. If your report is difficult to interpret due to poor formatting or the grammar and spelling are a mess, it will detract from the technical results of the assessment. Consider a tool such as Grammarly or LanguageTool (but be aware these tools may ship some of your data to the cloud to "learn"), which is much more powerful than Microsoft Word's built-in spelling and grammar check.
- Use raw command output where possible, but when you need to screenshot a console, make sure it's not transparent and showing your background/other tools (this looks terrible). The console should be solid black with a reasonable theme (black background, white or green text, not some crazy multi-colored theme that will give the reader a headache). Your client may print the report, so you may want to consider a light background with dark text, so you don't demolish their printer cartridge.
- Keep your hostname and username professional. Don't show screenshots with a prompt like `azzkicker@clientsmasher`.
- Establish a QA process. Your report should go through at least one, but preferably two rounds of QA (two reviewers besides yourself). We should never review our own work (wherever possible) and want to put together the best possible deliverable, so pay attention to the QA process. At a minimum, if you're independent, you should sleep on it for a night and review it again. Stepping away from the report for a while can sometimes help you see things you overlook after staring at it for a long time.
- Establish a style guide and stick to it, so everyone on your team follows a similar format and reports look consistent across all assessments.
- Use autosave with your notetaking tool and MS Word. You don't want to lose hours of work because a program crashes. Also, backup your notes and other data as you go, and don't store everything on a single VM. VMs can fail, so you should move evidence to a secondary location as you go. This is a task that can and should be automated.
- Script and automate wherever possible. This will ensure your work is consistent across all assessments you perform, and you don't waste time on tasks repeated on every assessment.