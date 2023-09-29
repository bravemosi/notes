# github authentication

first it's better to know about:

## Public key/private key cryptography

also known as asymmetric cryptography, is a cryptographic method that uses a pair of keys to secure data and enable secure communication between parties.

**Public Key:** The public key is intended to be shared openly and widely. It is used to encrypt data or verify digital signatures. The public key can be freely distributed and is associated with a specific individual or entity.

**Private Key:** The private key is kept secret and known only to the owner. It is used to decrypt data or create digital signatures. The private key must be securely stored and should never be shared with anyone.

Here's how public key/private key cryptography works:

- Encryption: If someone wants to send an encrypted message to a person, they use the recipient's public key to encrypt the message. Once encrypted, only the recipient's private key can decrypt and read the message.

- Digital Signatures: If someone wants to digitally sign a document or message to prove its authenticity and integrity, they use their private key to create a unique digital signature. The recipient can then use the sender's public key to verify the signature and ensure the message hasn't been tampered with.

## GCM,GPG and SSH

GCM (Git Credential Manager) is a cross-platform tool that provides a secure way to store and manage **Git credentials(username and email)**. It is designed to simplify the process of **authenticating** with remote Git repositories.we use it to authenticate over https.

SSH (Secure Shell) and GPG (GNU Privacy Guard) are both cryptographic tools used for different purposes(both use public key cryptography).

**1. Purpose:**

- SSH: SSH is primarily used for secure remote access, file transfer, and secure command execution on remote systems.
- GPG: GPG is used for encryption, digital signatures, and secure communication, primarily through email encryption and file encryption.

It's important to note that while SSH and GPG both use public key cryptography, they serve different purposes. SSH focuses on secure remote access and file transfer, while GPG is more oriented towards encryption, digital signatures, and secure email communication.

> :bulb:**Note:**
>
> **use gcm for authentication and gpg for signing commits and tags over https.**
>
> **or use ssh for authentication and signing.**

when you are configuring ssh,gpg, ..., it may ask you to give it a passphrase and then everytime you have to give it the passphrase when is needed:

### passphrase

A passphrase is a sequence of words or a sentence used as an added layer of security to protect sensitive information, such as an encryption key or a private key.Passphrases are typically longer than traditional passwords, consisting of multiple words, random characters, or a combination of both.Passphrases should be memorable to the user but difficult for others to guess

a passphrase exmaple: `whispering12Sunflower!`

### authentication and signing with ssh

#### use SSH to connect to GitHub

**1. Generate SSH Key Pair(public key/private key):**

- Run the command `ssh-keygen -t ed25519 -C "your_email@example.com"`. Replace the email address with your GitHub-associated email.
  this will add a `.ssh` folder in root path, and inside of it two files: `id_ed25519` file which contains private key and `id_ed25519.pub` which contains public key. you can read both files with a text editor file and see keys.
- Optionally, you can set a passphrase to provide an additional layer of security.

**2. Add the Public Key to GitHub:**

- Copy the public key inside `~/.ssh/id_ed25519.pub`.
- Visit your GitHub account settings and navigate to the "SSH and GPG keys" section.
- Click on "New SSH key" and select "Authentication Key" type and paste the public key into the designated field and save.

**3. Test the SSH Connection:**

- Run the command `ssh -T git@github.com` in the terminal to test the SSH connection to GitHub.
- You should receive a message like `Hi username! You've successfully authenticated...`, indicating a successful connection.

**4. Configure Git to Use SSH:**

- If you have an existing Git repository, navigate to its root directory in the terminal.
- Run the command `git remote set-url origin git@github.com:username/repository.git`. Replace `username` with your GitHub username and `repository` with the name of your repository.
- This configures Git to use SSH for remote operations with your GitHub repository.Git commands like `git clone`, `git push`, and `git pull` will now use SSH for authentication and data transfer.(it will ask for pathphrase if you set one)

**5. Add Private Key to the SSH Agent**:
The ssh-agent is a helper program that keeps track of users' identity keys and their passphrases. The agent can then use the keys to log into other servers without having the user type in a password or passphrase again.

To add a private key to the SSH agent:

```shell
# to start the SSH agent or check if it is already running.
`eval $(ssh-agent -s)`
# will start the agent and display the agent's process ID (PID)

# add the private key to the SSH agent
`ssh-add ~/.ssh/id_ed25519`
# If the private key is password-protected, you will be prompted to enter the passphrase.

`ssh-add -l` # list the keys currently added to the SSH agent
```

