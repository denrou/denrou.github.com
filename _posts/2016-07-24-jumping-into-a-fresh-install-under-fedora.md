---
layout: post
title:  Jumping into a fresh install under fedora
date: "2016-07-24 19:05:31"
published: true
tags: [linux]
---

<img src="images/fulls/fresh_install_fedora.jpg" class="fit image">

As every operating system, a Linux based system need updates from time to time.
Period between updates depend on the distribution which is used.
For example, a *rolling release* distribution such as *Arch Linux* will be continuously updated.

My distribution is *Fedora* for different reasons : i) I like *Gnome*, the official desktop environment, ii) it is really easy to manage and iii) it has very recent versions of the softwares I use.
*Fedora* is updated every six months.
When a new release is out, we face two possibilities :

1. upgrade the system (via *fedup*)
2. re-install the system

Upgrading the system does not erase any program, configuration file or document that we have.
It simply replaces programs by their newest version.
On the contrary, a fresh install will delete everything, and we're back with a blank page, i.e. a basic system configuration.

The reason I prefer the later is that it allows me to keep my system clean.
I am the kind of person who try lot of programs.
After a couple of months, I have installed softwares that I never use.
So every six months, I erase everything and install only programms that I know I need on a daily basis.
You can see it as a spring cleaning for my PC.

This article presents my procedure to upgrade the system.

# Things to do *a priori*

## Backup

It goes without saying, but it's always better to say it, **a fresh install will erase everything on the disk.**
Everything that we want to save need backups.
There are plenty of solutions to realize those backups; I use *rsync* to do so which is a command line tool that we can find on almost (if not all) linux operating systems.

Here's the bash program I use to make my backup:



{% highlight bash %}
# Used for the name of the backup
TIME=$(date "+%y%m%d")

# Directories to backup
TO_BACKUP="/home/denis"

# Directory which will store the backup
BACKUP_TO="/données/Backup_"$TIME

# Time to backup
rsync -az $TO_BACKUP $BACKUP_TO
{% endhighlight %}

Note that the directory `/données` I use to store my backups is a link to a second hard drive mounted on my PC, which will not be formatted.
Also, I only save my home directory but one could also want to save other directories such as `/usr/local/bin` or `/opt`.

## Preperation of the support

