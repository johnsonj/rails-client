# rails-client #

rails-client is an example vagrant/ansible setup for rapid Ruby on Rails development on Windows. It provisions a Ubuntu VM and necessary packages as well as providing guidance on file syncing.

## Pre-requisites ##

1. Ensure [Hyper-V is Enabled](http://blogs.technet.com/b/canitpro/archive/2014/03/11/step-by-step-enabling-hyper-v-for-use-on-windows-8-1.aspx)
2. Install [Vagrant](https://www.vagrantup.com/downloads.html)
2. Install [cwRsync](http://www.rsync.net/resources/binaries/cwRsync_3.1.0_Installer.zip)
3. Add the cwRsync bin folder to your PATH (default: `C:\Program Files (x86)\cwRsync\bin`)
4. Run the following command: `mklink /j C:\c C:\`

## First Time Setup ##

1. Clone this repo into: `C:\rails\rails-client\`
2. Create a new Rails app (or put your existing project): `C:\rails\dev\`
3. Open a command line as an Administrator and run `vagrant up` from `C:\rails\rails-client\`
4. Once provisioned, run `vagrant rsync-auto` to begin syncing folders

## Day-to-day Usage ##

1. Open a command line as an Administrator
2. Run `vagrant up && vagrant rsync-auto`
3. SSH into your machine: vagrant ssh. Default username/password is `vagrant/vagrant`
4. Access your machine via: `rails.dev` (eg: `http://rails.dev:3000`)

## File Syncing ##

By default, rails-client will rsync `C:\rails\dev` on the host to `/rails/dev` on the guest. This folder will be automatically updated every time a file is changed on the host. **Any changes made on the guest will be overwritten.** 

It will also sync `C:\rails\dev` to `/rails/dev_persistent` via SMB. This folder is a network mount so any changes made on the guest will show up on the host. It has some performance issues so in general I recommend only using that mount when you need to run rails generators and other commands that produce valuable output.

The `C:\rails\rails-client\ansible` folder will be mounted to `/ansible`

## Network Access ##

The guest needs to connect to a network with DHCP. This is a limitation I'd like to fix in the near future. For your convenience, a HOSTS entry is created so you can access your guest by connecting to `rails.dev`

## Authentication ##

When the VM is started (via `vagrant up`) it will prompt you for a username/password. This should be your active Windows username/password. If you're joined to a domain use the format `username@domain`. If you're logged in via MSA, use the user account name it's associated with, not your MSA, and your MSAs password.

The default user credentials for the VM are set by Vagrant. The username and password are both `vagrant` by default.

## Issues ##

See the issue tracker

## Improvements ##

Pull requests welcome!

## Recommended Tools ##

- [ConEmu](https://conemu.codeplex.com/) - Mutliple tabs!
- [SourceTree](http://www.sourcetreeapp.com/) - Awesome Windows based git tool
- [SublimeText](http://www.sublimetext.com/) - Speaks for itself