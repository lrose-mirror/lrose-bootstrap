# Install google-chrome-stable on linux

## Create repo

Create the repo file:

```
  touch /etc/yum.repos.d/google-chrome.repo
```

## populate it with the following

```
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
```

## Install chrome stable

```
  dnf install google-chrome-stable
```