The SSH agent will now handle the authentication process using the added private key when you attempt SSH connections or interact with SSH-enabled services, such as GitHub. This eliminates the need to enter the passphrase for each SSH operation, as the agent securely stores the decrypted private key in memory.

#### use ssh to sign commits and tags

we can use the **_same ssh key_** we generated previously to sign commits and tags:

1. configure git:

   ```shell
   #set ssh as gpg format
   git config --global gpg.format ssh

   #set ssh public key as signingkey
   git config --global user.signingkey ~/.ssh/id_ed25519.pub
   ```

2. add signingkey to github:

   - Copy the public key inside `~/.ssh/id_ed25519.pub`.
   - Visit your GitHub account settings and navigate to the "SSH and GPG keys" section.
   - Click on "New SSH key" and select "Signing Key" type and paste the public key into the designated field and save.

3. sign commits and tags:

   ```shell
   git commit -S -m "commit message" #uppercase s
   git tag -s v1.0 #lowercase s
   ```

   or if you want to sign all commits and tags:

   ```shell
   git config --global commit.gpgsign true
   git config --global tag.gpgsign true
   ```

### install gcm and store credentials

[gcm github](https://github.com/git-ecosystem/git-credential-manager)

1. installed gcm app on your system

2. configure it:

   ```shell
   git-credential-manager configure

   #or

   git config --global credential.helper <path to gcm app>
   #in linux: /usr/local/bin/git-credential-manager
   ```

3. we need a credential store as well:

- on linux and mac: we use gpg/pass
  (pass is a password manager that has a command-line interface, and uses gpg for encryption and decryption of stored passwords.)

select credential store:

```shell
export GCM_CREDENTIAL_STORE=gpg
# or
git config --global credential.credentialStore gpg
```

Before you can use this credential store, it must be initialized by the `pass`
utility, which in-turn requires a valid GPG key pair. To initialize the store,
run:

```shell
pass init <gpg-id>
```

..where `<gpg-id>` is the user ID of a GPG key pair on your system.

pass init command will initialize store at '~/.password-store'

- on Windows:
  we use default store in windows:

  ```shell
  SET GCM_CREDENTIAL_STORE="wincredman"
  #or
  git config --global credential.credentialStore wincredman
  ```

  for different ways to handle Credential stores [see](https://github.com/git-ecosystem/git-credential-manager/blob/main/docs/credstores.md)

<!-- complete this later -->

### signing with gpg

- make sure gpg is installed on your system
- generate gpg key(public key/secret key):

```shell
#will create a .gnupg folder in root and sets public and private key in it
gpg --full-generate-key

#answer comming questions(you can choose deafaults)
#then it will ask these important question(enter your github info):
Real name: <your github username>
Email address: <your github email>
Comment: github key #just a comment don't matter what to put in here
#asks for pathphrase

#... it generates public and private key...
```

- list secret keys:

  ```shell
  gpg --list-secret-keys --keyid-format=long

  #above command should return like this:
  /home/mosi/.gnupg/pubring.kbx
  -----------------------------
  sec   rsa3072/2R32T66A990EZ52F 2023-09-22 [SC]
        5A1C5E1FF80136ABCA6953782R32T66A990EZ52F
  uid                 [ultimate] bravemosi (github key) <bravemosi@gmail.com>
  ssb   rsa3072/0D0483E928233173 2023-09-22 [E]
  ```

- export public key from private key:

  ```shell
  gpg --armor --export 2R32T66A990EZ52F

  #it will show the public key
  -----BEGIN PGP PUBLIC KEY BLOCK-----
    # a really long key in here
  -----END PGP PUBLIC KEY BLOCK-----
  ```

- Visit your GitHub account settings and
  navigate to the "SSH and GPG keys" section.
  Click on "New GPG key" and paste the public key.

- configure git:

  ```shell
  git config --global --unset gpg.format # if anything else like ssh is set
  git config --global signinigkey 2R32T66A990EZ52F
  git config --global gpg.program <path to gpg> # for example in linux: /usr/bin/gpg
  ```

- To add your GPG key to your .bashrc startup file, run the following command:

```shell
[ -f ~/.bashrc ] && echo -e '\nexport GPG_TTY=$(tty)' >> ~/.bashrc
```

- make gpg-agent remember your passphrase
  in `.gnupg/gpg-agent.conf`(if file doesn't exist create it)
  add these lines to it:

  ```shell
  allow-preset-passphrase # make gpg remember passphrase
  default-cache-ttl 3600 # 3600 second is cache time-to-live
  #Restart the GPG agent to apply the configuration changes
  gpg-connect-agent reloadagent /bye
  ```
