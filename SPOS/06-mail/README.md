## Postfix
```bash
apt-get install postfix netcat mailutils
```
```bash
/var/spool/mail
/var/log/mail.log
```
```bash
postqueue -p
postqueue -f
postsuper -d
```
#### Sending test e-mails
```bash
echo "Test-1" | mail -s "Test mail" silhavyj@silhavyj.spos
```
```bash
nc localhost 25

HELO silhavyj.spos
MAIL FROM: silhavyj@silhavyj.spos
RCPT TO: pepa@silhavyj.spos
DATA
Subject: Testovaci
Hello Word
123
.
QUIT
```
#### Setting up a home mail directory and changing smtp_banner
```bash
vim /etc/postfix/main.cf
```
```bash
smtpd_banner = $myhostname Microsoft Exchange
home_mailbox = Maildir/
mailbox_command =
```
```bash
/etc/init.d/postfix restart
```
#### Adding aliases
```bash
vim /etc/aliases
silhavyj: root
```
```bash
newaliases
```
## Mutt
```bash
apt-get install mutt
```
```bash
mutt -f /var/spool/mail/root # mailbox
mutt -f ~pepa/Maildir        # maildir    
```
####  home folder
```bash
su - pepa
vim ~/.muttrc
```
```bash
set folder     = imap://pepa:pepa123@localhost:143/
set spoolfile  = imap://pepa:pepa123@localhost:143/
```
## Dovecot
```bash
apt-get install dovecot-imapd
apt-get install dovecot-pop3d
```
### changing mailbox to ~/Maildir
```bash
vim /etc/dovecot/conf.d/10-mail.conf
```
```bash
mail_location = maildir:~/Maildir
#   mail_location = mbox:~/mail:INBOX=/var/mail/%u
```
```bash
/etc/init.d/dovecot restart
```
### setting up SSL
```bash
vim /etc/dovecot/conf.d/10-ssl.conf
```
```bash
ssl_cert = </etc/dovecot/private/dovecot.pem
ssl_key = </etc/dovecot/private/dovecot.key
```
```bash
/etc/init.d/dovecot restart
```
### POP3
```bash
nc localhost 110

USER pepa
PASS pepa123
LIST
RETR 1
DELE 1
QUIT
```
### IMAP
```bash
nc localhost 143

1 LOGIN pepa pepa123
2 LIST "" "*"
3 STATUS INBOX (MESSAGES)
4 EXAMINE INBOX
5 FETCH 1 BODY[]
6 LOGOUT
```
## Virtual e-mails
#### Chaning accounts for receiving
```bash
vim /etc/postfix/main.cf
```
```bash
virtual_alias_domains_map = hash:/etc/postfix/virtual_domains
virtual_alias_maps = hash:/etc/postfix/virtual
```
#### Virtual domains
```bash
vim /etc/postfix/virtual_domains
```
```bash
prvni.spos  OK
druha.spos  OK
```
```bash
postmap /etc/postfix/virtual_domains
```
#### Virtual accounts
```bash
vim /etc/postfix/virtual
```
```bash
info@prvni.spos silhavyj
info@druha.spos pepa
```
```bash
postmap /etc/postfix/virtual
```
#### Sending test e-mails
```bash
echo "Test-info" | mail -s "Test mail" info@prvni.spos
echo "Test-info" | mail -s "Test mail" info@druha.spos
```
