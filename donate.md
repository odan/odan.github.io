---
title: Donate
date: 2020-04-05
layout: default
comments: true
published: true
description: Donate
keywords: donate paypal stripe 
---

# Donate

If you think my blog and support is useful for you, 
I would appreciate a small donation:

* **[PayPal](https://www.paypal.me/dopitz/5)**
* **[Buy a Coffee](https://ko-fi.com/dopitz)**

You can **buy** all Slim articles (> 270 pages) bundled into a eBook (PDF and EPUB). 

Exclusive content:

* Slim 4 Introducing
* JSON Web Token (JWT) authentication
* PHP Templates. Rendering PHP view scripts into a PSR-7 Response with `slim/PHP-View`

<div style="text-align: center;">
<img src="https://user-images.githubusercontent.com/781074/92961116-0293a880-f46f-11ea-9e84-90ef9781e0c8.png">
</div>

<script src="https://cdn.jsdelivr.net/npm/sweetalert2@10"></script>

<div id="paypal-button-container"></div>
<script src="https://www.paypal.com/sdk/js?client-id=AdgjzHMB-hvxJGnVoxlkeGzRwFlItWRfDCLLIeHMwqmi6TpUU24NlOyWjbCmPiHTzpekYxqgXA1IyJYz&currency=EUR" data-sdk-integration-source="button-factory"></script>
<script>
    paypal.Buttons({
        style: {
            shape: 'rect',
            color: 'gold',
            layout: 'vertical',
            label: 'paypal',

        },
        createOrder: function(data, actions) {
            return actions.order.create({
                purchase_units: [{
                    amount: {
                        value: '9'
                    }
                }]
            });
        },
        onApprove: function(data, actions) {
            return actions.order.capture().then(function(details) {
                //alert('Transaction completed by ' + details.payer.name.given_name + '!');
                Swal.fire({
                    icon: 'success',
                    text: 'Transaction completed. Thank you! Please check your inbox. If you did not receive your activation email, please check your spam filter settings and your spam folders.',
                });
            });
        },
        onCancel: function (data, actions) {
            Swal.fire({
                icon: 'error',
                text: 'Transaction canceled. Please try it again!',
            });
        }
    }).render('#paypal-button-container');
</script>

Please note: If you have not received your ebook, please check your spam email folder.
