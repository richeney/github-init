# GitHub Init

## Introduction

I use GitHub repos all the time, in combination with Visual Studio Code and WSL2. I wanted a simple command to speed up the creation of a GirtHub repo linked as the remote origin for the local repos I was creating without having to dive out of vscode and its integrated terminal.

So I scrubbed together a little function in my ~/.bashrc file so I could call `github init`.

## Example usage

```bash
mkdir -p /git/example-repo
cd /git/example-repo
echo "# README" > README.md
github init
```

You can check all is working as expected using `git remote -v` and `git push`.

## Pre-reqs

You need a GitHub account. I have assumed that you have [configured 2FA](https://help.github.com/en/github/authenticating-to-github/configuring-two-factor-authentication) and that you have generated a [personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line).

I have also assumed that you are working on your own laptop and that no-one else is able to see your ~/.bashrc file.

## Modifying ~/.bashrc

1. Edit your ~/.bashrc file and add the following:

    ```bash
    # Make it easier to create a git repo

    export GITHUB_TOKEN=987654321eadd959e9f0c7cdbc9d0edfa7ffb3
    github() {
      [ "$1" == "init" ] || { echo "Usage: github init"; return 1; }
      [ -z "$GITHUB_TOKEN" ] && { echo "Error: GITHUB_TOKEN is undefined"; return 1; }

      repo=$(basename $(pwd))
      echo "* text=auto eol=lf" > ./.gitattributes
      git init
      git add *
      git add .gitattributes
      git commit -m "Initial commit"
      curl  https://api.github.com/user/repos?access_token=$GITHUB_TOKEN -d {\"name\":\"$repo\"}
      git remote add origin https://github.com/richeney/$repo.git
      git push origin master

      printf "\nCreated https://github.com/richeney/$repo\n"
      return 0
    }
    ```

1. Replace the value of the GITHUB_TOKEN to your personal access token
1. Replace `richeney` with your GitHub userid
1. Save the profile
1. Reread the profile using `source ~/.bashrc`

> For security reasons the example token is not valid!!

## Git Credentials Store (optional)

If you are on a shared system then you have probably configured your .gitconfig file to use a credential store using the following commands:

```bash
git config --global credential.user richeney
git config --global credential.helper store
```

(Where richeney is the GitHub userid.)

If you `cat ~/.git-credentials` after you've authenticated then your file should look similar to this:

```text
https://richeney:987654321eadd959e9f0c7cdbc9d0edfa7ffb3@github.com
```

If so then you can modify the line in your profile, e.g.:

```bash
export GITHUB_TOKEN==$(sed -E 's!^https://richeney:(.*)@github.com$!\1!' ~/.git-credentials)
```

The benefit of making that change is that it is a little more secure and it will continue to work if you refresh your personal access token.
