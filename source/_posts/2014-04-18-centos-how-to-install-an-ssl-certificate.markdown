---
layout: post
title: "CentOS: How to install an SSL certificate"
date: 2014-04-18 22:36:30 -0300
comments: true
published: false
categories: [ CentOS, AWS ]
---
## Objective

Install an SSL certificate on CentOS using GoDaddy as certification authority. This is a standard procedure when you need to configure a HTTPS server.

## Procedure

Install the SSL module

    yum install mod_ssl openssl

Generate private and public keys as follows

    openssl genrsa –des3 –out <key name>.key 2048
    openssl req –new –key <key name>.key –out <key name>.csr 

In the case of standard certificates for HTTPS the key name corresponds to the domain name.

Log into your GoDaddy.com account and at the SSL Certificates section (after you have purchased one), click on Lunch. From the filters list, click Certificates and then select the name of the certificate you would like to download. 
Click download and select the Server Type (Apache)




