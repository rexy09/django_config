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
    eg.
    ``` bash
    ssh-keygen -t rsa -b 4096 -C "fredmuju@gmail.com"
    ```
    Where

    ``4096`` is the encryption size. This is fine by default, only change it if required.

    ``email@exmaple.com`` is your email address

4. Choose file name for your ssh keys

    Running the first command will prompt you to enter file to save your ssh keys. The default file name is ``id_rsa``. If you plan on making many SSH keys, give your key a name for they specific purpose or project.

   
    Enter file in which to save the key (``USER_PATH``/.ssh/``id_rsa``):
    ```bash
    root/.ssh/id_rsa
    ```
    
5. Add generated keys to the ssh-agent

    ```bash
    eval "$(ssh-agent -s)"
    ```

    ```bash
    ssh-add ~/.ssh/id_rsa
    ```
    
    
6. Add public key to authorized keys.

    Remember to change ``id_rsa`` to the appropriate file name if you had changed it.

    ```bash
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    ```
7. Adding a new SSH key to your account
```
cat ~/.ssh/id_rsa.pub
```
8. Testing your SSH connection
```
ssh -T git@github.com
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


Managing multiple SSH keys for multiple repositories is a common scenario, especially when working with different Git hosting services (e.g., GitHub, GitLab, Bitbucket) or multiple accounts on the same service. Here's how you can set up and manage multiple SSH keys:

---

### 1. **Generate SSH Keys for Each Repository/Account**
Generate a unique SSH key for each repository or account. For example:

```bash
# For GitHub account 1
ssh-keygen -t rsa -b 4096 -C "fredmuju@gmail.com" -f ~/.ssh/id_rsa_github_account1

# For GitHub account 2
ssh-keygen -t rsa -b 4096 -C "fredmuju@gmail.com" -f ~/.ssh/id_rsa_github_account2
```
```bash
# For GitLab account
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_gitlab

# For Bitbucket account
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_bitbucket
```

- Replace `your_email@example.com` with your email.
- Replace the `-f` value with the desired filename for each key.

---

### 2. **Add SSH Keys to the SSH Agent**
Add each key to the SSH agent so you don't have to enter the passphrase every time:

```bash
# Start the SSH agent if it's not running
eval "$(ssh-agent -s)"

# Add each key
ssh-add ~/.ssh/id_rsa_github_account1
ssh-add ~/.ssh/id_rsa_gitlab
ssh-add ~/.ssh/id_rsa_bitbucket
```

---

### 3. **Update the SSH Config File**
Edit the `~/.ssh/config` file to define which key should be used for which repository or host. If the file doesn't exist, create it.

```bash
nano ~/.ssh/config
```

Add the following configuration for each key:

```plaintext
# GitHub Account 1
Host github.com-account1
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github_account1

# GitHub Account 2
Host github.com-account2
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github_account2

# GitLab Account
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_rsa_gitlab

# Bitbucket Account
Host bitbucket.org
    HostName bitbucket.org
    User git
    IdentityFile ~/.ssh/id_rsa_bitbucket
```

- Replace `github.com-account1` with a unique identifier for the GitHub account.
- Use the appropriate `HostName` and `IdentityFile` for each key.

---

### 4. **Add SSH Keys to Git Hosting Services**
Add the public keys to the respective Git hosting services:

- **GitHub**: Go to **Settings > SSH and GPG keys > New SSH key**.
- **GitLab**: Go to **User Settings > SSH Keys**.
- **Bitbucket**: Go to **Personal Settings > SSH keys**.

Copy the public key (e.g., `~/.ssh/id_rsa_github_account1.pub`) and paste it into the appropriate service.

```bash
cat ~/.ssh/id_rsa_github_account1.pub
```
```bash
cat ~/.ssh/id_rsa_github_account2.pub
```
---

### 5. **Clone Repositories Using the Correct SSH Key**
When cloning a repository, use the `Host` value defined in the `~/.ssh/config` file. For example:

```bash
# For GitHub Account 1
git clone git@github.com-account1:username/repository.git

# For GitLab
git clone git@gitlab.com:username/repository.git

# For Bitbucket
git clone git@bitbucket.org:username/repository.git
```
---

### 6. **Update Existing Repositories**
If you already have repositories cloned, update the remote URL to use the correct `Host` value:

```bash
git remote set-url origin git@github.com-account1:username/repository.git
```
---

### 7. **Verify SSH Connections**
Test the SSH connection for each host:

```bash
ssh -T git@github.com-account1
ssh -T git@gitlab.com
ssh -T git@bitbucket.org
```
You should see a success message for each service.
