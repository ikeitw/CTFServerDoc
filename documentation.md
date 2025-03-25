
# CTF Website Setup Guide

## Step 1: Change the User Password
Change the password for the VM's user:
```sh
sudo passwd student
```

## Step 2-3: Configure SSH Access
### Generate an SSH Key on Your Host Machine
Run the following command:
```sh
ssh-keygen -t rsa -b 4096
```
Copy the SSH key:
```sh
type $env:USERPROFILE\.ssh\id_rsa.pub
```

### Enable SSH Login Without a Password on the Linux VM
Run these commands:
```sh
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
```
_Paste your SSH key there._

Set correct permissions:
```sh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
To add the teacher’s SSH key:
```sh
nano ~/.ssh/authorized_keys
```
_Paste the teacher’s SSH key._

## Step 4: Install Apache
Update the package list and install Apache:
```sh
sudo apt update
sudo apt install apache2 -y
```
Check if Apache is running:
```sh
sudo systemctl status apache2
```
If it’s not running, start it:
```sh
sudo systemctl start apache2
```
To check if Apache is working, visit:  
[CTF Website](http://CTF-debian12-30.ctf.edu.technet.howest.be)

## Step 5: Configure Apache VirtualHost
Create a new configuration file:
```sh
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/ctf-website.conf
```
Edit the configuration file:
```sh
sudo nano /etc/apache2/sites-available/ctf-website.conf
```
Modify it to look like this:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ctf-website.local
    DocumentRoot /var/www/ctf-website

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

## Step 6: Create Website Directory
The default web user is `www-data`.  
Create the website folder:
```sh
sudo mkdir -p /var/www/ctf-website
```
Add a test index file:
```sh
echo "<h1>Hello from CTF Website</h1>" | sudo tee /var/www/ctf-website/index.html
```
Set proper permissions:
```sh
sudo chown -R www-data:www-data /var/www/ctf-website
sudo chmod -R 755 /var/www/ctf-website
```
Enable the new configuration and disable the default:
```sh
sudo a2ensite ctf-website.conf
sudo a2dissite 000-default.conf
sudo systemctl reload apache2
```
Check the website:  
[CTF Website](http://CTF-debian12-30.ctf.edu.technet.howest.be)

**Expected Output:**  
![Step 6 Screenshot](screen1.jpg)

## Step 7: Create Website Structure
Run the following commands to create the directory structure:
```sh
sudo mkdir -p /var/www/html/a /var/www/html/b /var/www/html/1/2/3 /var/www/html/mickey /var/www/html/mouse /var/www/html/mickeymouse
```
Create the necessary files:
```sh
sudo touch /var/www/html/a/test.html
sudo touch /var/www/html/1/2/3/test.html
sudo touch /var/www/html/mickey/index.html
sudo touch /var/www/html/mouse/index.html
sudo touch /var/www/html/mickeymouse/index.html
```
Add content to the files:
```sh
echo "<h2>This is A - test.html</h2>" | sudo tee /var/www/html/a/test.html
echo "<h2>This is nested file at /1/2/3/test.html</h2>" | sudo tee /var/www/html/1/2/3/test.html
echo "<h2>Welcome to Mickey's page</h2>" | sudo tee /var/www/html/mickey/index.html
echo "<h2>Welcome to Mouse's page</h2>" | sudo tee /var/www/html/mouse/index.html
echo "<h2>This is MickeyMouse merged site</h2>" | sudo tee /var/www/html/mickeymouse/index.html
```
Set the correct ownership:
```sh
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```
Test the website:  
[CTF Website](http://CTF-debian12-30.ctf.edu.technet.howest.be/1/)

### Fix Document Root Issue
Since the document root is `/var/www/ctf-website`, move the directories:
```sh
sudo mv /var/www/html/a /var/www/ctf-website/
sudo mv /var/www/html/b /var/www/ctf-website/
sudo mv /var/www/html/1 /var/www/ctf-website/
sudo mv /var/www/html/mickey /var/www/ctf-website/
sudo mv /var/www/html/mouse /var/www/ctf-website/
sudo mv /var/www/html/mickeymouse /var/www/ctf-website/
```
Fix ownership again:
```sh
sudo chown -R www-data:www-data /var/www/ctf-website/
```

## Step 8: Test the Website Again
Visit the following link to verify that the website is working correctly:  
[CTF Website](http://CTF-debian12-30.ctf.edu.technet.howest.be/1/)

**Expected Output:**  
![Step 8 Screenshot](screen2.jpg)  
![Step 8 Screenshot](screen3.jpg)  
![Step 8 Screenshot](screen4.jpg)  
![Step 8 Screenshot](screen5.jpg)  
![Step 8 Screenshot](screen6.jpg)  
![Step 8 Screenshot](screen7.jpg)  
![Step 8 Screenshot](screen8.jpg)  
![Step 8 Screenshot](screen9.jpg)  

---

## Step 9: Use robots.txt to Restrict Search Engine Crawlers
Create a `robots.txt` file:
```sh
sudo nano /var/www/ctf-website/robots.txt
```
Add the following content:
```txt
User-agent: *
Disallow: /a/
Disallow: /b/
```
### Optional: Add `.htaccess` for Extra Protection
Create `.htaccess` files for restricted directories:
```sh
sudo nano /var/www/ctf-website/a/.htaccess
sudo nano /var/www/ctf-website/b/.htaccess
```
Add this content to each `.htaccess` file:
```apache
RewriteEngine On
RewriteCond %{HTTP_USER_AGENT} ^.*(bot|crawl|slurp|spider).*$ [NC]
RewriteRule ^.*$ - [F,L]
```
### Set Correct Permissions
```sh
sudo chown www-data:www-data /var/www/ctf-website/robots.txt
sudo chmod 644 /var/www/ctf-website/robots.txt
sudo chown -R www-data:www-data /var/www/ctf-website/
sudo chmod -R 755 /var/www/ctf-website/
```
### Reload Apache to Apply Changes
```sh
sudo systemctl reload apache2
```
Test the robots.txt file:  
[robots.txt](http://ctf-debian12-30.ctf.edu.technet.howest.be/robots.txt)

**Expected Output:**  
![robots.txt Screenshot](screen10.jpg)

---

## Step 10: Verify Apache Configuration
Check the `ctf-website.conf` configuration file:
```sh
cat /etc/apache2/sites-enabled/ctf-website.conf
```
It should look like this:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ctf-website.local
    DocumentRoot /var/www/ctf-website

    ErrorLog ${APACHE_LOG_DIR}/ctf-error.log
    CustomLog ${APACHE_LOG_DIR}/ctf-access.log combined
</VirtualHost>
```

