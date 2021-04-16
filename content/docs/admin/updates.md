+++
title = "Updating BookStack"
description = "How to update BookStack to the lastest version"
date = "2017-01-01"
type = "admin-doc"
+++

BookStack is updated regularly and is still in beta although we do try to keep the platform and upgrade path as stable as possible. The latest release can be found on [GitHub here](https://github.com/BookStackApp/BookStack/releases) and detailed information on releases is posted on the [BookStack blog here](/tags/releases/).

**Before updating you should back up the database and any file uploads to prevent potential data loss**. <br>
Backup and restore documentation can be found [here](/docs/admin/backup-restore).

 Updating is currently done via Git version control. To update BookStack you can run the following command in the root directory of the application:

```bash
git pull origin release && composer install --no-dev -a && php artisan migrate
```

This command will update the repository that was created in the installation, install the PHP dependencies using `composer` then run then update the database with any required changes.

In addition, Clearing the cache is also recommended:

```bash
php artisan cache:clear
php artisan config:clear
php artisan view:clear
```

Check the below list for the version you are updating to for any additional instructions.

---

## Version Specific Instructions

The below lists things you may need to be aware of when upgrading to a newer version of BookStack.


#### Updating to v21.04 or higher

**Requirements Change** - The minimum required PHP version has changed from 7.2 to 7.3. If you originally updated using the Ubuntu 18.04 installation script, the below commands should help you to upgrade to PHP8:

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install -y php8.0 php8.0-fpm php8.0-curl php8.0-mbstring php8.0-ldap php8.0-xml php8.0-zip php8.0-gd php8.0-mysql libapache2-mod-php8.0
sudo a2dismod php7.2
sudo a2enmod php8.0
systemctl restart apache2
```

**User Reference Changes** - References to users in URLs of profile pages, and within search filters, has been changed to be name (Slugified) based instead of ID based. Old links or saved search filters may no longer work as expected.

#### Updating to v0.31.0 or higher

**Requirements Change** - The minimum required PHP version has changed from 7.2 to 7.2.5. Additionally, the `Tidy` PHP extension is no longer required.

**GitLab Authentication** - The `read_user` scope will now be passed and is required on the "Application" setup within GitLab. Not having this scope may lead to errors when users attempt to authenticate via GitLab.

**Security & IFrame Usage** - By default BookStack will set headers to prevent usage within an iframe. You can set trusted iframe hosts through the `ALLOWED_IFRAME_HOSTS` option in your `.env` file. See the [security page](/docs/admin/security#iframe-control) for more information on this option.

#### Updating to v0.30.6, v0.30.7 or higher

**Security** - v0.30.6 and v0.30.7 both address issues where page content could be visible to those without permission. If a chapter was visible to a user, but all of it's pages were made not visible, then the details of these pages could be visible. Within the BookStack interface, the names of the pages and preview content could be seen. If the parent book was exported then this would include the content of the pages that had been restricted. If using BookStack v0.30.6, then all non-visible page content could be visible in plaintext exports. Please see the blog release pages for more details: [v0.30.6](/blog/beta-release-v0-30-6/), [v0.30.7](/blog/beta-release-v0-30-7/).

#### Updating to v0.30.5 or higher

**Security** - v0.30.5 fixes an potential vulnerability where a user with edit permissions could perform server-side requests using the export system. Additionally it was found that, if using standard email/password authentication, the system host URL could be manipulated on the forgot password form which could allow for email phishing attempts. Ensure you have set the `APP_URL` option in your `.env` file to help prevent this. Please see the [blog release page for more details](/blog/beta-release-v0-30-5/).

#### Updating to v0.30.4 or higher

**Security** - v0.30.4 fixes a couple of XSS vulnerabilities that could be exploited by untrusted users via page content and page link attachments. Please see the [blog release page for more details](/blog/beta-release-v0-30-4/).

#### Updating to v0.30 or higher

**Security** - Possible Privilege Escalation. During the v0.30 release cycle
it was advised that current privilege escalation situations are not made clear when applying role permissions.
Any user with a "Manage app settings", "Manage users" or "Manage roles & role permissions" system permission 
assigned to one of their roles could technically alter their own permissions to gain wider access.
A clear advisory of these cases has been added in the UI in v0.30
but admins are advised to review which users have these permissions with the above in mind.


**LDAP & SAML Group Matching** - During the v0.30 release cycle it was found that 
BookStack roles would be matched to LDAP/SAML groups based upon the role display name, which is expected,
but only those roles with a matching "name" value would be considered. This "name" field was redundant, 
and has now been removed, but it would store a cleaned version the first-set name of the role.
All roles will now be considered before being matched on name which may mean that roles which did not sync before, 
that would have been expected to based on their name, may now start to sync.


#### Updating to v0.29.3 or higher

**Security** - v0.29.3 fixes an issue where the names of restricted/private books could seen by those without permission, if added to a shelf. This issue was introduced in BookStack v0.28.0.

#### Updating to v0.29.2 or higher

**Security** - v0.29.2 fixes a XSS security vulnerability in the comment system, that was introduced in BookStack v0.18. Upon updating the command `php artisan bookstack:regenerate-comment-content` should be ran to regenerate comment content to ensure that it is safe.

#### Updating to v0.28 or higher

**Requirements Change** - Minimum PHP version has increased from 7.0.5 to 7.2.

If you installed BookStack on Ubuntu 16.04 using the install script, You should be able to upgrade your PHP version to 7.4 by running the following commands:

```bash
sudo add-apt-repository -y ppa:ondrej/php
sudo apt update
sudo apt install -y php7.4 php7.4-fpm php7.4-curl php7.4-mbstring php7.4-ldap php7.4-tidy php7.4-xml php7.4-zip php7.4-gd php7.4-mysql
sudo sed -i.bak 's/php7\.0-fpm/php7.4-fpm/' /etc/nginx/sites-available/bookstack
sudo systemctl restart nginx.service
```

#### Updating to v0.26 or higher

**Internet Explorer Support** - IE11 Support has now been dropped. Please use a modern browser.

**Translations** - Since many interfaces and lines of text have been updated, It may take a little while for some translations to catch-up. Expect to see more English text than usual if you're using a non-English language option.

**Images** - Due to changes how images are handled, as detailed below, some types of images may become inaccessible. Old logo images will be deleted when changed. Unused Book/Shelf cover images & User profile images will be become inaccessible after the update so you may want to delete them before upgrade.

**Security** - On previous versions of BookStack it was possible for users to insert JavaScript via the Markdown editor using `on*` html attributes. These will now be removed on page render unless you have set `ALLOW_CONTENT_SCRIPTS=true`. If untrusted users has access to your BookStack you may want to scan for `<<space_char>>on` in the HTML column of the pages table to identify any malicious intent.

#### Updating to v0.25.3 or higher

**Security** - On previous versions of BookStack it was possible to upload PHP files via the image upload endpoints. If you have a BookStack instance where untrusted users could upload image files, and you were using the default file storage option, It would have been possible for the user to access anything that the PHP process could. This would likely include, at minimum, any credentials stored in the `.env` file.

#### Updating to v0.25.2 or higher

**Configuration Change** - The .env option `REDIS_CLUSTER` has now been removed. If more than one redis server is provided they will automatically be clustered by BookStack.

#### Updating to v0.25 or higher

**Security** - During the release cycle for Version v0.25 it was found that page content includes could leak their content as preview text to users that don’t have permission to view the included content. It’s recommended to re-save any pages that included other page content that’s restricted to ensure included text is not shown in page preview text.

**Requirements Change** - Minimum required version of PHP has changed from 7.0.0 to 7.0.5.

**Configuration Change** - The .env option `GRAVATAR_URL=false` has been replaced by `AVATAR_URL=false`.


#### Updating to v0.24 or higher

Version v0.24 changes the way the homepage option is stored. After updating, You may need to re-configure this setting.

If updating from a much older BookStack version (Pre v0.20) you may need to update the permission and search indexes. You can do this by running the following commands from your BookStack install folder:

```bash
php artisan bookstack:regenerate-search
php artisan bookstack:regenerate-permissions
```

#### Updating to v0.19 or higher

Version v0.19 needs the following requirement change:

* Minimum required version of PHP has changed from 5.6.4 to 7.0.0.

#### Updating to v0.18 or higher

Version v0.18 introduced a commenting system. After updating you should check the permissions for all roles if you'd like to enable comments for your users.

#### Updating to v0.13 or higher

The v0.13 release contained some new features and updates which change the requirements of BookStack.

* Minimum required version of PHP has changed from 5.5.9 to 5.6.4.
  Upgrade your PHP version if below 5.6.4.
* PHP-Tidy extension is now required.
  - On Ubuntu 16.04 this can be installed via `sudo apt install php7.0-tidy`.
  - On Ubuntu 14.04 (Using the defauly PHP option) this can be installed via `sudo apt-get install php5-tidy`.
* Page attachments will be stored in the `storage/uploads` folder (Unless you use Amazon S3). This folder will be created on update. Ensure your webserver has write permissions for this folder.
