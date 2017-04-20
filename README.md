# Let-s-Encrypt-Certificate-and-JBoss-WildFly

Creating/Renewing Let’s Encrypt Certificate JBoss WildFly

Common file related certificate gethttpsforfree.com

<i>If you will have the following <b>PEM</b>-encoded files:</i>

- cert.pem:   Your domain's certificate
- chain.pem:   The Let's Encrypt chain certificate
- fullchain.pem:   cert.pem and chain.pem combined
- privkey.pem:   Your certificate's private key

Download the certbot-auto script

```
cd /home/utente
sudo wget https://dl.eff.org/certbot-auto
sudo chmod a+x certbot-auto
```

Now that certbot is hopefully installed, we need to ask it to create/renew certificate.

<ul>
	<li>./certbot-auto renew</li>
	<li>./certbot-auto certonly --agree-tos --rsa-key-size 4096 --renew-by-default -m dnsadmin@mydomain.com --webroot /var/www/html/ -d mail.mydomain.com --renew-by-default –dry-run</li>
</ul>

At the end, the command show somithings like this:

<i>new certificate deployed with reload of apache server; fullchain is
/etc/letsencrypt/live/YOURDOMAIN/fullchain.pem</i>

We need to get the public and private keys into Wildfly. Instead of (Apache, Nginx) was setup with the public and private keys pointed to separately, but Wildfly (generally, Java) works off of a keystore.
We need to convert the PEM file into a P12 file that is readable by the keytool.

```
openssl pkcs12 -export -in /etc/letsencrypt/live/YOURDOMAIN/fullchain.pem -inkey /etc/letsencrypt/live/YOURDOMAIN/privkey.pem -out KEYSTORENAME.p12 -name KEYSTOREALIAS
```

YOURDOMAIN replacement is the folder corresponding to the domain 
that you’re generating the key for, and was present in the listed output from the previous step. 

KEYSTORENAME will become part of the generated file name, 
and will be used in the WildFly configuration, as will the KEYSTOREALIAS. 

These can be anything of your choice. 

Once you’ve pressed enter, you’ll be prompted (and verified) for a new password. 
This new password will be used in a moment when we generate the keystore.  (PREVIOUSPASSWORD)


Generating the keystore

```
keytool -importkeystore -deststorepass WILDFLY_NEW_STORE_PASS -destkeypass WILDFLY_NEW_KEY_PASS -destkeystore NEW_KEYSTORE_FILE.jks -srckeystore KEYSTORENAME.p12 -srcstoretype PKCS12 -srcstorepass PREVIOUSPASSWORD -alias KEYSTOREALIAS
```

```
sudo cp NEW_KEYSTORE_FILE.jks /opt/wildfly/standalone/configuration/
```

Find the <security-realms> section and specifically the one you’re setting up

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
