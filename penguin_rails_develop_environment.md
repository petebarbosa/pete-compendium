# Clean Debian development environment

This is how I usually start a new environment for development. This setup was inspired by the one Le Wagon provides all it's students. I removed some parts, since it's used in classes, but won't be used in real life. And if you want an amazing bootcamp, please do check out [Le Wagon](https://www.lewagon.com/).

## Zsh & Git

Let's install them, along with other useful tools:
- Open an **Ubuntu terminal**
- Copy and paste the following commands:

```bash
sudo apt update
```

```bash
sudo apt install -y curl git imagemagick jq unzip vim zsh tree
```

These commands will ask for your password: type it in.

### GitHub CLI installation

Let's now install [GitHub official CLI](https://cli.github.com). It's a software used to interact with your GitHub account via the command line.

In your terminal, copy-paste the following commands and type in your password if asked:

```bash
sudo apt remove -y gitsome # gh command can conflict with gitsome if already installed
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
```

```bash
sudo apt update
```

```bash
sudo apt install -y gh
```

To check that `gh` has been successfully installed on your machine, you can run:

```bash
gh --version
```

## Oh-my-zsh

Let's install the `zsh` plugin [Oh My Zsh](https://ohmyz.sh/).

In a terminal execute the following command:

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## GitHub CLI

First in order to **login**, copy-paste the following command in your terminal:

:warning: **DO NOT edit the `email`**

```bash
gh auth login -s 'user:email' -w
```

gh will ask you few questions:

`What is your preferred protocol for Git operations?` With the arrows, choose `SSH`.
`Generate a new SSH key to add to your GitHub account?` Press `Enter` to ask gh to generate the SSH keys for you.

If you already have SSH keys, you will see instead `Upload your SSH public key to your GitHub account?`.

`Enter a passphrase for your new SSH key (Optional)`. Type something you want and that you'll remember. It's a password to protect your private key stored on your hard drive.

`Title for your SSH key`. You can leave it at the proposed "GitHub CLI".

You will then get the following output:

```bash
! First copy your one-time code: 0EF9-D015
- Press Enter to open github.com in your browser...
```

Select and copy the code (`0EF9-D015` in the example), then press `Enter`.

Your browser will open and ask you to authorize GitHub CLI to use your GitHub account. Accept and wait a bit.

Come back to the terminal, press `Enter` again, and that's it.

To check that you are properly connected, type:

```bash
gh auth status
```

## git installer

Run the `git` installer:

```bash
cd ~/code/$GITHUB_USERNAME/dotfiles && zsh git_setup.sh
```

:point_up: This will **prompt** you for your name (`FirstName LastName`) and your email.

Please now **reset** your terminal by running:

```bash
exec zsh
```

## rbenv

Let's install [`rbenv`](https://github.com/sstephenson/rbenv), a software to install and manage `ruby` environments.

First, we need to clean up any previous Ruby installation you might have:

```bash
rvm implode && sudo rm -rf ~/.rvm
# If you got "zsh: command not found: rvm", carry on.
# It means `rvm` is not on your computer, that's what we want!
rm -rf ~/.rbenv
```

Then in the terminal, run:

```bash
sudo apt install -y build-essential tklib zlib1g-dev libssl-dev libffi-dev libxml2 libxml2-dev libxslt1-dev libreadline-dev libyaml-dev
```

```bash
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
```

```bash
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

```bash
exec zsh
```

## Ruby

### Installation

Now, you are ready to install the latest [ruby](https://www.ruby-lang.org/en/) version and set it as the default version.

Run this command to check what is the latest version:

```sh 
rbenv install --version
```

This will show you all stable versions. Run this command with the most recent one, which for now is:

```bash
rbenv install 3.3.5
```

Once the ruby installation is done, run this command to tell the system
to use the 3.3.5 version by default.

```bash
rbenv global 3.3.5
```

**Reset** your terminal and check your Ruby version:

```bash
exec zsh
```

Then run:

```bash
ruby -v
```

## Installing some gems


First, we'll update `bundler`, which is what lets us install gems:

```bash
gem update bundler
```

In your terminal, copy-paste the following command:

```bash
gem install colored faker http pry-byebug rake rails rest-client rspec rubocop-performance sqlite activerecord
```

:warning: **NEVER** install a gem with `sudo gem install`! Even if you stumble upon a Stackoverflow answer (or the terminal) telling you to do so.


## Node.js

[Node.js](https://nodejs.org/en/) is a JavaScript runtime to execute JavaScript code in the terminal. Let's install it with [nvm](https://github.com/nvm-sh/nvm), a version manager for Node.js.

In a terminal, execute the following commands:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | zsh
```

```bash
exec zsh
```

Then check installation:

```bash
nvm -v
```

Now let's install node's latest version:

```bash
nvm install node
```

When the installation is finished, run:

```bash
node -v
```

After installing, clear cache:

```bash
nvm cache clear
```

## yarn

[`yarn`](https://yarnpkg.com/) is a package manager to install JavaScript libraries. Let's install it:

In a terminal, run the following commands:

```bash
corepack enable
yarn set version stable
```

⚠️ If you see any error messages, try running `npm install -g corepack` and then run the commands above again.

```bash
exec zsh
```

Then run the following command:

```bash
yarn -v
```

## SQLite

In a few weeks, we'll talk about databases and SQL. [SQLite](https://sqlite.org/index.html) is a database engine used to execute SQL queries on single-file databases. Let's install it:

In a terminal, execute the following commands:

```bash
sudo apt-get install sqlite3 libsqlite3-dev pkg-config
```

Then run the following command:

```bash
sqlite3 -version
```

## PostgreSQL

Sometimes, SQLite is not enough and we will need a more advanced tool called [PostgreSQL](https://www.postgresql.org/), an open-source robust and production-ready database system.

Let's install it now.

Run the following commands:

```bash
sudo apt install -y postgresql postgresql-contrib libpq-dev build-essential
```

```bash
sudo -u postgres psql --command "CREATE ROLE \"`whoami`\" LOGIN createdb superuser;"
```

## Ubuntu settings

### Install video codec H264

```bash
sudo apt install libavcodec-extra -y
```

### Install useful terminal tools

`ncdu` is a text-based interface disk utility.

`htop` is an interactive process viewer.

```bash
sudo apt install ncdu htop
```

### Ubuntu inotify

Ubuntu is always tracking changes in your folders and to do this it uses inotify.
By default the Ubuntu limit is set to 8192 files monitored.

As programming involves a lot of files, we need to raise this limit.
In your terminal run:

```bash
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

