---
layout: article_with_sidebar
lang: en
title: 'Adding new property to a product - part 2 - value of this field must be saved after checkout'
---
# Introduction

This article is a continuation of an article about [adding a new property to a product]({{ baseurl_lang }}/basics/adding_new_property_to_a_product/{{ baseurl_lang }}/index.html) and then displaying this property's value on the invoice page. The problem with that mod is that if we change the property's value for some product, then invoices of orders that contain this product will change its value as well. In some cases, it is OK, but sometimes it is not (e.g. price property).

This article will explain how to save the product property's value when you place an order and even if you change this property's value later on, old invoices would still use saved value.

This article assumes that you are already familiar with the task described in the [Adding new property to a product](Adding-new-property-to-a-product_8225149.html) guide.

# Table of Contents

*   [Introduction](#introduction)
*   [Table of Contents](#table-of-contents)
*   [Implementation](#implementation)
*   [Module pack](#module-pack)

# Implementation

We start with creating a module similar to one from the first article.

1.  We create an empty module with developer ID **Tony** and module ID **ProductOrderPropertyDemo**.
2.  We decorate the `\XLite\Model\Product` class and create the `<X-Cart>/classes/XLite/Module/Tony/ProductOrderPropertyDemo/Model/Product.php` file with the following content: 

    {% highlight php %}{% raw %}
    <?php
    // vim: set ts=4 sw=4 sts=4 et:

    namespace XLite\Module\Tony\ProductOrderPropertyDemo\Model;

    /**
     * The "product" model class
     */
    abstract class Product extends \XLite\Model\Product implements \XLite\Base\IDecorator
    {
        /**
         * @Column (type="string", length=32)
         */
        protected $testField;
    }
    {% endraw %}{% endhighlight %}

    This code will add new field to a product class. The same as in the previous guide.

3.  We decorate the `\XLite\View\Model\Product` class and create the `<X-Cart>/classes/XLite/Module/Tony/ProductOrderPropertyDemo/View/Model/Product.php` file with the following content: 

    {% highlight php %}{% raw %}
    <?php
    // vim: set ts=4 sw=4 sts=4 et:

    namespace XLite\Module\Tony\ProductOrderPropertyDemo\View\Model;

    /**
     * Product view model
     */
    abstract class Product extends \XLite\View\Model\Product implements \XLite\Base\IDecorator
    {
        public function __construct(array $params = array(), array $sections = array())
        {
            parent::__construct($params, $sections);

            $this->schemaDefault += array (
                'testField' => array(
                    self::SCHEMA_CLASS    => 'XLite\View\FormField\Input\Text',
                    self::SCHEMA_LABEL    => 'Test field',
                    self::SCHEMA_REQUIRED => false,
                    ),
                );
        }
    }
    {% endraw %}{% endhighlight %}

    This code will create a new field on product details page in admin area, where you can specify a value of this property for a product. The same as in the previous guide.

4.  We create the `<X-Cart>/skins/default/en/modules/Tony/ProductOrderPropertyDemo/item.test-field.tpl` template that will be responsible for displaying this property on the invoice page. It will have the following content: 

    {% highlight php %}{% raw %}
    {* vim: set ts=2 sw=2 sts=2 et: *}
    {**
     * @ListChild (list="invoice.item.name", weight="20")
     *}
    <li class="test-field">
      <span class="name">{t(#Test field#)}:</span>
      <span class="test-field-value">{item.getTestField()}</span>
    </li>
    {% endraw %}{% endhighlight %}

    It is very similar to the template from the previous guide, but there is a key difference: we call the **testField** property as `{item.getTestField()}` instead of `{item.product.getTestField()}`. In other words, now we call this property from the `\XLite\Model\OrderItem` object, not from the `\XLite\Model\Product` one. The difference is that properties of `\XLite\Model\OrderItem` object are saved when you place an order and then they are not changed, even if you change the product's properties in admin area.

Now it is time to add this `$testField` property and its `getTestField()` accessor method to the `\XLite\Model\OrderItem` class and our mod will be ready.

We [decorate]({{ baseurl_lang }}/getting_started/step_3_-_applying_logic_changes.html) the `\XLite\Model\OrderItem` class and create the `<X-Cart>/classes/XLite/Module/Tony/ProductOrderPropertyDemo/Model/OrderItem.php` file with the following content: 

{% highlight php %}{% raw %}
<?php
// vim: set ts=4 sw=4 sts=4 et:

namespace XLite\Module\Tony\ProductOrderPropertyDemo\Model;

/**
 * Something customer can put into his cart
 */
abstract class OrderItem extends \XLite\Model\OrderItem implements \XLite\Base\IDecorator
{
    /**
     * @Column (type="string", length=32)
     */
    protected $testField;

    public function getTestField()
    {
        return $this->isOrderOpen() ? $this->getProduct()->getTestField() : $this->testField;
    }

    public function renew()
    {
        $result = parent::renew();

        if ($result) {
            $this->setTestField($this->getProduct()->getTestField());
        }

        return $result;
    }

    protected function getDeletedProduct()
    {
        $dumpProduct = parent::getDeletedProduct();
        $dumpProduct->testField = $this->getTestField();

		return $dumpProduct;
    }

    protected function saveItemState(\XLite\Model\Base\IOrderItem $item)
    {
        parent::saveItemState($item);
        $this->setTestField($item->getTestField());
    }

    protected function resetItemState()
    {
        parent::resetItemState();
        $this->testField = '';
    }
}

{% endraw %}{% endhighlight %}

Let us have a look at key points of this class implementation:

1.  We create the `$testField` property with the same parameters as we did in the `\XLite\Model\Product` class: 

    {% highlight php %}{% raw %}
        /**
         * @Column (type="string", length=32)
         */
        protected $testField;
    {% endraw %}{% endhighlight %}
2.  `getTestField()` method returns this property from the product object if an order has not been placed yet: 

    {% highlight php %}{% raw %}
    $this->getProduct()->getTestField()
    {% endraw %}{% endhighlight %}

    otherwise the value is returned as a `\XLite\Model\OrderItem` class' property: 

    {% highlight php %}{% raw %}
    $this->testField
    {% endraw %}{% endhighlight %}

    because at this point `xc_order_items` table already contains info about this field.

3.  These three methods: 

    {% highlight php %}{% raw %}
        public function renew()
        {
            $result = parent::renew();

            if ($result) {
                $this->setTestField($this->getProduct()->getTestField());
            }

            return $result;
        }

        protected function saveItemState(\XLite\Model\Base\IOrderItem $item)
        {
            parent::saveItemState($item);
            $this->setTestField($item->getTestField());
        }

        protected function resetItemState()
        {
            parent::resetItemState();
            $this->testField = '';
        }
    {% endraw %}{% endhighlight %}

    are implemented as follows, because they are called when `\XLite\Model\OrderItem` object has not yet taken property values from an `\XLite\Model\Product` object. That is why we define `$this->testField` property explicitly here.

4.  `getDeletedProduct()` method is called when we have to use a product that has been already deleted. In this case, we build this fake `\XLite\Model\Product` object based on the values saved in the `\XLite\Model\OrderItem` one.

This mod is ready. Now we need to re-deploy the store and check the results.

As a first step we go to an admin area and define the **Test Field** property for some product:  
![]({{ site.baseurl }}/attachments/8225458/8356212.png)

then go to your storefront, add this product to cart, proceed to checkout and place an order. You will see the following picture there:![]({{ site.baseurl }}/attachments/8225458/8356213.png)

If you go to your admin area, change the **Test Field** property for this product again and then reload the **Thank you** page, you will still see old value.

_Note: this mod does not display this property on invoice page in admin area and in email notifications. To get the idea of how to enhance this mod and show the value in admin area, please check the [Adding product images to order notifications]({{ baseurl_lang }}/design_changes/adding_product_images_to_order_notifications.html)__ guide._

# Module pack

You can download this module from here: [https://dl.dropboxusercontent.com/u/23858825/Tony-ProductOrderPropertyDemo-v5_1_0.tar](https://dl.dropboxusercontent.com/u/23858825/Tony-ProductOrderPropertyDemo-v5_1_0.tar)

## Attachments:

![](images/icons/bullet_blue.gif) [test-field-value-admin.png]({{ site.baseurl }}/attachments/8225458/8356212.png) (image/png)  
![](images/icons/bullet_blue.gif) [test-field-value-customer.png]({{ site.baseurl }}/attachments/8225458/8356213.png) (image/png)