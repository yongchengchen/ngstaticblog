<!--
Categories = ["Development", "Magento"]
Description = ""
Tags = ["Development", "Magento"]
date = "2016-10-26T21:47:31-08:00"
menu = "main"
title = "Magento2 knockoutjs work with angularjs"
-->

# Magento2 Let angularjs work with knockout.js

* Why doing this?

Before Mangento2 we already using angularjs and also developed many angularjs resources(expecially modals).
So we just want to reuse these resources.

Here's an example for Mangento2 admin panel. (For frontend, I recommend you just only use knockout.js)

* 1. Create a Block class Container Tab

This container tab is for containing angularjs template. 

You can contain many templates you want by passing "child_templates" parameters via layout.xml

```php
<?php

namespace Yong\Angularjs\Block\Adminhtml;

use Magento\Backend\Block\Widget\Tab;
use Magento\Backend\Block\Widget\Tabs;
use Magento\Backend\Block\Widget;

class ContainerTab extends Tab {
    /**
     * Prepare html output
     *
     * @return string
     */
    protected function _toHtml() {
        if ($this->hasData('child_templates')) {
            foreach($this->getData('child_templates') as $name => $childTemplate) {
                $child = $this->addChild($name,
                    'Magento\Backend\Block\Widget', 
                    ['template'=>$childTemplate]
                );
            }
        }
        
        return $this->getChildHtml();
    }
}
```

* 2. Create a UI Modifier to prepare knockoutjs swap data fields

```php
<?php
namespace Yong\Angularjs\Ui\DataProvider\Product\Form;

use Magento\Catalog\Ui\DataProvider\Product\Form\Modifier\AbstractModifier;
use Magento\Catalog\Api\Data\ProductAttributeInterface;
use Magento\Framework\Stdlib\ArrayManager;
use Magento\Catalog\Model\Locator\LocatorInterface;

use Magento\Framework\UrlInterface;
use Magento\Ui\Component\Container;
use Magento\Ui\Component\Form\Fieldset;
use Magento\Ui\Component\Form;
use Magento\Ui\Component\DynamicRows;

/**
 * Customize Price field
 */
class Modifier extends AbstractModifier
{
    const FIELD_NAME = 'angularjs_swap_data';
    const FIELDSET_NAME = 'angularjs_swap_sets';

    /**
     * @var ArrayManager
     */
    protected $arrayManager;

    /**
     * @var LocatorInterface
     */
    protected $locator;

    private $swapFieldNames;

    /**
     * @param LocatorInterface $locator
     * @param ArrayManager $arrayManager
     */
    public function __construct(
        LocatorInterface $locator,
        ArrayManager $arrayManager,
        $swapFieldNames = [self::FIELD_NAME]
    ) {
        $this->locator = $locator;
        $this->arrayManager = $arrayManager;
        if (!is_array($swapFieldNames)) {
            $swapFieldNames = [$swapFieldNames];
        }
        $this->swapFieldNames = $swapFieldNames;
    }

    /**
     * {@inheritdoc}
     */
    public function modifyMeta(array $meta)
    {
        $this->meta = $meta;
        $this->addFieldset();

        return $this->meta;
    }

    public function modifyData(array $data) {
        return $data;   
    }

    protected function addFieldset()
    {
        $children = [];
        $i = 0;
        foreach($this->swapFieldNames as $swapFieldName) {
            $children[$swapFieldName] = $this->getFieldConfig($swapFieldName, 10*$i++);
        }
        $this->meta = array_replace_recursive(
            $this->meta,
            [
                static::FIELDSET_NAME => [
                    'arguments' => [
                        'data' => [
                            'config' => [
                                'label' =>'',
                                'componentType' => Fieldset::NAME,
                                'dataScope' => 'data',
                                'collapsible' => true,
                                "visible" => true,
                                'sortOrder' => 10,
                            ],
                        ],
                    ],
                    'children' => $children
                ],
            ]
        );

        return $this;
    }

    protected function getFieldConfig($swapFieldName, $sortOrder)
    {
        return [
            'arguments' => [
                'data' => [
                    'config' => [
                        'label' => '',
                        'componentType' => \Magento\Ui\Component\Form\Field::NAME,
                        'formElement' => \Magento\Ui\Component\Form\Element\Input::NAME,
                        'dataScope' => $swapFieldName,
                        'dataType' => \Magento\Ui\Component\Form\Element\DataType\Text::NAME,
                        'sortOrder' => $sortOrder,
                        "visible" => true,
                        'required' => true,
                    ],
                ],
            ],
            'children' => [],
        ];
    }
}

```

