# GNOME Keyring for UXP

An extension to store passwords and form logins in gnome-keyring.

This replaces the default password manager in UXP applications with an implementation which uses GNOME Keyring. This is a centralised system-based password manager, which is more simple to handle than per-application management.

You need `libgnome-keyring0` (or similar, depending on distro package names) to be installed, for this extension to work. When this extension is enabled, passwords stored in the default password manager are not accessible. See [migrating old passwords](#migrating-old-passwords) for an efficient set of instructions to manually work around this.

You can find more technical information on the [Bugzilla issue](https://bugzilla.mozilla.org/show_bug.cgi?id=309807) related to this.

# Usage

You can change the keyring in which passwords are saved by creating or editing the preference item `extensions.uxp-gnome-keyring.keyringName`. The default keyring is used otherwise. This is a per-profile setting, so if you don't manually change it, all profiles will share the same keyring.

You can backup your passwords easily, separately from the rest of your profile. Your keyrings are stored `~/.local/share/keyrings` - though it is possible this may change in the future (determined by your desktop environment, not this extension).

You can also take advantage of the more fine-tuned keyring management features of gnome-keyring, such as:

* No need to prompt for password, if you store in the `login` keyring and the password for that keyring is the same as your login password.
* If the keyring is already open, don't need to prompt for a password each time you start Firefox or Thunderbird.
* You can explicitly re-lock the keyring when you feel you need to.
* In gnome-keyring 3, the keyring password prompt disables keyboard input to other windows, so you don't need to worry about accidentally typing it somewhere you shouldn't

Note: gnome-keyring stores the passwords encrypted on permanent storage but it keeps unlocked passwords in memory without encryption. As a result, programs with access to the memory space of gnome-keyring (such as debuggers and applications running as root) may be able to extract the passwords. The same applies to the default password manager implementations, so this extension should not be any less secure.

# Non-working cases and workarounds

Passwords will not be saved or filled in if:

* the username or password element has attribute `autocomplete="on"`

  * workaround: delete the attribute using the DOM inspector

* the username or password element is already filled in by the page

  * see https://bugzilla.mozilla.org/show_bug.cgi?id=618698
  * note: not a browser bug

* (mozilla bug): the page is XML+XSLT

  * see https://bugzilla.mozilla.org/show_bug.cgi?id=354706

These are not directly fixable by this extension.

# Migrating old passwords

This extension cannot migrate passwords on its own.  However you can use the [Password Backup Tool](https://addons.palemoon.org/addon/password-backup-tool/) extension to export your passwords from the default password manager and then re-import them into gnome-keyring:

* Before installing this extension install the [Password Backup Tool](https://addons.palemoon.org/addon/password-backup-tool/) extension.
* Export all the passwords saved in the password manager to a XML file.
* Install this extension and restart your browser.
* Import the passwords from the previously exported file.  They will be stored in the keyring specified by `extensions.gnome-keyring.keyringName`.
* Once you're done, you are free uninstall the [Password Backup Tool](https://addons.palemoon.org/addon/password-backup-tool/) extension and [shred](https://www.gnu.org/software/coreutils/manual/html_node/shred-invocation.html) the XML file.

The old data of the default password manager remains untouched, so you also need to delete that manually if you want to. This is done by going to your profile folder, and [shred](https://www.gnu.org/software/coreutils/manual/html_node/shred-invocation.html)-ing `key3.db` and `signons.sqlite`. After deleting, the data may still be forensically retrievable from your disk, but if you were protecting it with a master password, this data would still be be encrypted.

Deleting old data will also clear the master password for the default password manager. If you don't clear it, you'll still be asked for it when you choose to "show passwords", even if this extension is active.