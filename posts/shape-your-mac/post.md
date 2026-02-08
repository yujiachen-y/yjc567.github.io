# 📖 Foreword

Have you ever furnished your own house? To be honest, I haven't, since I always rent. However, I do arrange things in my home. Right now, I'm lying on my sofa and writing this doc. To make it more engaging, I decided to include a picture of my current setup.

![](attachments/Pasted%20image%2020260207151738.png)

I moved to Shanghai less than two months ago, but I've already made this place feel familiar. This has allowed me to live happily and somewhat "efficiently" — meaning the room's arrangement perfectly suits my daily routine.

  

It's the same with a computer, especially a work computer. We don't own our work computers; they belong to the company. But since we spend most of our screen time on them, it's important to make sure they make us feel happy and efficient.

# 🔧 Basic Settings

## 🗂️ Dotfiles

https://github.com/yujiachen-y/dotfiles

Dotfiles can be considered the metadata of your computer. Personally, I use [a Brewfile](https://github.com/yujiachen-y/dotfiles/blob/main/macos/Brewfile) to manage my Mac's dependencies. As long as an app can be installed via Homebrew, you should add it to your Brewfile. In fact, only a few apps cannot be installed using Homebrew.

  

I also store my [.zshrc](https://github.com/yujiachen-y/dotfiles/blob/main/zsh/.zshrc) and [.vimrc](https://github.com/yujiachen-y/dotfiles/blob/main/.vimrc) files in Dotfiles, so that I don't need to configure my zsh and vim repeatedly.

  

Additionally, it's common practice to store [Mac settings](https://github.com/yujiachen-y/dotfiles/blob/main/macos/system_settings.sh) in dotfiles. However, you still need to go to System Settings to configure some options that can't be changed via command lines. BTW, every time I set up a Mac, there are often UI changes and new options in System Preferences. So, the manual setup helps me discover what changes Apple has made lol.

## 💻 Command Line Config

![](attachments/Pasted%20image%2020260207151751.png)

I use [Warp](https://www.warp.dev/) as my terminal emulator. However, I believe the differences between Terminal.app, iTerm2, and Warp are not significant. For me, Warp has 3 main advantages:

1. **Suggestions**: I find them better than [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions).
    
2. **AI Copilot**: In my opinion, all AI copilot products can be replaced by directly asking ChatGPT. (Update: Warp has agent mode too.)
    
3. **Built-in Shell Prompt**: Although it’s not as flexible or functional as [p10k](https://github.com/romkatv/powerlevel10k), I’m considering reverting to my old [.p10k.zsh](https://github.com/yujiachen-y/dotfiles/blob/main/zsh/.p10k.zsh) setup.
    

  

Have you ever checked how many lines of code are in your `.zshrc` file? What do those lines do? I checked mine, and it’s only [15 lines](https://github.com/yujiachen-y/dotfiles/blob/3a77776963e3fee4a3df70c7e29230d40c1c2419/zsh/.zshrc). Typically, a `.zshrc` file is full of `[oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)` configurations, but I only keep a few essential plugins and delete the unnecessary ones. Here are some `oh-my-zsh` plugins worth mentioning:

- **[dotenv](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/dotenv/README.md)**: Automatically loads your .env files into environment variables.
    
- **[git](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/git/README.md)**: I use commands like `gaa` (`git add --all`), `gsmsg` (`git commit --message`), and `gcn!` (`git commit --verbose --no-edit --amend`) every day.
    
    - A git tip: please use `gcn!` to avoid unnecessary commit information in PR.
    
- **[z](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/z/README.md)**: Access your most visited directories with very few keystrokes.
    

  

As mentioned earlier, I use Warp for its built-in shell prompt, which looks like this:

![](attachments/Pasted%20image%2020260207151907.png)

  

However, it’s not very flexible and cannot be used in other terminal emulators. If you use Warp’s shell prompt, you might encounter issues in VSCode or other text editors and IDEs. [Embedding Warp into VSCode is particularly challenging](https://github.com/warpdotdev/Warp/issues/257#issuecomment-1274198741).

![](attachments/Pasted%20image%2020260207151928.png)
  

I recommend using [p10k](https://github.com/romkatv/powerlevel10k) as your shell prompt. It’s highly flexible and can be used across different terminal emulators since it’s not tied to any specific one.

![](attachments/Pasted%20image%2020260207151943.png)

  

I want to mention 2 of my favorite macOS commands here: [pbpaste](https://ss64.com/mac/pbpaste.html) and [pbcopy](https://ss64.com/mac/pbcopy.html). There are already some useful examples on the manual pages.


## ⚙️ System Settings

> As mentioned earlier, some settings below can be incorporated into your dotfiles.

Implementing these settings will enhance your productivity:

- **Prevent Automatic Sleeping**
    

![](attachments/Pasted%20image%2020260207152057.png)

  

- **Three Finger Drag**
    

![](attachments/Pasted%20image%2020260207152107.png)

[How to select or drag using three fingers on your MacBook track pad](https://youtu.be/-Fy6imaiHWE "Share link")

  

- **Reduce Keyboard Delay Time**: Set your keyboard delay time to the shortest possible setting. As coders, we can't afford to waste time waiting for keyboard delays.
    

![](attachments/Pasted%20image%2020260207152238.png)
  

- **Disable Double-Space Period**: This feature can be annoying, so it's best to turn it off.
    

![](attachments/Pasted%20image%2020260207152444.png)

  

- **Swap Caps Lock and Command Keys**: The keys you use most frequently should be more accessible. Swap the Caps Lock key with the Command key.
    

![](attachments/Pasted%20image%2020260207152453.png)

  

- **Disable Press and Hold**: This setting can only be disabled via the command line. If you use [Vim](https://opusclip.larksuite.com/wiki/Uu5NwIeBFilqHRk2dhSu5MNYsUg#LlCwdefVKo7DO3xvxUquBptVs9f), you might not need to disable this setting, as pressing and holding a key is not considered good practice in Vim.
    

![](attachments/Pasted%20image%2020260207152516.png)

```Bash
defaults write -g ApplePressAndHoldEnabled -bool false
```

  

# 📱 Apps

> As I mentioned earlier, all apps listed here should be managed by your Brewfile.

## 🌟 Raycast

![](attachments/Pasted%20image%2020260207152616.png)

[Raycast is an everything store for Mac shortcuts.](https://www.raycast.com/store) Here are some shortcuts I often use:

| Clipboard History                    | Search Emoji                         | Window Management                    | Music Control                        | Kill Process                         | Search Browser Tabs                  | Reminder                             |
| ------------------------------------ | ------------------------------------ | ------------------------------------ | ------------------------------------ | ------------------------------------ | ------------------------------------ | ------------------------------------ |
| ![](attachments/Pasted%20image%2020260207152629.png) | ![](attachments/Pasted%20image%2020260207152640.png) | ![](attachments/Pasted%20image%2020260207152650.png) | ![](attachments/Pasted%20image%2020260207152704.png) | ![](attachments/Pasted%20image%2020260207152716.png) | ![](attachments/Pasted%20image%2020260207152736.png) | ![](attachments/Pasted%20image%2020260207152745.png) |

Raycast is super useful with your custom hotkeys.

![](attachments/Pasted%20image%2020260207152819.png)

## 🌟 AltTab

On a Mac, you can use Command + Tab to switch between apps. However, the built-in app switcher doesn't allow you to switch between windows within the same application, which can be inconvenient if you need to frequently switch between windows of a single app.

![](attachments/Pasted%20image%2020260207152829.png)

[AltTab](https://alt-tab-macos.netlify.app/) solves this problem by allowing you to switch between windows across different Mac desktops. This way, you don't need to use a mouse to switch between windows and desktops.

![](attachments/Pasted%20image%2020260207152836.png)


# 💾 Backup

> There are two types of people:
> 
> - Those who do backups
>     
> - Those who will do backups
>     
> 
> Any data you own that you haven’t backed up is data that could be gone at any moment, forever. Here we will cover some good backup basics and the pitfalls of some approaches.
> 
> ## **3-2-1 Rule**
> 
> The [3-2-1 rule](https://www.us-cert.gov/sites/default/files/publications/data_backup_options.pdf) is a general recommended strategy for backing up your data. It state that you should have:
> 
> - at least **3 copies** of your data
>     
> - **2** copies in **different mediums**
>     
> - **1** of the copies being **offsite**
>     
> 
> The main idea behind this recommendation is not to put all your eggs in one basket. Having 2 different devices/disks ensures that a single hardware failure doesn’t take away all your data. Similarly, if you store your only backup at home and the house burns down or gets robbed you lose everything, that’s what the offsite copy is there for. Onsite backups give you availability and speed, offsite give you the resiliency should a disaster happen.
> 
>   
> 
> From [Backups](https://missing.csail.mit.edu/2019/backups/)

# 💭 Reflecting thoughts

- Routinely review your workflow and try to improve your efficiency.
    
- [Fold knowledge into data, so program logic can be stupid and robust.](https://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2878263)
    
- [Programmer time is expensive; conserve it in preference to machine time.](https://www.catb.org/~esr/writings/taoup/html/ch01s06.html#id2878666)
    
- [We shape our tools and thereafter our tools shape us](https://mcluhangalaxy.wordpress.com/2013/04/01/we-shape-our-tools-and-thereafter-our-tools-shape-us/)

## Cite this article

APA:
Yu, J. (July 14, 2024). Shape Your Mac. Jiachen Yu. https://www.yujiachen.com/shape-your-mac/

BibTeX:
```bibtex
@online{yu2024shapeyourmac,
  author = {Yu, Jiachen},
  title = {Shape Your Mac},
  year = {2024},
  publisher = {Jiachen Yu},
  url = {https://www.yujiachen.com/shape-your-mac/},
  urldate = {2026-02-08},
}
```
