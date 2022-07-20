## Verify Tor Browser Signature on Windows Using Gpg4win

Verification in this context means checking that no one has tampered with your software in between it leaving the developers and arriving at your PC.

In the old days, when most websites used HTTP only, verification was very important. Nowadays, with most websites using HTTPS, it's less easy for someone to tamper with a piece of software before it reaches you. 

But it does still happen. If you work in a country where the government insists that you install their own root CA certificate, then it's very important that you verify your downloads. But even without that, there was a famous case a few years ago where even Linux Mint was tampered with before people downloaded it.

So verification is still worth doing if you're the sort of person who cares about security.

### 1. Download Gpg4win

On Windows, you're going to use a tool called Gpg4win.

Download the most recent version from https://www.gpg4win.org.

You can make a donation if you wish before you get to the download page.

The download file is currently named `gpg4win-4.0.3.exe`.

### 2. Verify Gpg4win

Gpg4win itself needs to be verified before you install it.

On the web page, click where it says **Check integrity**. That opens a new tab for the [package integrity checking page](https://www.gpg4win.org/package-integrity.html).

There you'll see a code signing certificate number. It's currently serial number `4F7382A39E57A34E167CF912` from the issuer `GlobalSign GCC R45 CodeSigning CA 2020`. It may have changed by the time you read this.

1. Right-click on your downloaded file. 
2. Select **Properties**. 
3. Open the **Digital Signatures** tab.
4. Select the signature, and click **Details**.
5. Click **View Certificate**.
6. 6. Select the **Details** tab.

The actual serial number and issuer should match the expected values from the website.

### 3. Install Gpg4win

If you have a match, you can run the Gpg4win installer.

### 4. Import Tor Developers' Signing Key

Go to the Tor Project website at https://www.torproject.org.

1. In the top navigation bar, click on **Support**.
2. On the left-hand side, click **Tor Browser**.
3. Expand the section **How can I verify Tor Browser's signature?**.
4. Scroll down to the subsection headed **Fetching the Tor Developers key**.

You will see a command to fetching the Tor Developers' signing key. You need to issue that command in a Windows command prompt.

1. In Windows, hold down the **Windows** key on your keyboard, and press **R**.
2. In the run box, type `cmd`.
3. Press **Enter**.
4. A Windows command prompt window opens.

In the Windows command prompt window, issue the command given. You can copy it from the website with **Ctrl**+**C**, and paste it into the Windows command prompt window just by right-clicking on your mouse.

```bash
gpg --auto-key-locate nodefault,wkd --locate-keys torbrowser@torproject.org
```

This should show you an import message for key `EF6E286DDA85EA2A4BA7DE684E2C6E8793298290`. Notice that the key and subkey expire from time to time. If you imported the Tor Developers' signing key a long time ago, you may need to reimport it now.

Auto-locating key in a web key directory sometimes fails. If that happens, you might be able to import the key using one of the alternative methods in the subsection headed **Workaround (using a public key)**. That gives instructions for downloading binary or ASCII versions of the Tor Developers' signing key. If you do download the key by this method, the easiest way to import it is to use the graphical user interface for Gpg4win, which is a program named Kleopatra. The GUI has a special button for import.

### 5. Export the Key to a File

After importing the key, save it to a file on its own by issuing the command that follows in a a Windows command prompt window:

```bash
gpg --output ./tor.keyring --export 0xEF6E286DDA85EA2A4BA7DE684E2C6E8793298290
```

In the above command, the Tor Developers' signing key is identified by its fingerprint, which is `0xEF6E286DDA85EA2A4BA7DE684E2C6E8793298290`.

### 6. Download Tor Browser

Now you're all set to download and verify Tor Browser for Windows.

At the top of the page, click the **Download Tor Browser** link to go to the [download page](https://www.torproject.org/download).

Because we're doing verification, you need both the Windows download _and_ the corresponding signature.

They have related names. These names increase with each version number. Right now we are on version 11.5, so the names are:

* `torbrowser-install-win64-11.5_en-US.exe`
* `torbrowser-install-win64-11.5_en-US.exe.asc`

You can see that the signature file just has `.asc` added on the end of its name.

### 7. Verify Tor Browser

Go back to the instructions you were working on:

1. In the top navigation bar, click on **Support**.
2. On the left-hand side, click **Tor Browser**.
3. Expand the section **How can I verify Tor Browser's signature?**.
4. This time, scroll down to the subsection headed **Verifying the signature**.

The model instruction, which looks as follows, refers to the then-current version, which at that time was version 9.0:

```bash
gpgv --keyring .\tor.keyring Downloads\torbrowser-install-win64-9.0_en-US.exe.asc Downloads\torbrowser-install-win64-9.0_en-US.exe
```

Open a Windows command prompt. If you closed the window from before, just reopen it as before.

1. In Windows, hold down the **Windows** key on your keyboard, and press **R**.
2. In the run box, type `cmd`.
3. Press **Enter**.
4. A Windows command prompt window opens.

Issue the verification command, but substitute in the current version. For us, that's version 11.5. So the command right now is:

```bash
gpgv --keyring .\tor.keyring Downloads\torbrowser-install-win64-11.5_en-US.exe.asc Downloads\torbrowser-install-win64-11.5_en-US.exe
```

When the verification is complete, you should see a message, `Good signature from "Tor Browser Developers (signing key) <torbrowser@torproject.org>"`.

### 8. Install Tor Browser

When you've safely verified your download, you can run the installer. That's the one that ends with just `.exe`.

### 9. Run Tor Browser

After installation is complete, prove it's worked by launching Tor Browser for the first time. 
