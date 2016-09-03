# ml-setup

*Simple installation of machine learning and data science software*

Copy/paste the contents of `bash_profile_example` into `~/.bash_profile` as the user who will run the commands below. Edit `~/.bash_profile` to suit your needs (if you're just testing this with Vagrant no editing is necessary). Logout, log back in.

Save your work, close all apps - if a kernel upgrade occurs during the process, your system **WILL REBOOT**.

Run the installer:

```
sudo ansible-playbook -i inventory main.yml
```

That's it.
