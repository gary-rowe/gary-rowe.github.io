---
author: Gary Rowe
title: How to recover lost bitcoins from an Android wallet
layout: post
tags:
  - HowTo
  - Android
  - Bitcoin
redirect_from: /agilestack/2011/12/28/how-to-recover-lost-bitcoins-from-an-android-wallet/
---

## Background

A little while ago I discovered [Bitcoin][2]. Put simply, it allows people to send any amount of money, anywhere in the world in about ten minutes without incurring an excessive transaction fee. It’s based on a very effective use of digital signatures and a public ledger of transactions to avoid double spending. In short, it is money, reinvented. If you want to send someone less than a dollar on the other side of the world, you can do it with Bitcoin.

There are many Bitcoin clients out there (I contribute development effort to the [MultiBit][3] project) and some have taken advantage of the Android platform. In particular there is Andreas Schildbach’s [Bitcoin Wallet][4] (an outstanding mobile solution with many ongoing developments) and Brian Armstrong’s [Bitcoin Android][5] (an early client that has seen a reduction in recent commits). I’ve used Andreas’ code for some time now as my primary mobile wallet and had an excellent experience.

And then my smartphone died as a result of [faulty networking hardware][6].

Well, not a proper death, just enough to cause all the non-factory applications to stop working and to put my bitcoins (small “b” for the unit of currency with the Bitcoin protocol) at risk. Bitcoin requires you to keep your private keys safe since they provide access to your bitcoins in the public ledger (called the blockchain). Lose those private keys and you lose your bitcoins. Forever.

What follows is a guide that shows what I had to do to get my bitcoins back. If you find yourself in the same situation, perhaps this will help you. It does assume that you’re very technically proficient. If the idea of installing the Android SDK and using a shell to run up adb fills you with horror, stop now.

## What you have to do

1) Do not attempt to re-install your Bitcoin wallet application – it will very likely delete your local wallet file and that’s the end of your keys.

2) You absolutely have to root your phone. Mine is (at the time of writing) a HTC Desire, so I used unrevoked3 (see <http://unrevoked.com/>). This should not delete any data from your system, but will enable you to access otherwise protected files.

3) Install the [Android SDK][7] and get the “platform tools” variant for your platform so that you get the adb application.

4) Hook up your phone with a USB cable (you’d probably have left it in after rooting it)

5) Fire up a terminal session and enter the following command (I’m using “>”, “$” and “#” to represent where you are in the shells, don’t actually type them)

```bash
> cd <wherever you've installed Android SDK>
> adb shell
$
```

In the adb shell switch to become root

```bash
$ su
#
```

If you get a # then you’re root. If not, try again with the unrevoked3.

6) Navigate to the wallet file directory

**For the Andreas Schildbach Bitcoin Wallet do this**

```bash
# cd /data/data/de.schildbach.wallet/files
```

Copy the all-important private key file somewhere that adb can get to it

```bash
# cat key-backup-base58 > /data/local
```

You’ll notice that “cp” and “mv” are not options for adb against a production build of Android. Hence the sneaky use of `cat`. And, yes, I did try mounting the partition as read-write and totally failed to get it to work.

**For the Brian Armstrong Android Wallet do this**

```bash
# cd /data/data/com.bitcoinwallet/files
```

Copy the all-important private key file somewhere that adb can get to it

```bash
# cat prodnet.wallet > /data/local/prodnet.wallet
```

7) Exit out of the shells (root then adb)

```bash
# exit
$ exit
```

8) Pull the private key files off the device and somewhere local

```bash
> adb pull /data/local/key-backup-base58
> cat key-backup-base58
```

The Schildbach key is stored as a simple base58 format that looks a bit like this:

```text
5dsflkjsflklnsdaflsmdflmsortofthing
```

The Armstrong key is more complex, it is stored as a serialized ECKey. To get the private key out of it I had to use the [BitCoinJ][8] library (version 0.2) and make use of the DumpWallet.java file. Poking around with an IDE and debugger lead me to the private key that looked a bit like this:

```text
42a34b31e9a4eedf56980ee0fc32fe6e675ff7007651ff3sortofthing
```

So now you have to get this private key back into a safe place.

9) Use Mt Gox to import the key

By far the fastest way is to just register an account with Mt Gox (a major Bitcoin exchange) and use their very flexible private key import facility. Just select “Private key” as a deposit method and copy-paste the contents of your key. The Mt Gox exchange can recognise a wide variety of formats and if it recognises your input then it’ll immediately give you a balance associated with that key. A short while later your Mt Gox account will be credited with the bitcoins and you can do with them as you wish.

10) Finally, do a cleanup to make sure you don’t leave the private key lying around for anyone to grab easily

```bash
> adb shell
$ su

# cd /data/local
# rm key-backup-base58
# rm prodnet.wallet
# rm prodnet.keychain
# exit

$ exit

> exit
```

And you’re done.

This shows what must be done to recover your private keys and is correct at the time of writing. However, you should be aware that soon all private keys will be encrypted so that in addition to the above steps, you will also need to know the passphrase to gain access to the private key.

 [1]: https://twitter.com/share
 [2]: http://lovebitcoins.org "Bitcoin"
 [3]: https://github.com/Multibit-Legacy/multibit-website-legacy/blob/develop/src/main/resources/views/html/en "MultiBit"
 [4]: https://bitcoinj.org/ "BitcoinJ"
 [5]: https://github.com/barmstrong/bitcoin-android "GitHub"
 [6]: http://www.youtube.com/watch?v=KLl9Q5ur9Oc "HTC Desire reboot problem"
 [7]: http://developer.android.com/sdk/index.html "Android SDK"
 [8]: https://bitcoinj.org/ "BitCoinJ"