---

## Step 11: Disable Default Apache Site
Run the following command:
```sh
sudo a2dissite 000-default.conf
```
Reload Apache:
```sh
sudo systemctl reload apache2
```

---

## Step 12: Harden Apache Security Settings
Edit the main Apache configuration file:
```sh
sudo nano /etc/apache2/apache2.conf
```
Find these lines:
```apache
ServerTokens OS
ServerSignature On
```
Modify them as follows:
```apache
ServerTokens Prod
ServerSignature Off
```
Restart Apache to apply the changes:
```sh
sudo systemctl restart apache2
```
Test the configuration by visiting:  
[CTF Website](http://ctf-debian12-30.ctf.edu.technet.howest.be/a/12312)

**Expected Output:**  
![Step 12 Screenshot](screen11.jpg)

---

## Step 13: Improve Apache Security by Restricting Directory Listings
Edit the Apache site configuration:
```sh
sudo nano /etc/apache2/sites-available/ctf-website.conf
```
Inside the `<VirtualHost *:80>` block, add the following:
```apache
<Directory /var/www/ctf-website>
    Options -Indexes +FollowSymLinks
    AllowOverride All
</Directory>
```
### Final Configuration Should Look Like:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ctf-website.local
    DocumentRoot /var/www/ctf-website

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/ctf-website>
        Options -Indexes +FollowSymLinks
        AllowOverride All
    </Directory>

</VirtualHost>
```
Reload Apache:
```sh
sudo systemctl reload apache2
```
Test the website again:  
[CTF Website](http://ctf-debian12-30.ctf.edu.technet.howest.be/1/)

**Expected Output:**  
![Step 13 Screenshot](screen12.jpg)

## Step 14: Enable Hostname Lookups
Edit the Apache configuration file:
```sh
sudo nano /etc/apache2/apache2.conf
```
Add this line at the bottom:
```apache
HostnameLookups On
```
Restart Apache to apply the changes:
```sh
sudo systemctl restart apache2
```
Test it using:
```sh
sudo tail -f /var/log/apache2/access.log
```
Then visit:  
[CTF Website](http://ctf-debian12-30.ctf.edu.technet.howest.be/1/)

---

## Step 15: Enable Security Headers
Enable the Apache headers module (if not already enabled):
```sh
sudo a2enmod headers
sudo systemctl restart apache2
```
Edit your VirtualHost configuration:
```sh
sudo nano /etc/apache2/sites-available/ctf-website.conf
```
Modify it to include security headers:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ctf-website.local
    DocumentRoot /var/www/ctf-website

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/ctf-website>
        Options -Indexes +FollowSymLinks
        AllowOverride All
    </Directory>

    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set Content-Security-Policy "script-src 'self';"
    Header always set Referrer-Policy "same-origin"
</VirtualHost>
```
Reload Apache:
```sh
sudo systemctl reload apache2
```
Verify the headers:
```sh
curl -I http://ctf-debian12-30.ctf.edu.technet.howest.be
```
**Expected Output:**  
![Security Headers Screenshot](screen13.jpg)

---

## Step 16: Enable Basic Authentication
### Install Required Package
```sh
sudo apt install apache2-utils -y
```
### Create a Password File and User
```sh
sudo htpasswd -c /etc/apache2/.htpasswd student
```
### Protect a Directory (e.g., `/mickeymouse`)
Edit the VirtualHost configuration:
```sh
sudo nano /etc/apache2/sites-available/ctf-website.conf
```
Add the following block inside `<VirtualHost *:80>`:
```apache
<Directory "/var/www/ctf-website/mickeymouse">
    AuthType Basic
    AuthName "Restricted Area"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>
```
Reload Apache:
```sh
sudo systemctl reload apache2
```
Test the authentication:  
[Restricted Page](http://ctf-debian12-30.ctf.edu.technet.howest.be/mickeymouse/)

**Login Credentials:**
- **Username:** `student`
- **Password:** `passwordctf30`

---

## Step 17: Create Additional Users for Authentication
### Add Users
```sh
sudo htpasswd -c /etc/apache2/.htpasswd mickey
```
Set password for `mickey`.  
Repeat for `mouse` (without `-c` flag):
```sh
sudo htpasswd /etc/apache2/.htpasswd mouse
```
Verify the `.htpasswd` file:
```sh
cat /etc/apache2/.htpasswd
```
Example output:
```
mickey:$apr1$CsUe9LMW$ojcF5dTjk7pJrTBvFuwX.1
mouse:$apr1$hpT01mqM$45p0yufN0cKjfGvPhPwic0
```

### Update VirtualHost Configuration
Edit the configuration file:
```sh
sudo nano /etc/apache2/sites-available/ctf-website.conf
```
Add the following blocks inside `<VirtualHost *:80>`:
```apache
<Directory "/var/www/ctf-website/mickey">
    AuthType Basic
    AuthName "Mickey Only Area"
    AuthUserFile /etc/apache2/.htpasswd
    Require user mickey
</Directory>

<Directory "/var/www/ctf-website/mouse">
    AuthType Basic
    AuthName "Mouse Only Area"
    AuthUserFile /etc/apache2/.htpasswd
    Require user mouse
</Directory>

<Directory "/var/www/ctf-website/mickeymouse">
    AuthType Basic
    AuthName "Mickey & Mouse Shared Area"
    AuthUserFile /etc/apache2/.htpasswd
    Require user mickey mouse
</Directory>
```
### Final Configuration:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ctf-website.local
    DocumentRoot /var/www/ctf-website

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/ctf-website>
        Options -Indexes +FollowSymLinks
        AllowOverride All
    </Directory>

    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set Content-Security-Policy "script-src 'self';"
    Header always set Referrer-Policy "same-origin"

    <Directory "/var/www/ctf-website/mickeymouse">
        AuthType Basic
        AuthName "Restricted Area"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>

    <Directory "/var/www/ctf-website/mickey">
        AuthType Basic
        AuthName "Mickey Only Area"
        AuthUserFile /etc/apache2/.htpasswd
        Require user mickey
    </Directory>

    <Directory "/var/www/ctf-website/mouse">
        AuthType Basic
        AuthName "Mouse Only Area"
        AuthUserFile /etc/apache2/.htpasswd
        Require user mouse
    </Directory>

    <Directory "/var/www/ctf-website/mickeymouse">
        AuthType Basic
        AuthName "Mickey & Mouse Shared Area"
        AuthUserFile /etc/apache2/.htpasswd
        Require user mickey mouse
    </Directory>
</VirtualHost>
```
Reload Apache:
```sh
sudo systemctl reload apache2
```
### Test Access:
1. **Mickey’s Page (only `mickey` can access):**  
   [Mickey's Page](http://ctf-debian12-30.ctf.edu.technet.howest.be/mickey/)

2. **Mouse’s Page (only `mouse` can access):**  
   [Mouse's Page](http://ctf-debian12-30.ctf.edu.technet.howest.be/mouse/)

3. **Mickey & Mouse Shared Page (both `mickey` and `mouse` can access):**  
   [Shared Page](http://ctf-debian12-30.ctf.edu.technet.howest.be/mickeymouse/)

## Step 18: Capturing HTTP Basic Authentication with Wireshark
To demonstrate how HTTP Basic Authentication credentials can be intercepted, use Wireshark.

### Steps:
1. Open Wireshark and start capturing network traffic.
2. Run the following command in your terminal or CMD:
    ```sh
    curl -u mickey:mickey http://ctf-debian12-30.ctf.edu.technet.howest.be/mickey/
    ```
3. In Wireshark, apply the following filter:
    ```
    http.authorization
    ```
4. You should see something like this in the captured packets:
    ```
    Authorization: Basic bWlja2V5Om1pY2tleQ==
    ```
5. Decode the base64 string:
    ```
    echo "bWlja2V5Om1pY2tleQ==" | base64 --decode
    ```
6. Output:
    ```
    mickey:mickey
    ```

**Note:** This step cannot be demonstrated due to VPN encryption, which prevents capturing unencrypted traffic.

---

## Step 19: Configuring HTTPS with a Self-Signed Certificate

### 1. Create a directory for SSL certificates:
```sh
sudo mkdir -p /etc/ssl/server
```
### 2. Generate a self-signed SSL certificate:
```sh
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/server/ctf-server.key -out /etc/ssl/server/ctf-server.crt
```
Provide the following details:
- **Country Name:** `BE`
- **State or Province:** `West-Flanders`
- **Locality:** `Bruges`
- **Organization Name:** `Howest`
- **Organizational Unit Name:** `CS`
- **Common Name:** `CTF-debian12-30.ctf.edu.technet.howest.be`
- **Email Address:** `group30ctf@howest.be`

### 3. Enable the SSL module in Apache:
```sh
sudo a2enmod ssl
```

### 4. Create a new configuration file for HTTPS:
```sh
sudo cp /etc/apache2/sites-available/ctf-website.conf /etc/apache2/sites-available/ctf-website-ssl.conf
```

### 5. Edit the SSL configuration:
```sh
sudo nano /etc/apache2/sites-available/ctf-website-ssl.conf
```
Add the following configuration:
```apache
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName ctf-website.local
    DocumentRoot /var/www/ctf-website

    ErrorLog ${APACHE_LOG_DIR}/ctf-ssl-error.log
    CustomLog ${APACHE_LOG_DIR}/ctf-ssl-access.log combined

    SSLEngine on
    SSLCertificateFile /etc/ssl/server/ctf-server.crt
    SSLCertificateKeyFile /etc/ssl/server/ctf-server.key

    <Directory /var/www/ctf-website>
        Options -Indexes +FollowSymLinks
        AllowOverride All
    </Directory>

    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set Content-Security-Policy "script-src 'self';"
    Header always set Referrer-Policy "same-origin"
</VirtualHost>
```

### 6. Enable the HTTPS site and restart Apache:
```sh
sudo a2ensite ctf-website-ssl.conf
sudo systemctl reload apache2
```

### 7. Test HTTPS:
Visit:  
[Secure CTF Website](https://ctf-debian12-30.ctf.edu.technet.howest.be/)

**Expected Output:**  
![HTTPS Screenshot](screen14.jpg)

---

## Step 20: VPN Limitation for HTTPS Demonstration
Due to VPN encryption, it is not possible to intercept HTTPS traffic for demonstration.

---

## Step 21: Redirect HTTP to HTTPS

### 1. Edit the HTTP site configuration:
```sh
sudo nano /etc/apache2/sites-available/ctf-website.conf
```
```sh
sudo nano /etc/apache2/sites-available/ctf-website-ssl.conf
```

### 2. Replace the `<VirtualHost *:80>` block with:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ctf-website.local
    Redirect permanent / https://ctf-debian12-30.ctf.edu.technet.howest.be/
</VirtualHost>
```

### 3. Enable the updated site configuration and reload Apache:
```sh
sudo a2ensite ctf-website.conf
sudo systemctl reload apache2
```

### 4. Test the redirection:
Visit:  
[CTF Website](http://ctf-debian12-30.ctf.edu.technet.howest.be/)

You should be redirected to the HTTPS version automatically.


## Step 22: Create a Root Certificate Authority (CA)
### 1. Create a folder for the CA:
```sh
sudo mkdir -p /etc/ssl/server/MyOrg-RootCA
cd /etc/ssl/server/MyOrg-RootCA
```
### 2. Generate the CA private key:
```sh
sudo openssl genrsa -out MyOrg-RootCA.key 4096
```
### 3. Generate the CA certificate:
```sh
sudo openssl req -x509 -new -nodes -key MyOrg-RootCA.key -sha256 -days 3650 -out MyOrg-RootCA.crt
```
Provide the following details:
- **Country Name:** `BE`
- **State or Province:** `West-Flanders`
- **Locality:** `Bruges`
- **Organization Name:** `Howest`
- **Organizational Unit Name:** `CS`
- **Common Name:** `MyOrg Root CA`
- **Email Address:** `group30ctf@howest`

### 4. Verify the CA files:
```sh
ls /etc/ssl/server/MyOrg-RootCA
```
Expected output:
```
MyOrg-RootCA.crt  MyOrg-RootCA.key
```

---

## Step 23: Create a Certificate Signed by the CA
### 1. Navigate to the SSL folder:
```sh
cd /etc/ssl/server
```
### 2. Generate a new private key and CSR:
```sh
sudo openssl genrsa -out signed.key 2048
sudo openssl req -new -key signed.key -out signed.csr
```
Provide the following details:
- **Country Name:** `BE`
- **State or Province:** `West-Flanders`
- **Locality:** `Bruges`
- **Organization Name:** `Howest`
- **Organizational Unit Name:** `CS`
- **Common Name:** `ctf-debian12-30.ctf.edu.technet.howest.be`
- **Email Address:** `group30ctf@howest`

### 3. Sign the CSR with the CA:
```sh
sudo openssl x509 -req -in signed.csr -CA MyOrg-RootCA/MyOrg-RootCA.crt -CAkey MyOrg-RootCA/MyOrg-RootCA.key -CAcreateserial -out signed.crt -days 365 -sha256
```

### 4. Update Apache's SSL Configuration:
Edit the Apache SSL configuration:
```sh
sudo nano /etc/apache2/sites-available/ctf-website-ssl.conf
```
Change these lines:
```apache
SSLCertificateFile /etc/ssl/server/ctf-server.crt
SSLCertificateKeyFile /etc/ssl/server/ctf-server.key
```
To:
```apache
SSLCertificateFile /etc/ssl/server/signed.crt
SSLCertificateKeyFile /etc/ssl/server/signed.key
SSLCACertificateFile /etc/ssl/server/MyOrg-RootCA/MyOrg-RootCA.crt
```
### 5. Reload Apache:
```sh
sudo systemctl reload apache2
```

---

## Steps 24-26: Import the CA Certificate
### 1. Copy the CA certificate to the web directory:
```sh
sudo cp /etc/ssl/server/MyOrg-RootCA/MyOrg-RootCA.crt /var/www/ctf-website/ca.crt
```
### 2. Download the CA certificate:
Visit:  
[Download CA Certificate](http://ctf-debian12-30.ctf.edu.technet.howest.be/ca.crt)

### 3. Import the CA certificate:
1. Press `Win+R`, type `certmgr.msc`, and hit **Enter**.
2. Navigate to:  
   `Trusted Root Certification Authorities → Certificates`
3. Right-click **Certificates**, select **All Tasks → Import**.
4. Select `ca.crt`, import it, and place it in **Trusted Root Certification Authorities**.
5. Complete the wizard and confirm the import was successful.

### 4. Restart your browser and visit:
[Secure CTF Website](https://ctf-debian12-30.ctf.edu.technet.howest.be/)

**Issue:** The browser might still display a warning due to missing SAN.

---

## Step 27: Fixing the SAN (Subject Alternative Name) Issue
### 1. Create a SAN configuration file:
```sh
sudo nano /etc/ssl/server/san.conf
```
Paste the following content:
```ini
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[ dn ]
C = BE
ST = West-Flanders
L = Bruges
O = Howest
OU = CS
CN = ctf-debian12-30.ctf.edu.technet.howest.be

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = ctf-debian12-30.ctf.edu.technet.howest.be
IP.1 = 192.168.30.101
```

### 2. Generate a new CSR with SAN:
```sh
cd /etc/ssl/server
sudo openssl req -new -nodes -out signed.csr -keyout signed.key -config san.conf
```

### 3. Sign the CSR with SAN using your CA:
```sh
sudo openssl x509 -req -in signed.csr -CA MyOrg-RootCA/MyOrg-RootCA.crt -CAkey MyOrg-RootCA/MyOrg-RootCA.key -CAcreateserial -out signed.crt -days 365 -sha256 -extensions req_ext -extfile san.conf
```

### 4. Reload Apache:
```sh
sudo systemctl reload apache2
```

Now visit:  
[Secure CTF Website](https://ctf-debian12-30.ctf.edu.technet.howest.be/)  
There should be no security warnings.

---

## Step 28: Client Certificate Authentication
### 1. Generate a client certificate and key:
```sh
cd /etc/ssl/server
sudo openssl genrsa -out client.key 2048
sudo openssl req -new -key client.key -out client.csr
```
Use `client-user` as the **Common Name**.

### 2. Sign the client certificate with your Root CA:
```sh
sudo openssl x509 -req -in client.csr -CA MyOrg-RootCA/MyOrg-RootCA.crt -CAkey MyOrg-RootCA/MyOrg-RootCA.key -CAcreateserial -out client.crt -days 365 -sha256
```

### 3. Create a `.p12` certificate for browser import:
```sh
sudo openssl pkcs12 -export -out client.p12 -inkey client.key -in client.crt -certfile MyOrg-RootCA/MyOrg-RootCA.crt
```

### 4. Configure Apache to Require Client Certificates:
Edit the SSL configuration:
```sh
sudo nano /etc/apache2/sites-available/ctf-website-ssl.conf
```
Add these lines inside `<VirtualHost *:443>`:
```apache
SSLVerifyClient require
SSLVerifyDepth 1
SSLCACertificateFile /etc/ssl/server/MyOrg-RootCA/MyOrg-RootCA.crt
```

### 5. Reload Apache:
```sh
sudo systemctl reload apache2
```

### 6. Test the configuration:
Visit:  
[Secure CTF Website](https://ctf-debian12-30.ctf.edu.technet.howest.be/)  
The browser should block access without a valid client certificate.

---

## Step 29: Transferring the Client Certificate
### 1. Copy the `.p12` file to the web directory:
```sh
sudo cp /etc/ssl/server/client.p12 /var/www/ctf-website/client.p12
sudo chmod 644 /var/www/ctf-website/client.p12
```

### 2. Modify Apache to allow certificate download:
Edit:
```sh
sudo nano /etc/apache2/sites-available/ctf-website-ssl.conf
```
Change:
```apache
SSLVerifyClient require
```
To:
```apache
SSLVerifyClient optional
```
Add:
```apache
<Location "/client.p12">
    SSLVerifyClient none
</Location>
```

### 3. Reload Apache:
```sh
sudo systemctl reload apache2
```

### 4. Download the Client Certificate:
[Download Client Certificate](https://ctf-debian12-30.ctf.edu.technet.howest.be/client.p12)

### 5. Restore Apache Settings:
Revert the `SSLVerifyClient` change and remove:
```apache
<Location "/client.p12">
    SSLVerifyClient none
</Location>
```

Delete the `.p12` file:
```sh
sudo rm /var/www/ctf-website/client.p12
```

### 6. Import the `.p12` file:
- Open `certmgr.msc`
- Import into **Personal > Certificates**
- Use password: `passwordctf30`

Now, visiting [Secure CTF Website](https://ctf-debian12-30.ctf.edu.technet.howest.be/) should work.

## Step 28: Creating a Professional Website

### 1. Create an HTML Page
Edit the `index.html` file:
```sh
sudo nano /var/www/ctf-website/index.html
```
Add the following content:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CTF Web Portal</title>
    <link rel="stylesheet" href="style.css">
    <script src="script.js"></script>
</head>
<body>
    <div class="container">
        <h1>Welcome to the CTF Website</h1>
        <p>This server is secured with HTTPS, client authentication, and follows hardening best practices.</p>
        <ul>
            <li><a href="/mickey/">Mickey's Page</a></li>
            <li><a href="/mouse/">Mouse's Page</a></li>
            <li><a href="/mickeymouse/">Shared Page</a></li>
        </ul>
    </div>
</body>
</html>
```

---

### 2. Add CSS for Styling
Edit the `style.css` file:
```sh
sudo nano /var/www/ctf-website/style.css
```
Paste the following CSS:
```css
body {
    font-family: 'Segoe UI', sans-serif;
    background-color: #f0f2f5;
    color: #333;
    text-align: center;
    margin-top: 100px;
}

.container {
    background: white;
    padding: 40px;
    max-width: 600px;
    margin: auto;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0,0,0,0.1);
}

a {
    color: #2e5aac;
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}

.welcome-message {
    opacity: 0;
    transform: translateY(-20px);
    transition: all 0.8s ease;
    background-color: #2e5aac;
    color: white;
    padding: 20px;
    border-radius: 0 0 10px 10px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.welcome-message.visible {
    opacity: 1;
    transform: translateY(0);
}
```

---

### 3. Add JavaScript for Animation
Edit the `script.js` file:
```sh
sudo nano /var/www/ctf-website/script.js
```
Paste the following JavaScript:
```js
document.addEventListener("DOMContentLoaded", () => {
    const welcomeBox = document.createElement("div");
    welcomeBox.className = "welcome-message";
    welcomeBox.innerHTML = "<h2>Welcome to the CTF Web Portal 👋</h2><p>Secure. Professional. Yours.</p>";
    
    document.body.prepend(welcomeBox);

    setTimeout(() => {
        welcomeBox.classList.add("visible");
    }, 100);
});
```

---

### 4. Verify the Website
After setting up the HTML, CSS, and JavaScript files, visit:  
[CTF Website](https://ctf-debian12-30.ctf.edu.technet.howest.be/)

Your website should display a professional interface with a smooth welcome animation.

---
