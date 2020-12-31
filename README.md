# How to setup SSH and ZSH on DigitalOcean Droplet Server

## SSH Setup

1. Update the SSH config to allow external connections
    - Open the virtual console from within your Digital Ocean account
    - When prompted enter username `root` and your password (this can be reset from the Access tab on the server dashboard)
    - Edit the ssh config file: `vim /etc/ssh/sshd_config`
    - Update the last line from `PasswordAuthentication no` to `PasswordAuthentication yes` then write and exit `:wq`
    - Create a ssh directory and authorized_keys file `mkdir ~/.ssh && touch ~/.ssh/authorized_keys`
    - Restart the ssh service `sudo service sshd restart`
    - Terminate connection `exit`

2. Setup SSH key for root user and login
    - On your local machine, run `ssh-keygen -f ~/.ssh/digital_ocean_rsa`
    - Copy the public key to the server `ssh-copy-id -i ~/.ssh/digital_ocean_rsa.pub root@droplet.ip` (substitute your server IP address for `droplet.ip`)
    - This will prompt you for `root` user's password on the server.
    - After typing in the password, the contents of your `~/.ssh/digital_ocean_rsa.pub` key will be appended to the end of the user account’s `~/.ssh/authorized_keys` file.
    - Now login to the server `ssh -i ~/.ssh/digital_ocean_rsa.pub root@droplet.ip`

3. Add a non-root user (while logged in as root user)
    - Create home and ssh directories: `mkdir -p /home/user/.ssh`
    - Create authorized key file for the new user `touch /home/user/.ssh/authorized_keys`
    - Create the new user and assign the home directory to it `useradd -d /home/user user`
    - Add User to sudo group `usermod -aG user user`
    - Assign ownerships `chown -R user:user /home/user
    - Update permissions for authorization files `chmod 700 /home/user/.ssh && chmod 644 /home/user/.ssh/authorized_keys`
    - Set up password for new user: `passwd user` (follow the prompts)
    - Verify the new user exists by switching to the user `su user` 
    - Check the new home directory is set properly `echo $HOME` (expected output is `/home/user`) 
    - Switch to root user: `exit`

4. Set up SSH for new user
    - Still logged into server as `root`, copy the `authorized_keys` file for root to the new ssh directory for new user `cp ~/.ssh/authorized_keys ~/home/.ssh/`
    - Restart the ssh service `sudo service sshd restart`
    - Terminate the connection `exit`
    - Login to the server with new user `ssh -i ~/.ssh/digital_ocean_rsa.pub user@droplet.ip`
    - Verify current directory `pwd` (expected `/home/user`)
    - Verify user `whoami` (expected `user`)
    - Terminate connection `exit`

5. Remove root access 
    - Open the virtual console from within your Digital Ocean account
    - If prompted, enter username `root` and your password (this can be reset from the Access tab on the server dashboard)
    - Edit the ssh config file: `vim /etc/ssh/sshd_config`
    - Update the line `PermitRootLogin yes` to `PermitRootLogin no` then save and exit `:wq`
    - Restart the ssh service `sudo service sshd restart`
    - Terminate connection `exit`

That's it! 

Return to your local terminal and login via SSH `ssh -i ~/.ssh/digital_ocean_rsa.pub user@droplet.ip`

## ZSH Setup

1. Login to the server with your non-root user
2. Run `sudo apt update && sudo apt install zsh` (zsh may already be installed)
3. Make it your default shell `chsh -s $(which zsh)`
4. Install powerline fonts `sudo apt install fonts-powerline`
4. Install the theme `git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k`
5. Source the theme in your zshrc file: `echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>! ~/.zshrc`
6. Source the zshrc file `source ~/.zshrc`
7. Configure the theme (this may happen automatically once you restart zsh, or you can trigger manually by running `p10k configure`) - follow the prompts and make sure you allow it to update the `zshrc` file at the end.

That's it! You can install further plugins if you wish such as auto-completion.