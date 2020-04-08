Samsung Firmware Magic
======================

Samsung distributes firmware updates for their SSDs for either "Windows" or
"Mac". Ironically, both of these are bootable Linux `.iso` files, containing the
actual firmware and update program.

The `.iso` files can be unpacked, but ultimately we end up with an obfuscated
binary blob, even for the meta information.

For the upstream file downloads, see

    https://www.samsung.com/semiconductor/minisite/ssd/download/tools/

Out of curiosity, I decided to create a decryption tool for this obfuscated
format, which is found in this repository.

Unpacking iso image to firmware blob
------------------------------------

First, we download a firmware iso:

    `wget http://downloadcenter.samsung.com/content/FM/201711/20171102105105735/Samsung_SSD_850_PRO_EXM04B6Q_Win.iso`

Next, we unpack the relevant file from the iso, the `initrd`:

    `7z x Samsung_SSD_850_PRO_EXM04B6Q_Win.iso initrd`

This file is a gzip-compressed cpio archive, so use 7z to strip gzip:

    `7z x initrd`

This produces `initrd~`, containing the uncompressed contents. From here we
extract the directory of interest, `root/fumagician`:

    `7z -ofw x 'initrd~' root/fumagician`

This creates `fw/root/fumagician` in the current directory:

```
$ cd fw/root/fumagician
$ ls -l
total 5408
-rw-rw-r-- 1 user user    2124 1971-03-22 19:52 DSRD.enc
-rw-rw-r-- 1 user user 4752867 1971-03-22 19:52 EXM04B6Q.enc
-rw-rw-r-- 1 user user  772516 2016-10-14 10:42 fumagician
-rw-rw-r-- 1 user user     290 2016-10-14 10:42 fumagician.sh
```

The files `DSRD.enc` (xml list of firmwares) and `EXM04B6Q.enc` (firmwares)
are the obfuscated files, that we can now decrypt.

Decrypting firmware blob
------------------------

The included `decode.py` script will unpack these `.enc` files, like so:

```shell
## show xml on stdout:
$ ./samsung-magic.py < fw/root/fumagician/DSRD.enc

## decrypt firmware to file:
$ ./samsung-magic.py < fw/root/fumagician/EXM04B6Q.enc > EXM04B6Q.bin
```

Seemingly, the folks at Samsung are huge fans of nesting things, because the
decrypted `EXM04B6Q.bin` file is actually a zip file, containing encrypted
firmware files:

```shell
$ unzip -l EXM04B6Q.bin
Archive:  EXM04B6Q.bin
  Length      Date    Time    Name
---------  ---------- -----   ----
  1048576  2017-02-19 10:41   EXM04B6Q_10170217.enc
  1048576  2017-02-19 10:41   EXM04B6Q_20170203.enc
  1048576  2017-02-19 10:41   EXM04B6Q_30170203.enc
  1048576  2017-02-19 10:41   EXM04B6Q_40170902.enc
  1048576  2017-02-19 10:41   EXM04B6Q_50170208.enc
  1048576  2017-02-19 10:41   EXM04B6Q_60170208.enc
---------                     -------
  6291456                     6 files
```

Luckily, the encryption is exactly the same, so `samsung-magic.py` can decrypt
these as well:

```
$ unzip EXM04B6Q.bin
$ ./samsung-magic.py < EXM04B6Q_10170217.enc > EXM04B6Q_10170217.bin
```

Now, at last, we have the raw firmware.

Enjoy!
