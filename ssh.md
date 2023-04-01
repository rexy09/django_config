# SSH Commands

### Generate SSH keys

1. Open terminal and navigate to home directory

    ``` bash
    cd ~
    ```

2. Run the command

    ``` bash
    ssh-keygen -t rsa -b 4096 -C "email@example.com"
    ```
    Where

    ``4096`` is the encryption size. This is fine by default, only change it if required.

    ``email@exmaple.com`` is your email address

3. Choose file name for your ssh keys

    Running the first command will prompt you to enter file to save your ssh keys. The default file name is ``id_rsa``. If you plan on making many SSH keys, give your key a name for they specific purpose or project.

   
    Enter file in which to save the key (``USER_PATH``/.ssh/``id_rsa``):
    ```bash
    ~/.ssh/id_rsa
    ```
    
4. Add generated keys to the ssh-agent

    ```bash
    eval "$(ssh-agent -s)"
    ```

    ```bash
    ssh-add ~/.ssh/id_rsa
    ```
    
    
5. Add public key to authorized keys.

    Remember to change ``id_rsa`` to the appropriate file name if you had changed it.

    ```bash
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    ```


## Add local computer SSH to local

Change ``YOUR_USER_NAME`` and ``IP_ADDRESS_OF_THE_SERVER``

Also, change ``id_rsa`` to the public key name if it was changed.

``` bash
ssh-copy-id -i ~/.ssh/id_rsa.pub YOUR_USER_NAME@IP_ADDRESS_OF_THE_SERVER
```

## Multiple keys for multiple repositories

1. Navigate to ssh directory 

    ``` bash
    cd ~/.ssh
    ```

2. Create ``config`` file

    ``` bash
    touch config
    ```

3. Create aliases for each repo

    ``` ssh
    Host github.com-repo-0
        Hostname github.com
        IdentityFile=/home/user/.ssh/repo-0_deploy_key

    Host github.com-repo-1
            Hostname github.com
            IdentityFile=/home/user/.ssh/repo-1_deploy_key
    ```

    Note: change ``repo-0`` in Host to the correct github repository name.

    Change ``repo-0_deploy_key`` and ``repo-1_deploy_key`` to the correct ssh keys, and verify that the path is correct

    Save the file

4. Use the aliases to clone the repositories or update the remote origin

    ``` bash
    git clone git@github.com-repo-1:USERNAME/REPOSITORY.git
    ```

    or 

    ``` bash
    git remote set-url origin git@github.com-repo-1:USERNAME/REPOSITORY.git
    ```


