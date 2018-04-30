---
title: Postfix
date: 2018-04-26 08:34:30
tags:
---
This is tutorial for set up Postfix in macos

### prerequired 
Saya anggap kalian sudah menginstall Postfix di environment kalian, jika belum, bisa baca ini untuk menginstall Postfix di lokal enviroment kalian. 
adapun tool yang dibutuhkan adalah :
+ Postfix
+ vi (atau text editor kesukaanmu)


Check Postfix 

bagaimana caranya mengecek kalau kita sudah meng install Postfix atau belum ? bisa di cek di cmd bila sudah ter install. 

    $ postfix
    postfix: error: to submit mail, use the Postfix sendmail command
    postfix: fatal: the postfix command is reserved for the superuser

<!-- more -->
## Edit postfix configuration file

setelah Postfix sudah terinstall maka sekarang giliran untuk configure Postfixnya. 

    $ sudo vi /etc/postfix/main.cf

pastikan bahwa variabel dibawah tersebut ada pada file main.cf. 

    mail_owner = _postfix
    setgid_group = _postdrop

kemudian tambahkan configure pada akhir. 

    # Postfix as relay
    #
    # Gmail SMTP
    relayhost=[smtp.gmail.com]:587
    # Hotmail SMTP
    #relayhost=smtp.live.com:587
    # Yahoo SMTP
    # relayhost=smtp.mail.yahoo.com:465
    # Enable SASL authentication in the Postfix SMTP client.
    smtp_sasl_auth_enable=yes
    smtp_sasl_password_maps=hash:/etc/postfix/sasl_passwd
    smtp_sasl_security_options=noanonymous
    smtp_sasl_mechanism_filter=plain
    # Enable Transport Layer Security (TLS), i.e. SSL.
    smtp_use_tls=yes
    smtp_tls_security_level=encrypt
    tls_random_source=dev:/dev/urandom

## Create sasl_passwd file

jalankan code dibawah untuk membuat password. 

    $ sudo sh -c 'echo "\n[smtp.gmail.com]:587 your_email@gmail.com:your_password" >> /etc/postfix/sasl_passwd'

Replace your_email@gmail.com and your_password with actual values.


    $ sudo postmap /etc/postfix/sasl_passwd

## Autorun postfix on boot and restart postfix

    sudo cp /System/Library/LaunchDaemons/com.apple.postfix.master.plist /Library/LaunchDaemons/org.postfix.custom.plist


    sudo vi /Library/LaunchDaemons/org.postfix.custom.plist

Change the label value from com.apple.postfix.master to org.postfix.custom Remove these lines to prevent exiting after 60s

    <string>-e</string>
    <string>60</string>

Add these lines before </dict>

    <key>KeepAlive</key>
    <true/>

Relaunch the daemon

    sudo launchctl unload /Library/LaunchDaemons/org.postfix.custom.plist
    sudo launchctl load /Library/LaunchDaemons/org.postfix.custom.plist


Turn on less secure apps for gmail

In [Gmail](https://accounts.google.com/ServiceLogin?service=accountsettings&passive=1209600&osid=1&continue=https://myaccount.google.com/lesssecureapps&followup=https://myaccount.google.com/lesssecureapps&emr=1&mrp=security) we must switch on the option "Access for less secure apps", otherwise we will get the error: SASL authentication failed


## Test
    + echo "Test sending email from Postfix" | mail -s "Test Postfix" youremail@domain.com
    + Check mail queue and possible delivery errors with mailq.
    + Check mail log with tail -f /var/log/mail.log.




