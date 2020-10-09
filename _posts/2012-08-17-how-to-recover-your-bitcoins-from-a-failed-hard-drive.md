---
author: Gary Rowe
title: How to recover your bitcoins from a failed hard drive
layout: post
tags:
  - Bitcoin
  - HowTo
redirect_from: /agilestack/2012/08/17/how-to-recover-your-bitcoins-from-a-failed-hard-drive/
---

It happens to all of us eventually. You’re working along quite merrily and then suddenly everything freezes up. You try switching it off and on again and the machine fails to boot up. In short, your hard drive has failed and you now face a serious interruption to your day. 

And slowly it dawns on your that you had a Bitcoin wallet on that disk. And that it is worth a lot of money. You must get that file back *no matter what*. 

Here is a set of instructions that will allow you to do exactly that, depending on the state of your drive and the effectiveness of your backup strategy.

### You’ve got a reliable continuous backup strategy in place

Well done. Your continuous backup strategy has saved you from a serious financial loss. You should be able to locate your encrypted backup wallet file with the up to date keys fairly easily depending on which Bitcoin client you’re using. 

Just skip to the “Get the wallet” section at the end of the article for more details.

### Everyone else

Don’t beat yourself up over it. I’ve been there, and I’m an IT professional. I’m not going to point the Finger Of Blame at you for having your backup process fail. What follows is a series of steps that you need to do to have the best chance of getting your bitcoins back. 

I’m going to assume that:

*   you are technically proficient and don’t mind taking hard drives to pieces if you have to,
*   you’ve got a good set of precision tools to hand,
*   you know how to handle electronic equipment using electrostatic precautions.

Some of these steps require those assumptions, but you might strike it lucky and not need them at all so use your own judgement. 

Here goes. Best of luck to you…

### Replace your busted hard drive right away

Shut down your computer and make sure it is off. This will increase the chances of any data recovery operation since the hard drive heads aren’t tracking over the platters.

Replace the hard drive immediately. Ideally, if your machine is under warranty get them to replace the hard drive, but make sure that you get the original busted one back. Of course, this will take days and you’re in a hurry. Also, the more security conscious Bitcoin folks will require that you don’t let the hard drive out of your sight since the keys could be duplicated. Use your own judgement here.

Assuming that you’re going to do this at home by yourself then know that a standard PC hard drive is pretty easy to replace with a standard set of screwdrivers, whereas a Mac will probably need precision screwdrivers like 00 [Phillips][2] and 00 [Torx][3]. Just [search for illustrated guides on replacing hard drives][4] for your brand of machine. 

**Cracking open the case will probably invalidate your warranty.**

Once the new hard drive is in place, you’ll probably want to start installing your stuff. Do all that you need to in order to get back up and running as soon as possible. 

### Calm down before you start data recovery

Take some time to calm down. So long as your busted hard drive is unpowered and stored properly, your data and bitcoins will wait.

Once you’ve regained a Zen-like frame of mind, put the busted hard drive into a [generic USB hard drive cradle][5]. These are available from most electronic stores quite cheaply and are very useful to have lying around when you are dealing with raw hard drives. Alternatively, you can mount your busted hard drive into a spare bay if available, or dangle an IDE or SATA cable out of the motherboard.

Power up your hard drive and listen carefully to it (see how useful that USB cradle is now?). If you hear a “rrrrr rrrrr rrrrr” noise then you’re in luck. However you might hear a “click click click” which is very bad. In the first case the drive is trying to overcome a bad sector and the heads are having trouble making sense of the data. In the second, those heads are broken and are tracking all over the platter probably causing more damage. You may want to start thinking about data recovery services at this point (see later) but you could be mistaken so it’s worth having a shot at recovery since you can’t make it any worse.

Install a well-regarded file recovery application for your operating system. I found that the [Stellar File Recovery application][6] did a good job.

Now point your data recovery software at the busted hard drive, and see if it can be mounted. If it comes straight up then you can skip to the “Get the wallet” section.

### Top tips for mounting the drive

Given that the drive isn’t mounting immediately try the following in order. You should power the drive down, do the operation, then power it back up and try to get it to mount in each case before proceeding to the next step. **Don’t just try them all in one go and hope for the best** because each one increases the level of damage on your drive and reduces the chances of getting the data back. Again, you are entirely responsible for your own actions and should use your own judgement.

1.  give it a couple of light taps with the side of a screwdriver to free up any stuck heads
2.  observing anti-static precautions carefully unscrew the hard drive control circuit board and clean any electrical contacts that show signs of wear or dirt, then carefully reassemble
3.  put the hard drive into a *sealed* freezer bag and put it in the freezer for an hour and a half (seriously – I have personal experience of this working)

With the freezer option, try to make sure that you have your data recovery software and hardware all set up and ready to go since it may be a one-off that it mounts and it may not stay mounted for long. Don’t rush, but don’t try it just before going out for dinner either.

### Forensic data recovery

If all the above fail then consider a specialist forensic data recovery service. Factors that you need to take into account are:

* you can always buy new bitcoins and take the hit against their present value
* it is very hard to completely erase data from magnetic media unless you actively use “shred” or smash it to pieces
* a partial recovery of an encrypted wallet is probably useless (encryption is all or nothing but see the next point)
* a partial recovery of a plaintext private key where the missing elements are few and known can be brute forced back into life (you will need a friendly developer to help you here)
* forensic data recovery is expensive due to the clean room requirements and specialised knowledge
* forensic data recovery is usually covered by confidentiality clauses unless a crime is detected
* your busted hard drive will be gone for weeks so think about bitcoin cash flow if you’re running a business

### Get the wallet

Now that your busted hard drive is co-operating (or your backup strategy has worked and your new drive is ready) you just need to look around for something like “wallet.dat” or “multbit.wallet” or similar. The online help for your Bitcoin client of choice will guide you here, but some common clients are covered below:

[Instructions for recovering the wallet file for the Satoshi client][7]  
[Instructions for recovering the wallet file for the MultiBit client][8]

### Moving on

Hopefully this article has helped you get your bitcoins back. Don’t get burned by a failed backup strategy a second time. Make sure you protect your bitcoins by remembering the following:

* **data does not exist if it is not in two places**
* the cost of a NAS box with RAID is small in comparison to forensic data recovery
* cloud based backup is convenient, but make sure you only put encrypted wallets there (an AES-encrypted WinZip file with a strong password is sufficient)
* take frequent backups on to a USB key in case you can’t access the cloud
* consider using a [brain wallet][9]

Please add your own experiences into the comments if they can help others.

 [1]: https://twitter.com/share
 [2]: https://en.wikipedia.org/wiki/Phillips_head#Phillips
 [3]: https://en.wikipedia.org/wiki/Torx
 [4]: https://duckduckgo.com/?q=how+to+replace+a+hard+drive
 [5]: https://duckduckgo.com/?q=usb+hard+drive+cradle
 [6]: http://www.stellarinfo.com/
 [7]: https://en.bitcoin.it/wiki/Securing_your_wallet#Restore
 [8]: https://github.com/Multibit-Legacy/multibit-website-legacy/blob/develop/src/main/resources/views/html/en/help_walletManagement.html
 [9]: http://bitcoinmagazine.net/brain-wallets-the-what-and-the-how/