Everything I'll say in this part can be found on the [Fedora's official documentation](https://docs.fedoraproject.org/en-US/Fedora/23/html/Installation_Guide/), which by the way is really well written.

The first thing to do is to download [the newest version of Fedora](https://getfedora.org/fr/workstation/download/).
Once done, we need to prepare the support which will install the system.
I use a USB key for that.
The `.iso` file that has been fetched can't just be copied on the key, since a `.iso` file is a container, not a program.
To extract those files on the key, we can either use a command line tool, `dd`, or a graphical interface.
I choose the later with the software `Disk` from the base system in Gnome 3.
Here are the steps to create a bootable key:

1. Open the software
2. On the left side, there's a list of all the disks connected to the PC. The USB key should be found in it. Select it.
3. On the upper right corner, click on the menu and choose `restore disk image`
4. Locate the `.iso` file and select it.
5. Go get a coffee

Once done, you should see that the key as 4 partitions.
If not, you probably clicked on the wrong menu on step 2.

# Install

I'll go real quick here because the install wizard in very simple and everything is explained on [the documentation](https://docs.fedoraproject.org/en-US/Fedora/23/html/Installation_Guide/).
During the installation, I completely erased my primary hard drive and selected the automatic configuration.

# Things to do _a posteriori_

We have now a basic system running with the latest developments.
But I need specific softwares, and I want my configurations back.
I'll explain in this part my process to get an operational system.

For this part, I'll assume you're a root user.



{% highlight bash %}
# Let me be god
su -
{% endhighlight %}


## Update

First of all, get fresh updates.



{% highlight bash %}
dnf update
{% endhighlight %}

## Restore the backup

All my documents are stored in a backup folder on my second hard drive.
This hard drive is not mounted yet on the system, to do so, we can follow [this documentation](https://docs.fedoraproject.org/en-US/Fedora/14/html/Storage_Administration_Guide/s3-disk-storage-parted-create-part-fstab.html).
Here's the command I use to do it:



{% highlight bash %}
# Create a mount point
mkdir /données

# Add an entry to fstab so the disk is mounted at startup
echo -e "/dev/sda1 /données\t\text4 defaults" >> /etc/fstab

# Mount the disk now
mount /données
{% endhighlight %}

Now we can restore our files with a bash script.

__Note that I did NOT used these commands so check them before doing anything.__



{% highlight bash %}
# Find the latest backup folder assuming backup are named Backup_DATE
LATEST_BACKUP=$(ls /données | grep "Backup_" | sort | tail -n 1)

rsync -a /home ${LATEST_BACKUP}/*
{% endhighlight %}

As the note said, I did not used this.
Why? Because part of why I'm doing a fresh install is also to let me reorganise my personnal files.
Therefore, I prefer manually copy those where I want to.

## SSH

In order to connect to my personnal server, or to push stuff on git, I need to retrieve the keys.
SSH keys are stored in a folder `.ssh/` in my home directory.
So I can copy the folder from the backup:


{% highlight bash %}
LATEST_BACKUP=$(ls /données | grep "Backup_" | sort | tail -n 1)
cp -r ${LATEST_BACKUP}/denis/.ssh /home/denis/
{% endhighlight %}

## Passwords

I use [pass](https://www.passwordstore.org/) to manage and store my passwords.
The thing I like with pass is that it stores each password in a separate file (encrypted with GnuPG) which makes password versioning very efficient.
So retrieving all my password is as simple as:


{% highlight bash %}
dnf install pass

git clone XX.XX.XX.XX:~/.password-store
{% endhighlight %}

## Gnome-shell

Now that we have our password, we can re-configure our accounts (gmail, dropbox, ...).
Also, there are many [gnome-shell extensions](https://extensions.gnome.org/) that can be installed to personnalized the desktop environment.
Here are the one I like:


{% highlight bash %}
dnf install $(echo "
gnome-tweak-tool
gnome-shell-extension-openweather
gnome-shell-extension-pomodoro
gnome-terminal-nautilus
" | sed ':a;N;$!ba;s/\n/ /g')

# Notice the nice sed command to remove return carriage...
# Thanks to Zsolt Botykai and kenorb (see more: https://stackoverflow.com/questions/1251999/how-can-i-replace-a-newline-n-using-sed?page=1&tab=votes#tab-top)
{% endhighlight %}

## Firefox

Firefox is already installed, but as gnome-shell, there are [many addons](https://addons.mozilla.org/fr/firefox/extensions/) which make our life easier.
The good thing is that I just need to synchronized firefox with my account to get everything.

## Multimedia

By default, we can't read the main video format with Fedora.
The reason is that some codec to read those video are not proprietary, which is against the philosophy of Fedora.
If we want to read those video, we need to install the codec manually:


{% highlight bash %}
# Activiate the repository which hold non-free softwares
sudo dnf install --nogpgcheck http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm

# Download useful libraries
dnf install $(echo "
gstreamer1-libav
gstreamer1-vaapi
gstreamer1-plugins-{good,good-extras,ugly}
" | sed ':a;N;$!ba;s/\n/ /g')
{% endhighlight %}

Note that the [official documentation](http://doc.fedora-fr.org/wiki/Lecture_de_fichiers_multim%C3%A9dia) is not quite up to date at the time I'm writing those lines.

## Shotwell

This is my photo manager.
I could just open it to import all photos, but we can speed up the process by copying the old database:


{% highlight bash %}
BACKUP=$(ls /données | grep "Backup_" | sort | tail -n 1)

# The database storage
cp -r ${BACKUP}/denis/.local/share/shotwell /home/denis/.local/share

# The thumbnails storage
cp -r ${BACKUP}/denis/.cache/shotwell /home/denis/.cache/
{% endhighlight %}

## R

This is my main tool.
I use it for work and for personnal projects.
For exemple, this article is written in `rmarkdown` and the site is generated using a wrapper to convert all internal articles in the correct format.
So R is pretty important for me.
So first lets install it :


{% highlight bash %}
dnf install R
{% endhighlight %}

In the meantime, I can download [rstudio](https://www.rstudio.com/products/rstudio/download/), the IDE I use.
Then I can install packages I find useful.


{% highlight r %}
install.packages(c(
  "devtools",
  "dplyr",
  "ggvis",
  "knitr",
  "htmltools",
  "leaflet",
  "RColorBrewer",
  "rmarkdown",
  "shiny",
  "stringr",
  "tidyr"
))
{% endhighlight %}

## LaTeX

I use LaTeX for the letters I can write, my curriculum, and basically all static manuscript.


{% highlight bash %}
# First install texlive and the composant I need
dnf install $(echo "
texlive 
texlive-babel-french* 
texlive-placeins* 
texlive-siunitx* 
texlive-tabulary* 
texlive-multirow* 
texlive-circuitikz* 
texlive-pgfplots* 
texlive-tocbibind* 
texlive-minitoc* 
texlive-silence* 
texlive-glossaries* 
texlive-lettre* 
texlive-hyperref* 
texlive-xetex 
'tex(eu1enc.def)' 
texlive-textpos* 
texlive-biblatex 
texlive-framed* 
texlive-titling*
" | sed ':a;N;$!ba;s/\n/ /g')

fmtutil --all

# Copy the personnal fonts
BACKUP=$(ls /données | grep "Backup_" | sort | tail -n 1)
cp -r ${BACKUP}/denis/.fonts /home/denis/
cp -r ${BACKUP}/denis/.local/share/fonts /home/denis/.local/share/

# texstudio
sudo dnf install texstudio qt5-qtsvg
{% endhighlight %}

## Images

To create figures or change some photos:


{% highlight bash %}
dnf install $(echo "
gimp
inkscape
" | sed ':a;N;$!ba;s/\n/ /g')

{% endhighlight %}

## Text 

I write everything in LaTeX, but one useful thing in a WISIWIG is their ability to correct mistakes.
I use the extension [Grammalect](http://www.dicollecte.org/grammalecte/telecharger.php) in Libreoffice to correct my wording.
