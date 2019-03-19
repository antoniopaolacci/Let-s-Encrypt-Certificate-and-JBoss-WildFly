# Let-s-Encrypt-Certificate-and-JBoss-WildFly

Creating/Renewing Let’s Encrypt Certificate JBoss WildFly

Common file related certificate gethttpsforfree.com

<i>If you will have the following <b>PEM</b>-encoded files:</i>

- cert.pem:   Server certificate only
- chain.pem:   Root and intermediate certificates only, Let’s Encrypt chain
- fullchain.pem:   Previous cert.pem and chain.pem combined
- privkey.pem:   Your certificate's private key (do not share this with anyone)

Download the certbot-auto script

```
cd /home/utente
sudo wget https://dl.eff.org/certbot-auto
sudo chmod a+x certbot-auto
```

Now that certbot is hopefully installed, we need to ask it to renew/create certificate.

Stop all services running on port 80.

<ul>
	<li>certbot-auto renew</li>
	<li>certbot-auto certonly --standalone --standalone-supported-challenges http-01 --agree-tos --rsa-key-size 4096 --renew-by-default --email admin@example.com -d example.com -d www.example.com</li>
</ul>

[go to renew section](https://github.com/antoniopaolacci/Let-s-Encrypt-Certificate-and-JBoss-WildFly/blob/master/README.md#certbot-auto-renew)

<br>
At the end, the command show somethings like this:
<br><br>
<i>
IMPORTANT NOTES:<br>
 - Congratulations! Your certificate and chain have been saved at<br>
   /etc/letsencrypt/live/***/fullchain.pem. Your cert will<br>
   expire on ***. To obtain a new or tweaked version of this<br>
   certificate in the future, simply run certbot-auto again. To<br>
   non-interactively renew *all* of your certificates, run<br>
   "certbot-auto renew"<br>
 - If you like Certbot, please consider supporting our work by:<br>

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate<br>
   Donating to EFF:                    https://eff.org/donate-le
</i>
<br>

We need to get the public and private keys into Wildfly. Instead of (for example Apache, Nginx) that were setup with the public and private keys pointed to separately, Wildfly (generally Java) works off of a keystore.
We need to convert the PEM file into a P12 file that is readable format by the keytool.

Use openssl tools.

```
openssl pkcs12 -export -in /etc/letsencrypt/live/YOURDOMAIN/fullchain.pem -inkey /etc/letsencrypt/live/YOURDOMAIN/privkey.pem -out KEYSTORENAME.p12 -name KEYSTOREALIAS
```

YOURDOMAIN replacement is the folder corresponding to the domain that you’re generating the key for, and was present in the listed output from the previous step. 

KEYSTORENAME will become part of the generated file name (.p12), and will be used in the WildFly xml part of configuration, 
as will the KEYSTOREALIAS. 

Once you’ve pressed enter, you’ll be prompted (and verified) for a new password. 
This new password will be used in a moment when we generate the keystore.  (called it PREVIOUSPASSWORD and must be the same of other next password)


Generating the keystore java (.jks)

```
/usr/lib/jvm/jdk1.7.0_80/bin/keytool -importkeystore -deststorepass WILDFLY_NEW_STORE_PASS -destkeypass WILDFLY_NEW_KEY_PASS -destkeystore NEW_KEYSTORE_FILE.jks -srckeystore KEYSTORENAME.p12 -srcstoretype PKCS12 -srcstorepass PREVIOUSPASSWORD -alias KEYSTOREALIAS
```

```
sudo cp NEW_KEYSTORE_FILE.jks /opt/wildfly/standalone/configuration/ (jboss server config directory)
```

WILDFLY_NEW_STORE_PASS is keystore password <br>
WILDFLY_NEW_KEY_PASS   is the destination keystore password <br>
NEW_KEYSTORE_FILE      is the final .jks name <br>


Find the <security-realms> section of standalone.xml config file and specifically the one you’re setting up

```
<server-identities>
   <ssl>
      <keystore path="NEW_KEYSTORE_FILE.jks" 
                relative-to="jboss.server.config.dir" 
                keystore-password="WILDFLY_NEW_STORE_PASS" 
                alias="KEYSTOREALIAS" 
                key-password="WILDFLY_NEW_KEY_PASS"/>
   </ssl>
</server-identities>
```

Start the application server

```
service wildfly start
```


### certbot-auto renew 

Disable cronjob task that automatic restart wildfly if is down/crash, comment line with <i>#</i> char.

```
crontab -e 
```

Stop all services running on port 80

```
service apache2 stop
service wildfly stop
```

Enter dir where related certs are and make a backup dir of previous certs

```
cd dir-new-ssl-cert
```

Display contents:
- <i>domain.p12</i>
- <i>domain.jks</i>
- <i>account.key</i>
- <i>domain.key</i>
- <i>intermediate.crt</i>
- <i>domain.crt</i>

Execute:
```
./certbot-auto renew  [--dry-run option to do a test]
```

Run following command with replacement of placeholders:

```
openssl pkcs12 -export -in /etc/letsencrypt/live/YOURDOMAIN/fullchain.pem -inkey /etc/letsencrypt/live/YOURDOMAIN/privkey.pem -out NEW_KEYSTORE_FILE.p12 -name fluikappalias
```

```
/usr/lib/jvm/jdk1.7.0_80/bin/keytool -importkeystore -deststorepass WILDFLY_NEW_STORE_PASS -destkeypass WILDFLY_NEW_KEY_PASS -destkeystore NEW_KEYSTORE_FILE.jks -srckeystore NEW_KEYSTORE_FILE.p12 -srcstoretype PKCS12 -srcstorepass fluikapp -alias fluikappalias
```

Install jks on jboss folder (ex: /opt/wildfly/standalone/configuration/...)  <br> 
<i>(ex: jboss.server.config.dir on file standalone.xml)</i>

```
sudo cp NEW_KEYSTORE_FILE.jks /opt/wildfly/standalone/configuration/ 
```

Restart all services:

```
service apache2 start
service wildfly start
```
