# Setup git and github account

## Github

From wiki:
> GitHub is a developer platform that allows developers to create, store, manage and share their code.
It uses Git software, providing the distributed version control of Git plus access control,
bug tracking, software feature requests, task management, continuous integration, and wikis for every project 

If you code just for yourself, you might not need such a tool, but in case you work in a group you want to share
your code and work together. Also, you might want to have your code on multiple devices. In such case github allows
easy synchronization of your codebase.

### Task

Go to [github.com](https://github.com) and create a personal account.

## Git

Git is a version control system(VCS). It helps teams of developers work on shared projects by enabling them to work
simultaneously on the same code. Imagine what you would have to do to work together on the same code if you didn't
have a VCS. It's hard event to think about how it can be implemented let alone do it. 

You don't need to fully understand what you can do with git. You'll understand it as we move forward.

### Task

Install git and add ssh access to your github account(more on what is ssh and why we need it later).

- Install git
  - Linux/macOS: use your package manager to install git
  - Windows: [go here](https://git-scm.com/download/win), download and install git. Choose default options 
while installing it.

- Add ssh access to your github account
  - Go to your user's home directory
  - Check if there's a folder `.ssh` and if there are any files like `id_rsa` and `id_rsa.pub` or similar 
inside the folder.
  - If there aren't, then generate an ssh key:
    - Run `ssh-keygen` in your terminal
    - You will be prompted to choose the name of the generated key. Press `enter` if you want the default name(`id_rsa`)
    - You will be prompted to choose a password for the key. You may live it blank by pressing `enter`
    - Now the key should be generated. Verify if the keys are there like in the second step
  - Go to [github.com](https://github.com) > click on profile icon in the top right corner >
Settings > SSH and GPG keys > New SSH Key
  - Choose a title (usually you want it to identify the device you give access to e.g. `home-desktop`)
  - Now go to `.ssh` folder, find the public key(has extension `.pub`), copy the contents of the file and paste it
in the key text field
  - Press `Add SSH key`

- Verify the access
  - Go to [github.com](https://github.com) 
  - Press `+` next to your profile icon
  - Choose `New repository`
  - Give it a name like `my-first-repo`
  - Choose `Add a README file` checkbox
  - Press `Create repository`. 
  > Congrats! :tada: You've created your first repository. This is where the shared code is stored
  - Press the green `Code` button. Choose `SSH`, and copy the provided line.
  - Now open terminal in the folder you'd like to store your projects in
  - Run `git clone <the line you copied>`.
  - If everything is done right you should be able to see a new folder names the same as the github repository
  > Congrats! :tada: Now you have a local copy of your repo that's attached to the remote repository on github.