* 3. Reserve knockoutjs swap fields in di.xml

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Yong\Angularjs\Ui\DataProvider\Product\Form\Modifier">
        <arguments>
            <argument name="locator" xsi:type="object">Magento\Catalog\Model\Locator\LocatorInterface</argument>
            <argument name="arrayManager" xsi:type="object">Magento\Framework\Stdlib\ArrayManager</argument>
            <argument name="swapFieldNames" xsi:type="string">angularjs_swap_data</argument>
        </arguments>
    </type>
    <virtualType name="angularjs_swap_dataset1"
        type="Yong\Angularjs\Ui\DataProvider\Product\Form\Modifier" >
        <arguments>
            <argument name="swapFieldNames" xsi:type="array">
                <item name="field1" xsi:type="string">angularjs_swap_data_1</item>
                <item name="field2" xsi:type="string">angularjs_swap_data_2</item>
            </argument>
        </arguments>
    </virtualType>
    
    <virtualType name="Magento\Catalog\Ui\DataProvider\Product\Form\Modifier\Pool">
        <arguments>
            <argument name="modifiers" xsi:type="array">
                <item name="comboproduct" xsi:type="array">
                    <item name="class" xsi:type="string">angularjs_swap_dataset1</item>
                    <item name="sortOrder" xsi:type="number">125</item>
                </item>
            </argument>
        </arguments>
    </virtualType>
</config>
```

* 4. Put your container tab to layout.xml

```xml
<?xml version="1.0"?>
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <update handle="catalog_product_form"/>
    <body> 
        <referenceContainer name="product_form" >
            <block class="Yong_Angularjs\Block\Adminhtml\ContainerTab">
                <arguments>
                    <argument name="config" xsi:type="array">
                        <item name="label" xsi:type="string" translate="true">Angularjs tab</item>
                        <item name="collapsible" xsi:type="boolean">true</item>
                        <item name="opened" xsi:type="boolean">true</item>
                        <item name="sortOrder" xsi:type="string">2</item>
                        <item name="canShow" xsi:type="boolean">true</item>
                        <item name="componentType" xsi:type="string">fieldset</item>
                    </argument>
                    <argument name="child_templates" xsi:type="array">
                        <item name="subblock1" xsi:type="string">Yong_Angularjs::catalog/product/edit/tab/angularjs.tab.phtml</item>
                    </argument>
                </arguments>
            </block>
        </referenceContainer>
    </body>
</page>

```

* 5. Add js resource and config requirejs-config.js

```php
var config = {
    map: {
        '*': {
            angular: 'Yong_Angularjs/bower/angular/angular.min',
            angularBootstrap: 'Yong_Angularjs/bower/angular-bootstrap/ui-bootstrap-tpls.min',
            ngstorage: 'Yong_Angularjs/bower/ngstorage/ngStorage.min',
            mymodalService: 'Yong_Angularjs/js/mymodalService',
            modalScript: 'Yong_Angularjs/js/modalScript',
            syncKo: 'Yong_Angularjs/js/syncKo',
            mytabController: 'Yong_Angularjs/mytabController'
        }
    },
    "shim": {
        "Yong_Angularjs/bower/angular/angular.min": {
                "exports": "angular"
            },
        "Yong_Angularjs/bower/angular-bootstrap/ui-bootstrap-tpls.min": {
                "exports": "angularBootstrap"
            },
        "Yong_Angularjs/bower/ngstorage/ngStorage.min": {
                "exports": "ngStorage"
            },
        "Yong_Angularjs/js/mymodalService": {
                "exports": "mymodalService"
            },
        "Yong_Angularjs/js/modalScript": {
                "exports": "modalScript"
            },
        "Yong_Angularjs/js/syncKo": {
                "exports": "syncKo"
            },
        "Yong_Angularjs/mytabController": {
                "exports": "mytabController"
            }
        }
};
```

* 6. Define angularjs directive "syncko" to sync angularjs object to knockoutjs swap data field(formart as JSON)

```javascript
define(['jquery'], function($){
    'use strict';
    function syncKo() {
        return function($scope, element, attrs) {
            $scope.$watch(attrs.from, function(newValue, oldValue) {
                if (newValue !== undefined) {
                    $('input[name='+ attrs.to +']').val(JSON.stringify(newValue)).trigger('change');
                }
            }, true);
        };
    }
    return syncKo;
});
```

* 7. Use knockoutjs swap fields in angularjs template
```html
<div class="container fieldset-wrapper" ng:controller="controller" id='ng-controller' 
    style="margin-left:20px;margin-top: 0" sync-ko from='products' to='angularjs_swap_data_1'>
```
