<!--
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2016-11-22T21:47:31-08:00"
title = "Magento2 Overwrite Core module's javascript"
-->

## Magento2 How to overwrite Core module's javascript

We purchased Magestore OneStepCheckout module for magento2 community edition. But for this module when we go to checkout page, update qty/remove item or just update shipping details, it will update payment methods.

If user have input payment details, then they want to update itme/shipping detials, because it will update payment methods as well, and all these input details will gone.

That's why I need to overwrite core module's javascript.

* 1. Define the new javascript that you overwrite from core module

Put the Js file to your module view/frontend/web/js/payment-service.js

```java
/**
 * Copyright © 2016 Magento. All rights reserved.
 * See COPYING.txt for license details.
 */
define(
    [
        'underscore',
        'Magento_Checkout/js/model/quote',
        'Magento_Checkout/js/model/payment/method-list',
        'Magento_Checkout/js/action/select-payment-method'
    ],
    function (_, quote, methodList, selectPaymentMethod) {
        'use strict';
        var freeMethodCode = 'free';

        return {
            isFreeAvailable: false,
            /**
             * Populate the list of payment methods
             * @param {Array} methods
             */
            setPaymentMethods: function (methods) {
                var self = this,
                    freeMethod,
                    filteredMethods,
                    methodIsAvailable;

                freeMethod = _.find(methods, function (method) {
                    return method.method === freeMethodCode;
                });
                this.isFreeAvailable = freeMethod ? true : false;

                if (self.isFreeAvailable && freeMethod && quote.totals().grand_total <= 0) {
                    methods.splice(0, methods.length, freeMethod);
                    selectPaymentMethod(freeMethod);
                }
                filteredMethods = _.without(methods, freeMethod);

                if (filteredMethods.length === 1) {
                    selectPaymentMethod(filteredMethods[0]);
                } else if (quote.paymentMethod()) {
                    methodIsAvailable = methods.some(function (item) {
                        return item.method === quote.paymentMethod().method;
                    });
                    //Unset selected payment method if not available
                    if (!methodIsAvailable) {
                        selectPaymentMethod(null);
                    } else {
                        selectPaymentMethod(quote.paymentMethod());
                    }
                }

                var oldMethods = methodList();
                if (oldMethods.length === methods.length) {
                    for(var i=0; i < oldMethods.length; i++) {
                        if (oldMethods[i].title != methods[i].title) {
                            methodList(methods);
                            break;
                        }
                    }
                } else {
                    methodList(methods);
                }
                //methodList(methods);
            },
            /**
             * Get the list of available payment methods.
             * @returns {Array}
             */
            getAvailablePaymentMethods: function () {
                var methods = [],
                    self = this;
                _.each(methodList(), function (method) {
                    if (self.isFreeAvailable && (
                        quote.totals().grand_total <= 0 && method.method === freeMethodCode ||
                        quote.totals().grand_total > 0 && method.method !== freeMethodCode
                        ) || !self.isFreeAvailable
                    ) {
                        methods.push(method);
                    }
                });

                return methods;
            }
        };
    }
);
```

* 2. Define it in your requirejs-config.js

```java
var config = {
    map: {
        '*': {
            "Magento_Checkout/js/model/payment-service" : 'Your_ModuleName/js/payment-service'
        }
    },
    "shim": {
        "Your_ModuleName/js/payment-service": {
                "exports": "Magento_Checkout/js/model/payment-service"
            }
        }
};
```

Then enjoy it!
