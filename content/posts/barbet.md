---

title: "Setup barbet (CalyxOS)"
date: 2022-08-23T08:00:30+05:30
draft: false
toc: true

---

## Stage 0000: Make backups

```bash
adb pull /storage/self/primary/DCIM
adb pull /storage/self/primary/dot-config
adb pull /storage/self/primary/Download
adb pull /storage/self/primary/Movies
adb pull /storage/self/primary/Music
adb pull /storage/self/primary/Pictures
adb pull /storage/self/primary/Signal
```


## Stage 0001: Install apps

### Main profile

 - [Gboard](https://play.google.com/store/apps/details?id=com.google.android.inputmethod.latin)
 - [Lawnchair](https://play.google.com/store/apps/details?id=ch.deletescape.lawnchair.plah)
 - [AdminControl](https://f-droid.org/en/packages/com.davidshewitt.admincontrol/)
 - [Aegis](https://f-droid.org/en/packages/com.beemdevelopment.aegis/)
 - [Battery Bot Pro](https://f-droid.org/en/packages/com.darshancomputing.BatteryIndicatorPro/)
 - [Bitwarden](https://play.google.com/store/apps/details?id=com.x8bit.bitwarden)
 - [Clipboard Cleaner](https://f-droid.org/en/packages/io.github.deweyreed.clipboardcleaner/)
 - [Google Camera](https://play.google.com/store/apps/details?id=com.google.android.GoogleCamera)
 - [DAVx5](https://f-droid.org/packages/at.bitfire.davdroid/)
 - [Firefox Focus](https://play.google.com/store/apps/details?id=org.mozilla.focus)
 - [Simple Gallery Pro](https://f-droid.org/en/packages/com.simplemobiletools.gallery.pro)
 - [K-9 Mail](https://f-droid.org/en/packages/com.fsck.k9/)
 - [Nextcloud](https://f-droid.org/en/packages/com.nextcloud.client/)
 - [Nextcloud Notes](https://f-droid.org/en/packages/it.niedermann.owncloud.notes/)
 - [Proton PVN](https://play.google.com/store/apps/details?id=ch.protonvpn.android)
 - [Signal](https://play.google.com/store/apps/details?id=org.thoughtcrime.securesms)
 - [QKSMS](https://f-droid.org/en/packages/com.moez.QKSMS/)
 - [Telegram FOSS](https://f-droid.org/en/packages/org.telegram.messenger/)
 - [Trail Sense](https://f-droid.org/en/packages/com.kylecorry.trail_sense/)
 - [VLC](https://f-droid.org/en/packages/org.videolan.vlc/)
 - [Weather](https://f-droid.org/en/packages/wangdaye.com.geometricweather/)

### Work profile

 - [Keep notes](https://play.google.com/store/apps/details?id=com.google.android.keep)
 - [Apple Music](https://play.google.com/store/apps/details?id=com.apple.android.music)
 - [Discord](https://play.google.com/store/apps/details?id=com.discord)
 - [HDFC Bank](https://play.google.com/store/apps/details?id=com.snapwork.hdfc)
 - [Infinity](https://f-droid.org/en/packages/ml.docilealligator.infinityforreddit/)
 - [Maps](https://play.google.com/store/apps/details?id=com.google.android.apps.maps)
 - [Mastodon](https://f-droid.org/en/packages/org.joinmastodon.android/)
 - [Photos](https://play.google.com/store/apps/details?id=com.google.android.apps.photos)
 - [Shazam](https://play.google.com/store/apps/details?id=com.shazam.android)
 - [Sheets](https://play.google.com/store/apps/details?id=com.google.android.apps.docs.editors.sheets)
 - [Sennheiser Smart Control](https://play.google.com/store/apps/details?id=com.sennheiser.control)
 - [Speedtest](https://play.google.com/store/apps/details?id=org.zwanoo.android.speedtest)
 - [Twitter](https://play.google.com/store/apps/details?id=com.twitter.android)
 - [Vi](https://play.google.com/store/apps/details?id=com.mventus.selfcare.activity)
 - [WhatsApp](https://play.google.com/store/apps/details?id=com.whatsapp)
 - [YouTube](https://play.google.com/store/apps/details?id=com.google.android.youtube)


## Stage 0010: Restore backups

Restore backups for the following apps:

 - Lawnchair
 - Aegis
 - Simple Gallery Pro
 - QKSMS

```bash
adb push /home/pratham/_android/dot-config /storage/self/primary/
```
