<!--
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2017-07-18T21:47:31-08:00"
title = "Magento2 MageStore Onestepcheckout bug when using braintree"
-->

## Magento2 MageStore Onestepcheckout bug when using braintree

We purchased Magestore OneStepCheckout module for magento2 community edition. Recently we setuped braintree payment gateway.

But soon we some time when we go to checkout page and input correct credit card details and place order, and payment can not be processed, it returns invalid card numbers.

It defenerly is not the credit card's problem.

So we just use chrome debuger to check the details, we found there are some js error log in the console.

"Can not replace element of id #credit-card-number"
"Can not replace element of id #expiration-month"
"Can not replace element of id #expiration-year"

At first we thought that's magento braintree module's bug, but when we debug into the UI component(I don't like UI component), and it seems some time this component is rendered 3 or 4 times.

Finally we found the problem is because onestepcheckout assemble address, shipping, cart detals options in one page, and all these parts are UI component. Once these part got databinding, and it will update payment infomation which will re-calculate avalible payment method lists and set back to knockoutjs payment-service(which is an observerarray).

But for this version knockoutjs doesn't handle multiple process access same observer array.

All these payment method UI component renderer  are called by observerarray subscriber, so it will render many times.

If some time we are not lucky, last renderer not finish yet, and payment methods list are updated, it will render again. Then the error shows.

* 1. Define the new javascript that you overwrite from core module

We just override the default payment-service, put all payment list to the queue first, and set a 2secs timer, if in w2secs there's  no methodlist come in, we render the last one.

Put the Js file to your module view/frontend/web/js/payment-service.js

```php
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
            methods_queue:[],
            timer:null,
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
                    }
                }
                self.methods_queue.push(methods);
                if (self.timer != null) {
                        clearTimeout(self.timer);
                }
                self.timer = setTimeout(function() {
                        console.log('setpaymentmethod time out trigered');
                        methodList(self.methods_queue.pop());
                        self.methods_queue = [];
                }, 2000);
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
