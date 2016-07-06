---
layout: article_with_sidebar
lang: en
title: 'Working with image resizing routine'
---
# Introduction

Default X-Cart _lazy_ resizes images and this article describes how to work with this process.

To illustrate the image-resizing process, imagine that you uploaded 5000px x 5000px product image and this image is used for thumbnail. Default thumbnail size is 160px x 160px and showing this big image to a client without resizing would unnecessarily slow down the load time of the page, that is why X-Cart will resize 5000x5000 image on-fly and then store this resized copy to an image cache.

# Table of Contents

*   [Introduction](#introduction)
*   [Table of Contents](#table-of-contents)
*   [Disabling image resizing routine](#disabling-image-resizing-routine)
*   [Module pack](#module-pack)

# Disabling image resizing routine

In order to disable image resizing routine in a whole store you can apply the following simple mod:

1.  [Create an empty module]({{ baseurl_lang }}/getting_started/step_1_-_creating_simplest_module.html). We are creating a module with developer ID **Tony** and module ID **DisableImageResize**.
2.  [Decorate]({{ baseurl_lang }}/getting_started/step_3_-_applying_logic_changes.html) the `\XLite\View\Image` class ([more info about classnames]({{ baseurl_lang }}/misc/x-cart_classes_structure_and_namespaces.html)), so that your class would look as follows: 

    {% highlight php %}{% raw %}
    <?php
    // vim: set ts=4 sw=4 sts=4 et:

    namespace XLite\Module\Tony\DisableImageResize\View;

    class Image extends \XLite\View\Image implements \XLite\Base\IDecorator
    {
    	protected function defineWidgetParams() {
    		parent::defineWidgetParams();
    		$this->widgetParams[self::PARAM_USE_CACHE] = new \XLite\Model\WidgetParam\Bool('Use cache', 0);
    	}
    }
    {% endraw %}{% endhighlight %}
3.  The only thing we change is we set `PARAM_USE_CACHE` to false. If you check default `\XLite\View\Image` class, you will see that this `PARAM_USE_CACHE` triggers this condition: 

    {% highlight php %}{% raw %}
    $url = $this->getParam(self::PARAM_USE_CACHE)
                    ? $this->resizedURL
                    : $this->getParam(self::PARAM_IMAGE)->getFrontURL();
    {% endraw %}{% endhighlight %}

    so if `PARAM_USE_CACHE` is true, then X-Cart returns a resized image, otherwise it returns default image URL.

4.  Now it is time to re-deploy the store and check the results in store-front.

# Module pack

You can download this module example here: [https://dl.dropboxusercontent.com/u/23858825/Tony-DisableImageResize-v5_1_0.tar](https://dl.dropboxusercontent.com/u/23858825/Tony-DisableImageResize-v5_1_0.tar)