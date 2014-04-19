---
layout: post
title: "Magento: How to know if we are at the homepage"
date: 2014-04-19 12:28:08 -0300
comments: true
categories: Magento
---
There are times when, from a certain template we need to know if we are at the home page. Assuming our home page has been defined at the CMS and it is called *home*, we could use this snippet:

     <?php 
        $page = Mage::app()->getFrontController()->getRequest()->getRouteName();
        
        if (($page == 'cms') && Mage::getSingleton('cms/page')->getIdentifier() == 'home'){
            $this->setColumnCount(4)
        }
     ?>

