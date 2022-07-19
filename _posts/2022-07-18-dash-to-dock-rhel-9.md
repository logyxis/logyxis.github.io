## Dash to Dock on RHEL 9, CentOS 9, Rocky Linux 9, or AlmaLinux 9

If you're running Red Hat Enterprise Linux 9 or a similar distribution, you'll notice that the launcher at the bottom functions more as a dash than as a dock. It's not there most of the time. It only appears when you click **Activities**. Then after you've selected an application, it disappears again, and you can't get it back.

If you want to change that behavior, you need to install two packages. The first one is called `gnome-shell-extension-dash-to-dock`. That provides the dash-to-dock functionality. And then to manage the extension, you need a second package, which is called `gnome extensions app`.

```bash
sudo yum install gnome-shell-extension-dash-to-dock gnome-extensions-app
```

After installing the packages, log off and log on again.

When you come back in, under your apps you now have an app called **Extensions**.

In there you have the dash-to-dock extension. When you toggle that on, you now have a dock at the bottom of the screen.

There are also some settings you can change, such as where you want the dock, and whether you want the dock on all monitors.
