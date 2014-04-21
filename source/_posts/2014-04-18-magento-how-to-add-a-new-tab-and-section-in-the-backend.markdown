---
layout: post
title: "How to add a new tab and section in the backend"
date: 2014-04-18 20:37:50 -0300
comments: true
categories: Magento
published: false
---
## Objective

We are going to see step by step how to add a custom tab, section and group of fields in the Magento backend configuration area.

{% img center /images/magento-backend-new-tab.png 231 79 'Magento backend custom tab' 'images' %}
{% img center /images/magento-backend-new-section.png 658 310 'Magento backend custom section' 'images' %}

## Procedure

This post assumes you have already created a new module so we must edit the following configuration file:

    /app/code/local/<company>/<modulo>/etc/config.xml

We must then add our custom *helper* like this:

    <config>
    <modules>
        <LCS_Ebay>
            <version>0.1.0</version>
        </LCS_Ebay>
    </modules>
    <global>
        <helpers>
            <ebay>
                <class>LCS_Ebay_Helper</class>
            </ebay>
        </helpers>

In this example we are planning to create a new section, therefore it is mandatory to create a new ACL role to be able to access this URL from the backend.
This new ACL role must be embedded into the *config.xml* file as well.

    <adminhtml>
        <acl>
            <resources>
                <admin>
                    <children>
                        <system>
                            <children>
                                <config>
                                    <children>
                                        <ebay>
                                            <title>Ebay</title>
                                        </ebay>
                                    </children>
                                </config>
                            </children>
                        </system>
                    </children>
                </admin>
            </resources>
        </acl>

Finally, we must edit this configuration file

    /app/code/local/<empresa>/<modulo>/etc/system.xml

in order to create our custom tab, section and groups of custom fields.

    <?xmlversion="1.0"?>
    <config>
        <tabs>
            <ebay translate="label" module="ebay">
                <label>Market Places</label>
                <sort_order>99999</sort_order>
            </ebay>
        </tabs>
        <sections>
            <ebay translate="label" module="ebay">
                <label>Ebay</label>
                <tab>ebay</tab>
                <frontend_type>text</frontend_type>
                <sort_order>1000</sort_order>
                <show_in_default>1</show_in_default>
                <show_in_website>1</show_in_website>
                <show_in_store>1</show_in_store>
                <groups>
                    <ebay translate="label" module="ebay">
                        <label>API Credentials</label>
                        <frontend_type>text</frontend_type>
                        <sort_order>100</sort_order>
                        <show_in_default>1</show_in_default>
                        <show_in_website>1</show_in_website>
                        <show_in_store>1</show_in_store>
                        <fields>
                            <dev_id translate="label">
                                <label>Dev Id</label>
                                <frontend_type>text</frontend_type>
                                <validate>required-entry</validate>
                                <sort_order>1</sort_order>
                                <show_in_default>1</show_in_default>
                                <show_in_website>1</show_in_website>
                                <show_in_store>1</show_in_store>
                            </dev_id>
                        </fields>
                    </ebay>
                </groups>
            </ebay>
        </sections>
    </config>

For further information about every tag, please check the following [link](http://www.magentocommerce.com/wiki/5_-_modules_and_development/admin/xml_structure_for_admin_configurations)



