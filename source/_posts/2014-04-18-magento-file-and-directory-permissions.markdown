---
layout: post
title: "Magento: File and Directory Permissions"
date: 2014-04-18 17:02:19 -0300
comments: true
categories: Magento
---
Once you install Magento for the first time, you need to make sure files and directories hold the right set of permissions. We assume in this example that we are working on a CentOS platform and the httpd (owner and group) is apache. Then we must assign apache as owner and group of the web root where Magento resides. In our case this location is
    
    /var/www/html/magento

Then we set the right permissions

    cd /var/www/html
    sudo chown apache:apache magento -R 

Then, we go to the magento directory and run the following commands

    cd magento
    sudo find . -type f -print -exec chmod 664 {} \;
    sudo find . -type d -print -exec chmod 775 {} \;
    chmod 550 mage
    sudo chmod o+w var app/etc